<a name="top"></a>

<div align="center">

<a href="https://www.landed.jobs/?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=repo-home"><picture><source media="(prefers-color-scheme: dark)" srcset="https://static.b100x.ai/github-repos/images/brand/landed-wordmark-dark.png"><img src="https://static.b100x.ai/email/landed-wordmark.png" alt="Landed" width="200"></picture></a>

<img src="https://static.b100x.ai/github-repos/images/ai-system-design-interview/banner.svg" alt="AI System Design Interview" width="100%">

[![Stars](https://img.shields.io/github/stars/landedjobs/ai-system-design-interview?style=social)](https://github.com/landedjobs/ai-system-design-interview)
[![License: MIT](https://img.shields.io/badge/License-MIT-6C2BD9.svg)](LICENSE)
[![Updated](https://img.shields.io/badge/updated-2026--11-00A86B)](#the-15-designs)
[![Designs](https://img.shields.io/badge/worked%20designs-15-6C2BD9)](designs/)
[![Diagrams](https://img.shields.io/badge/mermaid%20diagrams-15-6C2BD9)](designs/)
[![Visit Landed](https://img.shields.io/badge/Visit-Landed-6C2BD9?logo=rocket&logoColor=white)](https://www.landed.jobs/?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=repo-home)

**15 worked LLM and GenAI system design answers, each with estimates, a diagram, an eval plan and trade-offs, plus one answer framework and the numbers to know.**

*Maintained by [Landed](https://www.landed.jobs/?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=repo-home): daily AI-native job matches, agent help with every application, and mock-interview prep.*

</div>

<!-- landed-apply-cta -->
<table><tr><td>
<b>⚡ <a href="https://app.landed.jobs/signup?from=github&amp;cta=github-guide-apply&amp;item=ai-system-design-interview&amp;utm_source=github&amp;utm_medium=referral&amp;utm_campaign=github-ai-system-design-interview&amp;utm_content=guide-apply">Auto-apply to AI engineer jobs that match you →</a></b><br>
Upload your resume once. <a href="https://www.landed.jobs">landed</a> finds the roles that fit you and applies within hours of them going live, so you actually hear back.
</td></tr></table>


---

The AI system design round is now in most AI engineer, ML engineer and applied scientist loops, and it is the round people prepare for worst. Classic system design guides teach you to shard a database. They do not teach you how big a KV cache gets, how to gate a prompt change on an eval set, or what to do when a retrieved PDF tells your agent to email the customer list to a stranger.

This repo is a set of full answers. Each design walks one prompt the way a strong candidate would in 45 minutes: clarifying questions, requirements with numbers, estimates worked step by step, an architecture diagram, an eval plan, a trade-offs table, failure modes, what changes at 10x, and the follow-ups interviewers push on. Every number is a labeled estimate with its assumptions shown, so you can redo the math with the interviewer's numbers.

## Contents

- [What the round tests](#what-the-ai-system-design-round-tests)
- [The answer framework](#the-answer-framework)
- [The 15 designs](#the-15-designs)
- [The 45-minute round](#the-45-minute-round)
- [Common failure modes](#common-failure-modes)
- [Numbers to know](#numbers-to-know)
- [How to study with this repo](#how-to-study-with-this-repo)
- [Other resources](#other-resources)
- [FAQ](#faq)
- [Contributing](#contributing)

## What the AI system design round tests

The round looks like classic system design with a model in the middle. It is scored on different things. From public interview write-ups and the company guides we maintain, interviewers look for five signals:

1. **Scoping with numbers.** Can you turn "design a support bot" into requirements you could check: TTFT under 1 s, 95% faithfulness, under $0.02 per conversation?
2. **Knowing how you will measure quality.** Eval sets, metrics per layer, LLM judges you have calibrated, and a gate that blocks a bad prompt change. This is the step most candidates skip, and it costs the most points.
3. **Model and retrieval judgment.** Hybrid retrieval and reranking, small models for routing and extraction, a big model only where it earns its cost, and a clear view on when to fine-tune.
4. **Serving, latency and cost.** A latency budget per hop that adds up. Knowing that generation usually dominates the bill. Streaming, caching, batching, fallbacks.
5. **Safety and failure handling.** Treating retrieved text and tool output as untrusted, limiting what the model can do, and handling the long tail of slow or failed model calls.

Here is the change from five years ago. The hard questions are now about the model's behavior (is it right, is it safe, what does it cost) more than about the database.

## The answer framework

Use the same eight steps on every prompt. The full rubric, with what "strong" looks like at each step, is in **[framework.md](framework.md)**.

| # | Step | What you produce | Typical time |
|---|---|---|---|
| 1 | Requirements | Functional list, non-functional targets as numbers, out-of-scope list | 4 to 5 min |
| 2 | Estimates | QPS, tokens per day, storage, dollars or GPUs per day, the dominant term | 3 to 5 min |
| 3 | Data and evals | Eval set, metrics per layer, launch gate, online signals | 5 to 7 min |
| 4 | Retrieval and model choice | Retrieval design, model tier per task, prompting vs fine-tuning | 6 to 8 min |
| 5 | Serving and latency | Diagram, latency budget per hop, streaming, caching, fallbacks | 6 to 8 min |
| 6 | Guardrails | Trust boundaries, input and output checks, tool permissions | 3 to 5 min |
| 7 | Monitoring and cost | Dashboards, traces, cost per request, top 3 cost levers | 3 to 4 min |
| 8 | Scaling | What breaks at 10x and what you change | 3 to 5 min |

## The 15 designs

Each file follows the same sections, so you can compare designs side by side. Difficulty is our judgment of how much depth a strong answer needs. "Companies that ask it" lists companies whose products center on that system; each link goes to that company's interview guide, and each design file lists more.

| # | Design | Difficulty | Key trade-off | Companies that ask it |
|---|---|---|---|---|
| 01 | [RAG over 100M documents](designs/01-rag-over-100m-documents.md) | Hard | Recall vs index and prompt cost | [Cohere](https://www.landed.jobs/resources/guides/cohere-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Glean](https://www.landed.jobs/resources/guides/glean-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Pinecone](https://www.landed.jobs/resources/guides/pinecone-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index) |
| 02 | [Enterprise search with permissions](designs/02-enterprise-search-with-permissions.md) | Hard | Permission freshness vs recall and latency | [Glean](https://www.landed.jobs/resources/guides/glean-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Microsoft](https://www.landed.jobs/resources/guides/microsoft-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Notion](https://www.landed.jobs/resources/guides/notion-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index) |
| 03 | [Customer support agent](designs/03-customer-support-agent.md) | Medium | Automation rate vs policy safety | [Sierra](https://www.landed.jobs/resources/guides/sierra-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Decagon](https://www.landed.jobs/resources/guides/decagon-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Intercom](https://www.landed.jobs/resources/guides/intercom-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index) |
| 04 | [Coding agent platform](designs/04-coding-agent-platform.md) | Hard | Autonomy and context vs cost and safety per task | [Cursor](https://www.landed.jobs/resources/guides/anysphere-cursor-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Cognition](https://www.landed.jobs/resources/guides/cognition-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Replit](https://www.landed.jobs/resources/guides/replit-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index) |
| 05 | [Multi-tenant agent platform with MCP](designs/05-multi-tenant-agent-platform-mcp.md) | Hard | Tenant flexibility vs isolation and predictable cost | [Anthropic](https://www.landed.jobs/resources/guides/anthropic-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [OpenAI](https://www.landed.jobs/resources/guides/openai-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [LangChain](https://www.landed.jobs/resources/guides/langchain-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index) |
| 06 | [LLM inference serving at scale](designs/06-llm-inference-serving.md) | Hard | Throughput vs per-user latency | [NVIDIA](https://www.landed.jobs/resources/guides/nvidia-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Together AI](https://www.landed.jobs/resources/guides/together-ai-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Fireworks AI](https://www.landed.jobs/resources/guides/fireworks-ai-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index) |
| 07 | [LLM gateway](designs/07-llm-gateway.md) | Medium | Added hop vs central control of cost and reliability | [Portkey](https://www.landed.jobs/resources/guides/portkey-ai-india-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Vercel](https://www.landed.jobs/resources/guides/vercel-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Databricks](https://www.landed.jobs/resources/guides/databricks-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index) |
| 08 | [Eval pipeline and regression gating](designs/08-eval-pipeline-and-regression-gating.md) | Medium | Coverage and statistical power vs run cost | [Scale AI](https://www.landed.jobs/resources/guides/scale-ai-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Arize AI](https://www.landed.jobs/resources/guides/arize-ai-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Weights & Biases](https://www.landed.jobs/resources/guides/weights-biases-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index) |
| 09 | [Guardrails and prompt-injection defense](designs/09-guardrails-and-prompt-injection-defense.md) | Hard | Blast-radius limits vs latency and false refusals | [Anthropic](https://www.landed.jobs/resources/guides/anthropic-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [OpenAI](https://www.landed.jobs/resources/guides/openai-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Wiz](https://www.landed.jobs/resources/guides/wiz-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index) |
| 10 | [AI search engine like Perplexity](designs/10-ai-search-engine.md) | Hard | Own index vs search API cost and dependence | [Perplexity](https://www.landed.jobs/resources/guides/perplexity-ai-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Google DeepMind](https://www.landed.jobs/resources/guides/google-deepmind-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [xAI](https://www.landed.jobs/resources/guides/xai-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index) |
| 11 | [Semantic caching](designs/11-semantic-caching.md) | Medium | Hit rate vs false hits | [Redis](https://www.landed.jobs/resources/guides/redis-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Portkey](https://www.landed.jobs/resources/guides/portkey-ai-india-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Databricks](https://www.landed.jobs/resources/guides/databricks-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index) |
| 12 | [Document extraction pipeline](designs/12-document-extraction-pipeline.md) | Medium | Automation rate vs field-level accuracy | [Harvey](https://www.landed.jobs/resources/guides/harvey-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Snowflake](https://www.landed.jobs/resources/guides/snowflake-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [UiPath](https://www.landed.jobs/resources/guides/uipath-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index) |
| 13 | [Voice agent with a latency budget](designs/13-voice-agent.md) | Hard | Response latency vs turn-taking accuracy | [ElevenLabs](https://www.landed.jobs/resources/guides/elevenlabs-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Sierra](https://www.landed.jobs/resources/guides/sierra-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Decagon](https://www.landed.jobs/resources/guides/decagon-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index) |
| 14 | [Recommendations with LLM reranking](designs/14-recommendations-with-llm-reranking.md) | Hard | LLM quality vs a 200 ms serving budget | [Meta](https://www.landed.jobs/resources/guides/meta-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Amazon](https://www.landed.jobs/resources/guides/amazon-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Flipkart](https://www.landed.jobs/resources/guides/flipkart-india-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index) |
| 15 | [Fine-tuning platform and LoRA serving](designs/15-fine-tuning-platform-and-lora-serving.md) | Hard | Per-tenant quality vs GPU cost of many variants | [Together AI](https://www.landed.jobs/resources/guides/together-ai-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Fireworks AI](https://www.landed.jobs/resources/guides/fireworks-ai-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index), [Databricks](https://www.landed.jobs/resources/guides/databricks-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=readme-index) |

**Where to start.** If you have one evening, do [01](designs/01-rag-over-100m-documents.md), [03](designs/03-customer-support-agent.md) and [06](designs/06-llm-inference-serving.md). Those three cover retrieval, agents and serving, and most other prompts are a remix of them. For infra roles, add [07](designs/07-llm-gateway.md) and [15](designs/15-fine-tuning-platform-and-lora-serving.md). For applied AI and product roles, add [08](designs/08-eval-pipeline-and-regression-gating.md) and [09](designs/09-guardrails-and-prompt-injection-defense.md).

## The 45-minute round

A timeline that works for most prompts. Adjust when the interviewer steers you.

| Minute | What you do | What to say out loud |
|---|---|---|
| 0 to 5 | Clarify and scope | "Before I draw anything, a few questions about users, scale and what a wrong answer costs." |
| 5 to 10 | Requirements and estimates | "So roughly 350 QPS at peak and 45B input tokens a day. Generation is the cost driver." |
| 10 to 15 | Eval plan | "Here is how I will know it works before we launch, and what blocks a release." |
| 15 to 28 | High-level design and walkthrough | Draw the request path and async path. Walk one request with a latency budget. |
| 28 to 36 | Deep dive the interviewer picks | Usually retrieval quality, serving cost, agent reliability or safety. |
| 36 to 41 | Failure modes, guardrails, monitoring | "Three ways this fails in production, and what catches each one." |
| 41 to 45 | Scaling and wrap-up | "At 10x the index stops fitting in RAM, so I would move to..." Then summarize your key decisions. |

Two habits help more than any amount of content. Check in at each transition ("does this scope work for you?"). And when you make a choice, name the alternative and why you did not pick it.

## Common failure modes

What sinks candidates in this round, roughly in order of how often they come up in public interview write-ups and interviewer advice.

| Failure | What it looks like | Fix |
|---|---|---|
| No eval plan | "We'd A/B test it" as the only quality answer | Name the offline eval set, metrics per layer and the CI gate before the diagram |
| Diagram in minute two | Boxes before any requirement has a number | Spend the first 5 minutes on scope and estimates |
| One giant model for everything | Frontier model for routing, rewriting and answering | Route by task: small models for classification and extraction |
| Agent where a workflow fits | Open-ended loop for a fixed 4-step process | Use a workflow; use an agent loop only where the path varies |
| Hand-waved retrieval | "Put it in a vector DB" | Chunking, hybrid search, reranking, recall@k on a labeled set |
| No numbers | No QPS, tokens, cost or latency budget | Do the token math out loud and name the dominant cost |
| Guardrails as a buzzword | "We'll add guardrails" with no detail | Trust boundaries, least-privilege tools, human approval for irreversible actions |
| Ignoring the long tail | Designing for p50 LLM latency | Timeouts, hedging, fallbacks to a second model or provider, sized for p99 |
| No cost story | Never mentions what a request costs | Cost per request, top 3 levers, caching |
| Monologue | 20 minutes without a check-in | Ask at each step whether to go deeper or move on |

## Numbers to know

The short version. Full tables, derivations and worked mini-examples are in **[numbers.md](numbers.md)**. All figures are estimates or published specs; prices change often, so check current pricing.

| What | Rule of thumb |
|---|---|
| Tokens | 1 token ≈ 4 characters ≈ 0.75 English words. One page ≈ 650 to 700 tokens |
| Rate | 1M requests/day ≈ 12 QPS average. Plan for 2x to 5x at peak |
| Weights | Params × bytes. 70B in BF16 ≈ 140 GB, FP8 ≈ 70 GB, INT4 ≈ 35 to 40 GB |
| KV cache | 2 × layers × KV heads × head dim × bytes per token. Llama 3 70B ≈ 320 KiB/token in BF16, 2.5 GiB per 8K sequence |
| Decode speed bound | Memory bandwidth ÷ weight bytes. 8B BF16 on one H100 ≈ 200 tokens/s max at batch 1 |
| Prefill compute | ≈ 2 × params × prompt tokens FLOPs |
| H100 | 80 GB HBM, ~3.35 TB/s, ~990 TFLOPS BF16 dense |
| Embeddings | 1,024 dims float32 = 4 KB. 1B vectors = 4 TB raw, 1 TB int8, 128 GB binary |
| HNSW overhead | ~2 × M × 4 bytes per vector (~256 B at M = 32) |
| LLM latency (hosted) | TTFT p50 0.2 to 1 s, p99 2 to 10 s. Output 30 to 200 tokens/s |
| Retrieval latency | In-memory ANN 1 to 5 ms per shard. Cross-encoder rerank of 100 docs 30 to 100 ms on GPU |
| Price bands per 1M tokens | Small models $0.05 to $0.50 in. Frontier $2 to $15 in. Output 3x to 5x input |
| Discounts | Cached input often 50 to 90% off. Batch APIs often ~50% off |
| Self-hosted cost | GPU $/hr ÷ (tokens/s × 3,600) × 1M. Utilization decides whether it beats an API |

## How to study with this repo

1. **Read [framework.md](framework.md) once.** Then close it. You should be able to list the eight steps from memory.
2. **Pick a design and do it cold first.** Set a 45-minute timer, read only the problem statement, and answer out loud or on paper. Then compare against the worked answer.
3. **Redo the estimates with different numbers.** Change the QPS or corpus size by 10x and see which design decision flips. That is what the interviewer is checking.
4. **Answer every follow-up** in the "What interviewers probe" section of each design without looking.
5. **Practice with a partner** who pushes on one area for 10 minutes. Depth under pressure is where hire and no-hire split.

A week of prep that works: day 1 framework and numbers, days 2 to 5 three designs a day (one cold, two read), day 6 two mock rounds, day 7 redo the designs you struggled with.

## Other resources

Honest pointers to other good material. These complement this repo; none of them are ours unless marked.

**Repos**
- [themanojdesai/genai-llm-ml-case-studies](https://github.com/themanojdesai/genai-llm-ml-case-studies): a large index of company engineering blog posts on ML and LLM systems. Links rather than answers, so it pairs well with this repo for real-world examples.
- [neurarch-ai/awesome-llm-system-design](https://github.com/neurarch-ai/awesome-llm-system-design): a curated list of LLM system design resources.
- [ather-techie/ai-system-design-interview](https://github.com/ather-techie/ai-system-design-interview): another set of AI system design notes.
- [donnemartin/system-design-primer](https://github.com/donnemartin/system-design-primer): the classic system design reference. Still the best place for the non-AI fundamentals (load balancing, caching, sharding).
- [landedjobs/awesome-ai-engineer-interview](https://github.com/landedjobs/awesome-ai-engineer-interview) (ours): the full AI engineer loop, with questions per round and company breakdowns.
- [landedjobs/rag-engineer-interview-questions](https://github.com/landedjobs/rag-engineer-interview-questions) (ours): question bank focused on retrieval and RAG.

**Reading**
- [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) (Anthropic): the clearest write-up of workflows vs agents, which comes up in half of these designs.
- [Building A Generative AI Platform](https://huyenchip.com/2024/07/25/genai-platform.html) (Chip Huyen): a walk through the components of a production GenAI stack, in roughly the order you add them.
- [Patterns for building LLM-based systems and products](https://eugeneyan.com/writing/llm-patterns/) (Eugene Yan): evals, RAG, caching, guardrails and feedback as patterns.
- [What We Learned from a Year of Building with LLMs](https://applied-llms.org/): practitioners' lessons on evals, operations and strategy.

**Landed pages (ours)**
- [System design library](https://www.landed.jobs/resources/system-design?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=other-resources), including [Design ChatGPT](https://www.landed.jobs/resources/system-design/design-chatgpt?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=other-resources).
- Courses: [ML system design](https://www.landed.jobs/resources/courses/ml-system-design?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=other-resources), [AI infra interviews](https://www.landed.jobs/resources/courses/ai-infra-interviews?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=other-resources), [RAG systems](https://www.landed.jobs/resources/courses/rag-systems?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=other-resources), [Agents, evals and LLMOps](https://www.landed.jobs/resources/courses/agents-evals-llmops?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=other-resources).
- [Company interview guides](https://www.landed.jobs/resources/guides?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=other-resources) for 200 companies.

## FAQ

**Is the AI system design round different from the regular system design round?**
Yes, in what gets scored. You still need the classic pieces (APIs, storage, queues, caching, sharding). On top of that you are expected to plan evals, pick models per task, budget tokens and latency, and handle failures that only models have: hallucination, prompt injection, nondeterminism and a long latency tail. Some companies run both rounds; some replace the classic one for AI roles.

**Which roles get this round?**
AI engineer, ML engineer, applied scientist, AI infrastructure and increasingly backend and forward-deployed engineers at AI-first companies. Infra roles lean toward serving, gateways and fine-tuning platforms ([06](designs/06-llm-inference-serving.md), [07](designs/07-llm-gateway.md), [15](designs/15-fine-tuning-platform-and-lora-serving.md)). Product and applied roles lean toward RAG, agents and evals ([01](designs/01-rag-over-100m-documents.md), [03](designs/03-customer-support-agent.md), [08](designs/08-eval-pipeline-and-regression-gating.md)).

**Do I need to know GPU internals?**
For infra and serving roles, yes: KV cache math, batching, prefill vs decode, quantization, parallelism. For applied roles, you need the cost and latency consequences, not the kernels. [numbers.md](numbers.md) covers what both groups need.

**Are the numbers in these designs real?**
They are back-of-envelope estimates with the assumptions written out, plus published hardware specs. Interviewers care that your arithmetic is sound and that you know which number drives the design. Prices change every quarter; check current pricing before you put a dollar figure in a real design doc.

**Are these the exact questions companies ask?**
The prompts are representative. They match the systems these companies build and the topics in public interview reports, but we do not claim any company asks a given prompt word for word. The company interview guides linked from each design describe each company's actual rounds.

**Should I name specific vendors and tools?**
Name a category first ("a vector index with HNSW", "a cross-encoder reranker"), then a concrete example if it helps. Interviewers want to see that you understand the mechanism and could swap the vendor.

**How long should I spend preparing?**
With a classic system design base, about a week of focused practice covers the main patterns. See [How to study with this repo](#how-to-study-with-this-repo).

**Where do I find roles that run this round?**
The landed [AI engineer jobs repo](https://github.com/landedjobs/ai-engineer-jobs) lists live roles, and the [AI agent engineer jobs repo](https://github.com/landedjobs/ai-agent-engineer-jobs) covers agent-focused teams.

---

## Contributing

PRs and issues welcome: new designs, sharper estimates, corrections, better sources. Every number needs its assumptions, every source needs a working link, and every company claim needs to be honest about whether it is reported or representative. See **[CONTRIBUTING.md](CONTRIBUTING.md)** for the design template and quality bar.

---

### Part of the Landed AI-native jobs family

The umbrella maps every AI-native role; the siblings go deep per role.

- 🧭 [awesome-ai-native-jobs](https://github.com/landedjobs/awesome-ai-native-jobs): the umbrella that maps every AI-native job family
- 🎯 [awesome-ai-engineer-interview](https://github.com/landedjobs/awesome-ai-engineer-interview): the full AI engineer interview loop, round by round
- 🔎 [rag-engineer-interview-questions](https://github.com/landedjobs/rag-engineer-interview-questions): interview questions for RAG and retrieval roles
- 🚀 [ai-engineer-jobs](https://github.com/landedjobs/ai-engineer-jobs): 300 live AI engineer roles, auto-updated
- 🤖 [ai-agent-engineer-jobs](https://github.com/landedjobs/ai-agent-engineer-jobs): roles building agents
- 🧠 [llm-engineer-jobs](https://github.com/landedjobs/llm-engineer-jobs): LLM engineer roles
- 📘 [ai-interview-guides](https://github.com/landedjobs/ai-interview-guides): interview guides for 200 companies
- ❓ [ai-interview-questions](https://github.com/landedjobs/ai-interview-questions): 759 AI interview questions with answers
- 📦 [ai-engineer-portfolio-projects](https://github.com/landedjobs/ai-engineer-portfolio-projects): 80+ buildable portfolio projects

---

<div align="center">

### Stop spraying. Get **matched**, get **prepped**, get **Landed**.

[![Get Started](https://img.shields.io/badge/Get%20Started%20Free-→-6C2BD9?style=for-the-badge)](https://www.landed.jobs/?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=repo-home)

<sub>Maintained by [Landed](https://www.landed.jobs/?utm_source=github&utm_medium=referral&utm_campaign=github-ai-system-design-interview&utm_content=repo-home) · No affiliation with the companies named. Content MIT-licensed; linked resources retain their own licenses. Interview prompts are representative; numbers are labeled estimates.</sub>

<br/>

[![Star History Chart](https://api.star-history.com/svg?repos=landedjobs/ai-system-design-interview&type=Date)](https://star-history.com/#landedjobs/ai-system-design-interview&Date)

</div>
