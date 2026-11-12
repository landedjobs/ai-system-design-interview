# The AI system design answer framework

[Back to README](README.md) · [Numbers to know](numbers.md) · [All designs](README.md#the-15-designs)

One rubric, eight steps, applied to every prompt in this repo. It works for "design ChatGPT", "design a support agent" and "design an inference service" because all three share the same skeleton: a model in the middle, data going in, users waiting on the other end, and a bill at the end of the month.

Classic system design asks whether you can store and move data at scale. AI system design asks that plus three harder questions. How do you know the output is right? What happens when the model is wrong, slow or attacked? What does each request cost? Most candidates who fail this round answer the classic part well and skip those three.

## Contents

- [The eight steps at a glance](#the-eight-steps-at-a-glance)
- [Step 1. Requirements and scope](#step-1-requirements-and-scope)
- [Step 2. Estimates](#step-2-estimates)
- [Step 3. Data and evals](#step-3-data-and-evals)
- [Step 4. Retrieval and model choice](#step-4-retrieval-and-model-choice)
- [Step 5. Serving and latency](#step-5-serving-and-latency)
- [Step 6. Guardrails and safety](#step-6-guardrails-and-safety)
- [Step 7. Monitoring and cost](#step-7-monitoring-and-cost)
- [Step 8. Scaling and evolution](#step-8-scaling-and-evolution)
- [How the round is scored](#how-the-round-is-scored)
- [Phrases that signal seniority](#phrases-that-signal-seniority)

## The eight steps at a glance

| # | Step | Minutes | Output on the whiteboard |
|---|---|---|---|
| 1 | Requirements and scope | 4 to 5 | Functional list, non-functional numbers, explicit out-of-scope |
| 2 | Estimates | 3 to 5 | QPS, tokens per day, storage, GPUs or dollars per day |
| 3 | Data and evals | 5 to 7 | Eval set, metrics per layer, launch gate |
| 4 | Retrieval and model choice | 6 to 8 | Retrieval design, model tier per task, prompt shape |
| 5 | Serving and latency | 6 to 8 | Architecture diagram, latency budget per hop |
| 6 | Guardrails and safety | 3 to 5 | Trust boundaries, input and output checks, tool permissions |
| 7 | Monitoring and cost | 3 to 4 | Dashboards, alerts, cost per request, top cost levers |
| 8 | Scaling and evolution | 3 to 5 | What breaks at 10x and what you change |

Times assume a 45-minute round with ~5 minutes of intro and questions. If the interviewer pulls you deep on one step, follow them. The rubric is a map, not a script.

## Step 1. Requirements and scope

**Goal:** turn a vague prompt into a problem with numbers.

Ask about:
- **Users and use cases.** Who asks what? Internal employees, consumers, developers via API?
- **Quality bar.** What does a wrong answer cost? A bad movie suggestion is cheap. A wrong dosage or a wrong refund is expensive. This one answer sets how much you spend on evals and guardrails.
- **Scale.** Requests per day, peak factor, corpus size, number of tenants.
- **Latency.** Interactive chat (first token under ~1 s), voice (under ~800 ms voice-to-voice), batch (hours are fine).
- **Data constraints.** PII, data residency, can data go to a third-party model API?
- **Actions.** Does the system only answer, or does it act (refunds, code changes, emails)? Acting systems need a different safety design.

Write non-functional requirements as numbers: "TTFT p95 under 1.5 s, 99.9% available, faithfulness above 95% on the golden set, under $0.01 per request." A requirement without a number cannot be checked.

Say what you are not doing: "Permissions are out of scope for this round, I will mention where they plug in."

**Common miss:** jumping to the diagram in minute two. Interviewers mark it down even when the diagram is good.

## Step 2. Estimates

**Goal:** find the number that drives the design.

Do the arithmetic out loud and round hard. The usual chain:

1. Requests per day → average and peak QPS.
2. Tokens per request (input and output) → tokens per day.
3. Tokens per day × price → dollars per day. Or tokens per second ÷ per-GPU throughput → GPU count.
4. Corpus size → chunks → vectors → index bytes.

Then name the dominant term. In most LLM products it is generation cost or GPU count, not storage. In RAG at very large scale it can be index RAM. In voice it is latency, not cost.

Label everything as an estimate and say what would change your answer: "If the average prompt is 20K tokens instead of 4K, prefill dominates and I would add prompt caching first."

The reference numbers are in [numbers.md](numbers.md).

## Step 3. Data and evals

**Goal:** show how you will know it works before you build it.

This is the step that separates AI system design from classic system design, and the one most often skipped. Cover:

- **Eval set.** Where do examples come from? Real logs (best), expert-written cases, synthetic generation with human review. How many? A few hundred well-chosen cases beat 10,000 random ones. Include hard cases, adversarial cases and cases where the right answer is "I don't know".
- **Metrics per layer.** Retrieval (recall@k, nDCG), generation (faithfulness, correctness, format validity), agent (task success, pass^k over repeated runs, steps per task), system (latency, cost).
- **Judges.** LLM-as-judge for open-ended outputs, calibrated against human labels. Report agreement with humans before you trust it. Prefer pairwise or rubric-based judging over a bare 1 to 10 score.
- **Launch gate.** Which metric must not regress, by how much, on which slice. Every prompt or model change runs the gate in CI. See [design 08](designs/08-eval-pipeline-and-regression-gating.md).
- **Online signals.** Thumbs, edits, retries, escalation rate, task completion. Feed failures back into the eval set.
- **Training data,** if you fine-tune. Source, labeling, dedupe against the eval set to avoid leakage.

**Common miss:** "we would evaluate it with users." Too late and too noisy. Name the offline set and the gate.

## Step 4. Retrieval and model choice

**Goal:** pick the cheapest setup that clears the quality bar.

Decide in this order:

1. **Do you need retrieval?** If the knowledge changes, is private or is larger than a context window, yes.
2. **Retrieval design.** Hybrid (dense + BM25) by default, rerank with a cross-encoder, filter by metadata and permissions. Chunking that respects document structure. See [design 01](designs/01-rag-over-100m-documents.md).
3. **Model tier per task.** Small models for classification, routing, query rewriting and extraction. Large or reasoning models for hard generation and planning. Most production systems run 2 to 4 models, not one.
4. **Prompting vs fine-tuning.** Start with prompting and retrieval. Fine-tune when you need a format or style the model will not hold, lower latency from a smaller model, or lower cost at high volume. Fine-tuning rarely adds knowledge well. See [design 15](designs/15-fine-tuning-platform-and-lora-serving.md).
5. **Workflow vs agent.** If the steps are known, write a workflow with LLM calls at fixed points. Use an open-ended agent loop only when the path varies per input. Agents cost more, fail in more ways and are harder to evaluate.
6. **Structured output.** JSON schema or constrained decoding wherever a program reads the output.

**Common miss:** naming one frontier model for everything. Interviewers want to hear routing and why.

## Step 5. Serving and latency

**Goal:** draw the system and show the latency budget adds up.

Draw boxes for the request path and the async path separately. Then walk a single request with a time budget:

```
Gateway 10 ms → query rewrite 150 ms → retrieval 80 ms → rerank 100 ms → LLM TTFT 600 ms → stream
Total time to first token ≈ 940 ms
```

Topics to cover as the prompt requires:
- **Streaming** of tokens to the client. It changes perceived latency more than any optimization.
- **Parallelism.** Run independent calls together (dense and BM25 retrieval, guardrail checks alongside generation).
- **Timeouts, retries, fallbacks** to a second model or provider, with idempotency for anything that has side effects. See [design 07](designs/07-llm-gateway.md).
- **Caching** at three levels: provider prompt caching for shared prefixes, exact response caching, semantic caching with care. See [design 11](designs/11-semantic-caching.md).
- **Self-hosted serving** if relevant: continuous batching, paged KV cache, quantization, speculative decoding, autoscaling on queue depth. See [design 06](designs/06-llm-inference-serving.md).
- **State.** Where conversation history, agent state and long-running jobs live. Durable execution for agents that run minutes or hours.

**Common miss:** a diagram with no latency numbers, or an agent loop with no step limit.

## Step 6. Guardrails and safety

**Goal:** show you know where the system can be attacked or can cause harm, and that you put limits there.

- **Trust boundaries.** User input, retrieved documents, tool outputs and web pages are all untrusted. Any of them can carry a prompt injection.
- **Input checks.** Policy classifiers, PII detection, rate limits per user.
- **Output checks.** Grounding or citation checks, PII and secret scanning, format validation, policy classifiers.
- **Tool permissions.** Least privilege per tool, per user and per tenant. Human approval for irreversible actions. Spend and step limits.
- **Isolation.** Sandboxes for code execution, tenant separation for data and caches.
- **Latency cost.** Guardrails add 50 to 300 ms each when they call a model. Run them in parallel or on a cheaper model, and say which ones are blocking.

See [design 09](designs/09-guardrails-and-prompt-injection-defense.md) for the full layer.

**Common miss:** "we would add a guardrail" with no detail, or claiming a classifier stops prompt injection. It does not. Limiting what a hijacked model can do is the real defense.

## Step 7. Monitoring and cost

**Goal:** show you can run this in production and keep the bill under control.

Monitor:
- **System:** QPS, error rate by provider, TTFT and total latency p50/p95/p99, queue depth, GPU utilization, KV cache usage.
- **Quality:** online judge scores on sampled traffic, user feedback rate, refusal rate, escalation rate, hallucination flags.
- **Drift:** input distribution (new intents, new languages), retrieval hit rates, embedding drift after model changes.
- **Cost:** tokens and dollars per request, per feature, per tenant. Cost per successful task for agents.

Tracing matters more than in classic systems. Store the full trace (prompt, retrieved context, tool calls, output) for a sample of requests so you can debug a bad answer two weeks later. Mind PII when you store it.

Name the top 3 cost levers for your design. Usually: smaller model for easy traffic, shorter prompts (fewer chunks, trimmed history), and caching.

## Step 8. Scaling and evolution

**Goal:** show which part breaks first at 10x and what you would change.

Typical answers:
- Index outgrows RAM → disk-based ANN, quantization, two-stage retrieval.
- GPU cost grows linearly → routing, distillation to a smaller model, better batching, reserved capacity.
- Provider rate limits → multi-provider gateway, quota management, batch APIs for offline work.
- Eval cost grows with every release → tiered eval suites (fast smoke set on every PR, full set nightly).
- Multi-region → data residency, model availability per region, cache locality.

Close with one or two things you would build next and why. It shows you think in iterations.

## How the round is scored

Interviewers at different companies use different rubrics. The shape below is representative of what we see across public descriptions of AI system design rounds, not a copy of any company's internal rubric.

| Signal | Weak | Solid | Strong |
|---|---|---|---|
| Scoping | Starts drawing immediately | Asks questions, states assumptions | Finds the requirement that changes the design and says why |
| Estimates | None, or numbers with no use | Correct QPS and token math | Names the dominant cost or bottleneck and designs around it |
| Evals | "Users will tell us" | Offline eval set and metrics | Metrics per layer, calibrated judges, a CI gate, a feedback loop |
| Architecture | Boxes with arrows | Clear request and async paths | Latency budget per hop, failure handling on each hop |
| Model judgment | One big model for everything | Picks a model with reasons | Routes by task, knows when to fine-tune and when not to |
| Safety | Not mentioned | Lists guardrails | Trust boundaries, least privilege, limits on what a compromised model can do |
| Trade-offs | Presents one option | Mentions alternatives | Compares options with numbers and says when each wins |
| Communication | Monologue | Checks in | Drives the round, adapts when pushed, summarizes decisions |

## Phrases that signal seniority

Use them only when you mean them.

- "Before I tune prompts I want a labeled eval set, otherwise we are guessing."
- "Generation is the dominant cost here, so the index choice matters less than the number of chunks in the prompt."
- "I would start with a workflow and only add an agent loop where the path varies."
- "Retrieved documents are untrusted input. I would design as if one of them contains an injection."
- "That is a one-way door, so it gets human approval."
- "I would measure this on the p99, not the p50, because hosted LLM latency has a long tail."
- "Here is what I would cut if we had half the budget."

[Back to top](#the-ai-system-design-answer-framework)
