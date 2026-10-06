# 03 - Integrate: RAG pipeline run

Host `Darwin-arm64` · llama.cpp `b10488` ·
retrieval backend: **keyword overlap** · 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 0.0 | 1420.7 | 1420.7 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.0 | 1039.9 | 1040.0 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.0 | 0.0 | 1473.5 | 1473.6 |

Mean per stage (ms): embed **0.0** · retrieve **0.0** ·
llm **1311.4** · total **1311.4**
Dominant stage: **llm** (100% of total)

## Answers returned

**Why is goodput more useful than raw throughput?**

> Based on the provided context, **Goodput** is more useful than raw throughput primarily because it **ignores SLOs (Service Level Objectives) at saturation**.

While raw throughput measures the total requests per second (which can be high even when the system is overloaded), Goodput specifically counts only requests that met the Target Time-to-Fill (TTFT) and Target Time-to-Poll (TPOT) targets. Thi

**What problem does PagedAttention actually solve?**

> PagedAttention solves the problem of **internal fragmentation in GPU memory** caused by storing the Key-Value (KV) cache in non-contiguous pages.

By using non-contiguous pages, the system avoids the wasted space that would otherwise exist if the cache were stored contiguously (like in standard L2 or L3 caches). This optimization is particularly valuable for GPUs, where the vast majority of memory

**When does splitting prefill and decode help?**

> Splitting prefill and decode helps when the **prefill operation is compute-bound** and the **decode operation is memory-bandwidth-bound**.

In this context, prefill is computationally expensive and requires significant processing power, while decoding is the bottleneck due to memory bandwidth limitations. By separating these steps, the system can:
1.  **Prefill efficiently**: Perform the heavy com


## Which N16-N19 pieces are real

N16 Cloud/IaC: stub. N17 data pipeline: stub. N18 lakehouse: stub. N19 vector and
features: stub; this lab uses the in-memory `TOY_DOCS` corpus and keyword-overlap
retrieval, with no embedding server. N20 is real: the pipeline calls local
`llama-server`. LLM time dominates at 1311.4 ms (100% of measured time), which is
expected for local generation while embed and retrieval are stubs. To halve latency I
would first reduce LLM decode cost (for example fewer generated tokens or a faster
model/runtime setting), because optimizing a 0.0 ms stub cannot materially change total
latency.
