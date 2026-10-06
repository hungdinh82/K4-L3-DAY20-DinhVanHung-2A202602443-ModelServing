# 01 - Measure: latency baseline

Model `Qwen3.5 0.8B` · host `Darwin-arm64` · llama.cpp `b10488`
Settings: `threads=8` `ngl=99` `ctx=2048`
`max_tokens=64` · warm-up discarded
Completed requests: `Q4_K_M` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 2111 | 67 / 248 | 9.6 / 10.2 | 675 / 716 / 716 | 103.8 |
| UD-Q2_K_XL | 0.39 | 2074 | 65 / 69 | 9.8 / 10.1 | 674 / 703 / 703 | 101.8 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` and `Q4_K_M` decode within 2% of each other here, for 0.11 GB difference on disk.

## Observation

`UD-Q2_K_XL` saves 0.11 GB (22%) but decodes slightly slower: 101.8 versus 103.8
tok/s (about 2%). I asked both variants the same Goodput@SLO question; both returned
an on-topic answer, but Q2 brings no latency gain on this M1 Pro. I would keep Q4
unless the 0.11 GB saving is necessary, because it preserves a small speed margin.
