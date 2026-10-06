# 02 - Serve: load test + saturation reading

Host `Darwin-arm64` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=4` ·
`ngl=99`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 138 | 2.50 | 2600 | 4200 | 4300 | 6.8 | 0.0% |
| 50 | 151 | 2.56 | 17000 | 20000 | 21000 | 38.7 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **1.02x** (20% of linear) |
| P95 latency | **4.76x** |
| Effective concurrency at 50 users | 38.7 vs `--parallel 4` slots (occupancy/slot ratio 9.67) |

**Saturated.** Throughput delivered only 1.02x for 5x the offered load, and effective concurrency (38.7) is at or above all 4 decode slots. Saturation sets in somewhere at or below 50 users; the load you added beyond that point became queue time rather than throughput.

Throughput moved 1.02x while P95 moved 4.76x. That gap is the goodput argument: past saturation you buy throughput by spending latency, and if your SLO is a P95 target then the requests you added are no longer being served within it. (This lab does not fix an SLO number for you -- pick one in your write-up and state how much goodput you keep at it.)

## Reading

The server is saturated by 50 users: offered load rose 5x but RPS only rose 1.02x,
while P95 increased 4.76x to 20 s. Effective concurrency reached 38.7 against four
slots, and the simultaneous metrics run saw 45 deferred requests. The extra latency
is therefore predominantly queue time. I would first test a larger `--parallel`
setting while monitoring the same gauges: it can admit more overlapping requests, but
I would retain it only if P95 and goodput@SLO improve rather than merely moving the
queue into compute contention.
