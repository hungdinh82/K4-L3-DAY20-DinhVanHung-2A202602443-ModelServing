# 01 - Measure: latency baseline

Model `Qwen3.5 0.8B` · host `Darwin-arm64` · llama.cpp `b10488`
Settings: `threads=8` `ngl=99` `ctx=2048`
`max_tokens=64` · warm-up discarded
Completed requests: `Q4_K_M` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 1078 | 63 / 78 | 10.8 / 12.4 | 740 / 858 / 858 | 92.6 |
| UD-Q2_K_XL | 0.39 | 1044 | 65 / 84 | 9.1 / 11.2 | 636 / 791 / 791 | 110.2 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decodes **1.19x faster** than `Q4_K_M` here, for 0.11 GB less on disk.

## Observation

`UD-Q2_K_XL` saves 0.11 GB (22%) and, in this repeat, decodes 1.19x faster:
110.2 versus 92.6 tok/s. I asked both variants the same Goodput@SLO question and
both produced an on-topic answer. On this run, Q2 is the better latency-and-size
choice; the difference from an earlier close run also shows that short benchmarks
have run-to-run variance, so I report this measured repeat rather than claiming a
universal quantization rule.
