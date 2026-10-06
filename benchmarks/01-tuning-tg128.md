# 01 - Tune: thread-count sweep

Model `Qwen3.5-0.8B-Q4_K_M.gguf` · host `Darwin-arm64` · llama.cpp `b10488`
CPU: **8 physical · 8 logical** cores · `ngl=99` · metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 107.0 | 94% |
| 4 | 113.5 | 100% |
| 8 | 109.1 | 96% |
| 16 | 84.8 | 75% |

**Best**: `-t 4` at 113.5 tok/s
**Slowest tested**: `-t 16` at 84.8 tok/s (1.34x spread)
**Against the physical-core default** (`-t 8`, 109.1 tok/s): 1.04x

Use this in your run:

```bash
LAB_N_THREADS=4 make bench
```

## Explanation

The knee is at 4 threads (113.5 tok/s), not at all 8 physical cores. Decode is
largely constrained by moving model weights through memory; after four workers, more
threads add contention for the same memory bandwidth rather than useful parallel work.
At 16 threads the process oversubscribes the 8-core CPU, increasing scheduling and
cache pressure, so throughput falls to 84.8 tok/s. I therefore used 4 threads for
the persistent serving run.
