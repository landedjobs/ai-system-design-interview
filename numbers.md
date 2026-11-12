# Numbers to know for AI system design

[Back to README](README.md) · [Answer framework](framework.md) · [All designs](README.md#the-15-designs)

Interviewers do not expect exact figures. They expect you to know the order of magnitude, show the arithmetic, and say which assumption would change your design. Everything on this page is an estimate or a published spec. Hardware specs come from vendor datasheets. Prices move every quarter, so treat every dollar figure as a range and check current pricing before you quote it in a real design doc.

## Contents

- [Tokens and text](#tokens-and-text)
- [Time and rate conversions](#time-and-rate-conversions)
- [GPU specs](#gpu-specs)
- [Model weight memory](#model-weight-memory)
- [KV cache memory](#kv-cache-memory)
- [Prefill and decode speed](#prefill-and-decode-speed)
- [Embedding and index storage](#embedding-and-index-storage)
- [Latency of common calls](#latency-of-common-calls)
- [Cost per million tokens](#cost-per-million-tokens)
- [Self-hosted cost per token](#self-hosted-cost-per-token)
- [Training and fine-tuning memory](#training-and-fine-tuning-memory)
- [Worked mini-examples](#worked-mini-examples)

## Tokens and text

| Unit | Rough size | Note |
|---|---|---|
| 1 token (English) | ~4 characters, ~0.75 words | BPE tokenizers. Code and non-English text use more tokens per word |
| 1 page of prose | ~500 words, ~650 to 700 tokens | Dense PDFs with tables run higher |
| 1 short chat turn | 20 to 100 tokens | User message |
| Typical RAG prompt | 2K to 8K tokens | 5 to 10 chunks plus instructions |
| Agent step with tool results | 5K to 50K tokens | Grows every step unless you trim or summarize |
| 1 minute of speech | ~150 words, ~200 tokens | Useful for voice agents |

## Time and rate conversions

| Fact | Value |
|---|---|
| Seconds per day | 86,400 (round to 100K for quick math) |
| 1M requests/day | ~12 per second average |
| 100M requests/day | ~1,150 per second average |
| Peak to average | Plan for 2x to 5x depending on time zones |
| 99.9% availability | ~43 minutes of downtime per month |
| 99.99% availability | ~4.3 minutes per month |

## GPU specs

From NVIDIA datasheets, SXM variants, dense (no sparsity). Round these in the interview.

| GPU | HBM | Memory bandwidth | BF16 dense | FP8 dense |
|---|---|---|---|---|
| A100 80GB | 80 GB | ~2.0 TB/s | ~312 TFLOPS | not supported |
| H100 | 80 GB | ~3.35 TB/s | ~990 TFLOPS | ~1,980 TFLOPS |
| H200 | 141 GB | ~4.8 TB/s | ~990 TFLOPS | ~1,980 TFLOPS |
| L4 | 24 GB | ~0.3 TB/s | ~120 TFLOPS | ~240 TFLOPS |

NVLink on H100 is ~900 GB/s per GPU, which is why tensor parallelism stays inside one 8-GPU node. Real kernels reach 40 to 60% of peak FLOPs in prefill and 60 to 80% of peak bandwidth in decode (estimates).

## Model weight memory

Weights = parameters × bytes per parameter.

| Model size | BF16 (2 B) | FP8 / INT8 (1 B) | INT4 (~0.5 B) |
|---|---|---|---|
| 8B | ~16 GB | ~8 GB | ~4 to 5 GB |
| 70B | ~140 GB | ~70 GB | ~35 to 40 GB |
| 405B | ~810 GB | ~405 GB | ~200 to 230 GB |

A 70B model in BF16 does not fit on one 80 GB GPU. It needs 2 GPUs for weights alone and more for KV cache, so 4 is the usual minimum for real batch sizes. In FP8 it fits on 1 H200 or 2 H100s with room for cache.

## KV cache memory

The formula to memorize:

```
KV bytes per token = 2 (K and V) × layers × KV heads × head dim × bytes per value
```

| Model (published config) | Layers | KV heads | Head dim | Per token, BF16 | 8K context | 128K context |
|---|---|---|---|---|---|---|
| Llama 3 8B | 32 | 8 | 128 | 128 KiB | 1 GiB | 16 GiB |
| Llama 3 70B | 80 | 8 | 128 | 320 KiB | 2.5 GiB | 40 GiB |
| Same 70B without GQA (64 KV heads) | 80 | 64 | 128 | 2.5 MiB | 20 GiB | 320 GiB |

The last row is why grouped-query attention exists. An FP8 KV cache halves every number.

**How many concurrent sequences fit?** On 4 × H100 (320 GB) serving a 70B model in BF16: 320 minus 140 for weights minus ~20 GB of overhead ≈ 160 GB for KV. At 2.5 GiB per 8K sequence, that is ~60 full-length sequences. Most requests are shorter than the max, and PagedAttention allocates by block instead of by max length, so real concurrency is several times higher. Say this out loud. It is the reason paged KV caches matter.

## Prefill and decode speed

**Prefill** (reading the prompt) is compute-bound.

```
Prefill FLOPs ≈ 2 × parameters × prompt tokens
```

70B model, 2,000-token prompt: 2 × 70e9 × 2,000 = 2.8e14 FLOPs. One H100 at ~50% of 990 TFLOPS ≈ 0.57 s. Split over 4 GPUs with tensor parallelism ≈ 0.15 to 0.2 s. These are estimates, and attention cost adds more at long context.

**Decode** (one token at a time) is memory-bandwidth bound at small batch sizes. Each step reads all weights.

```
Max tokens/s per sequence at batch 1 ≈ memory bandwidth / weight bytes
```

| Setup | Weight bytes | Bandwidth | Upper bound |
|---|---|---|---|
| 8B BF16, 1 × H100 | 16 GB | 3.35 TB/s | ~200 tok/s |
| 70B FP8, 2 × H100 | 70 GB | 6.7 TB/s | ~95 tok/s |
| 70B BF16, 4 × H100 | 140 GB | 13.4 TB/s | ~95 tok/s |

Real numbers land at 50 to 80% of these bounds. Batching many sequences into one step reuses the same weight read, so aggregate throughput per GPU rises close to linearly with batch size until KV memory or compute runs out. That is the whole economics of LLM serving. See [design 06](designs/06-llm-inference-serving.md).

## Embedding and index storage

| Format | Bytes per vector (1,024 dims) | 1M vectors | 1B vectors |
|---|---|---|---|
| float32 | 4 KB | 4 GB | 4 TB |
| float16 | 2 KB | 2 GB | 2 TB |
| int8 scalar quantized | 1 KB | 1 GB | 1 TB |
| Product quantization (64 subvectors) | 64 B | 64 MB | 64 GB |
| Binary | 128 B | 128 MB | 128 GB |

Other dimensions scale linearly: 768 dims is 3 KB in float32, 1,536 dims is 6 KB.

HNSW graph links add roughly `2 × M × 4 bytes` per vector on the base layer. At M = 16 that is ~128 B, at M = 32 ~256 B. Upper layers add a few percent.

Quantized indexes lose some recall. The standard fix is to search the compressed index for the top few hundred and rescore with full-precision vectors.

## Latency of common calls

Typical ranges seen in practice. Every provider and load level differs, so measure your own.

| Operation | p50 | p99 |
|---|---|---|
| Same-region network round trip | ~0.5 ms | 1 to 2 ms |
| Cross-continent round trip | 70 to 150 ms | 200+ ms |
| Redis or in-memory KV get | < 1 ms | 1 to 5 ms |
| HNSW search, one in-memory shard | 1 to 5 ms | 10 to 20 ms |
| Embedding API call, one short query | 20 to 100 ms | 200 to 500 ms |
| Cross-encoder rerank, 100 candidates on GPU | 30 to 100 ms | 150 to 300 ms |
| Hosted LLM time to first token, short prompt | 0.2 to 1 s | 2 to 10 s |
| Hosted LLM output speed | 30 to 200 tokens/s | drops under load |
| Reasoning model thinking time | seconds | minutes |
| Streaming ASR final transcript after end of speech | 100 to 300 ms | 500 ms+ |
| TTS time to first audio byte | 75 to 300 ms | 500 ms+ |

Two rules follow. First, an LLM call costs more latency than everything else in a non-voice request put together, so count LLM calls on the critical path. Second, the p99 of a hosted LLM is wide. Design timeouts, hedged requests and fallbacks around the p99, not the p50 ([design 07](designs/07-llm-gateway.md)).

## Cost per million tokens

List prices as of 2025 to 2026 fell into rough bands. They change often. Check current pricing on the provider's page before you use a number.

| Tier | Input, per 1M tokens | Output, per 1M tokens |
|---|---|---|
| Small and fast models | $0.05 to $0.50 | $0.20 to $2 |
| Mid-tier models | $0.50 to $3 | $2 to $15 |
| Frontier models | $2 to $15 | $10 to $75 |
| Embedding models | $0.01 to $0.15 | n/a |

Patterns that hold across providers:
- Output tokens cost 3x to 5x input tokens.
- Cached input (prompt caching) is discounted, often 50 to 90%.
- Batch APIs with a 24-hour window are often around 50% off.
- Reasoning models bill thinking tokens as output tokens, so a short answer can cost like a long one.

## Self-hosted cost per token

```
$ per 1M tokens = GPU $ per hour / (aggregate tokens per second × 3,600) × 1,000,000
```

Example (all estimates): an H100 at $3 per hour serving an 8B model at 3,000 aggregate output tokens/s gives 3 / (3,000 × 3,600) × 1e6 ≈ **$0.28 per 1M tokens**. At 30% average utilization the effective price is ~$0.93. Utilization is the number that decides whether self-hosting beats an API.

GPU rental ranged from ~$2 to ~$10 per H100-hour in 2025 to 2026 depending on provider, region and commitment. Check current pricing.

## Training and fine-tuning memory

| Method | Memory per parameter (rough) | 8B model | 70B model |
|---|---|---|---|
| Full fine-tune, mixed precision Adam | ~16 bytes (weights, grads, fp32 master, 2 Adam states) | ~128 GB | ~1.1 TB |
| LoRA on BF16 base | ~2 bytes + small adapter states | ~16 to 20 GB | ~140 to 160 GB |
| QLoRA on 4-bit base | ~0.5 to 0.6 bytes + adapter states | ~6 to 10 GB | ~40 to 48 GB |

Activations come on top and depend on sequence length and batch size. Gradient checkpointing trades ~30% more compute for much less activation memory. LoRA adapters are small: rank 16 on the attention projections of an 8B model is tens of MB. That is why one GPU can serve hundreds of them ([design 15](designs/15-fine-tuning-platform-and-lora-serving.md)).

## Worked mini-examples

Practice these until you can do them in under a minute each.

**1. Chat app, 5M requests/day, 1,500 tokens in, 300 out, mid-tier model at $1 in and $5 out per 1M.**
Input: 5M × 1,500 = 7.5B tokens = $7,500/day. Output: 5M × 300 = 1.5B = $7,500/day. About $15K/day or $450K/month. Prompt caching on a 1,000-token shared system prompt at a 90% discount saves roughly $4,500/day.

**2. Can one H100 serve Llama 3 8B for 200 concurrent 4K-token chats?**
Weights 16 GB. KV at 128 KiB × 4,096 = 512 MiB per chat. 200 × 0.5 GiB = 100 GiB. Total ~116 GB. No. Use FP8 KV (50 GiB, total ~66 GB, fits with tight headroom) or two GPUs.

**3. How big is the index for 50M support articles' chunks?**
50M articles × 4 chunks = 200M vectors. 768 dims float32 = 3 KB. 600 GB raw, 150 GB int8, plus ~50 GB of HNSW links at M = 32. Fits in RAM across 4 to 6 machines.

**4. Voice agent budget.**
Endpointing 200 ms + ASR final 150 ms + LLM first token 350 ms + TTS first byte 150 ms + network 100 ms ≈ 950 ms. Over an 800 ms target, so cut endpointing with a turn-detection model and start TTS on the first sentence. See [design 13](designs/13-voice-agent.md).

## Sources for the specs

- [NVIDIA H100 product page](https://www.nvidia.com/en-us/data-center/h100/) for H100 bandwidth and FLOPs.
- [NVIDIA A100 product page](https://www.nvidia.com/en-us/data-center/a100/) for A100 specs.
- [Llama 3 model card](https://github.com/meta-llama/llama3/blob/main/MODEL_CARD.md) for layer and head counts.
- [Efficient Memory Management for LLM Serving with PagedAttention](https://arxiv.org/abs/2309.06180) (Kwon et al.) for KV cache waste and paging.
- [Transformer Inference Arithmetic](https://kipp.ly/transformer-inference-arithmetic/) (kipply) for the FLOPs and bandwidth reasoning used above.

[Back to top](#numbers-to-know-for-ai-system-design)
