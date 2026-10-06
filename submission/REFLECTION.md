# Reflection — Day 20 Lab (Personal Report)

> **Đây là báo cáo cá nhân.** Số liệu của bạn **không** so sánh được với bạn cùng lớp
> — chỉ so **before vs after trên chính máy bạn**. Rubric chấm độ rõ ràng của setup,
> đo lường và **lập luận**, không chấm tốc độ tuyệt đối.
>
> `make verify` sẽ fail nếu còn placeholder chưa điền. Đó là cố ý.

**Họ Tên:** Đinh Văn Hùng
**MSSV:** 2A202602443
**Cohort:** A20-K2
**Ngày submit:** 2026-10-06

---

## 1. Hardware & runtime  *(rubric 1, 2 — 10 điểm)*

> Từ `make probe`. Paste output hoặc điền tay.

- **OS:** macOS (Darwin 25.6.0, arm64)
- **CPU:** Apple M1 Pro
- **Cores:** 8 physical / 8 logical
- **CPU extensions:** NEON
- **RAM:** 16.0 GB
- **Accelerator:** Apple Metal
- **llama.cpp asset đã tải:** `llama-b10488-bin-macos-arm64.tar.gz`
- **Model đã dùng:** Qwen3.5 0.8B (`LAB_MODEL=qwen35-0.8b`)
- **Quantization:** Q4_K_M + UD-Q2_K_XL

**Chạy ở đâu:** laptop của tôi
_(Nếu dùng cloud fallback: nói rõ vì sao — RAM < 8 GB, setup fail, v.v. Không mất điểm.)_

**Setup story** (≤ 80 chữ): điều gì cần thay đổi để lab chạy trên máy bạn? Có bước
nào fail rồi phải workaround không?

Tôi dùng Qwen3.5 0.8B dù máy có đủ RAM cho model mặc định để giảm thời gian tải và
giữ vòng lặp đo nhanh. Python downloader gặp lỗi chứng chỉ TLS và cơ chế download bị
ngắt nhiều lần, nên tôi tải tay hai GGUF bằng `curl` với resume; runtime Metal vẫn là
binary prebuilt của lab.

---

## 2. Đo lường  *(rubric 3, 4, 5 — 20 điểm)*

> Paste bảng từ `benchmarks/01-quickstart-results.md` (`make bench` tự sinh).

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|---|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 1078 | 63 / 78 | 10.8 / 12.4 | 740 / 858 / 858 | 92.6 |
| UD-Q2_K_XL | 0.39 | 1044 | 65 / 84 | 9.1 / 11.2 | 636 / 791 / 791 | 110.2 |

**Quan sát** (≤ 60 chữ): 2-bit nhanh hơn bao nhiêu, và **có đáng không**? Bạn đã thử
hỏi cùng một câu trên cả hai (`make serve` vs `.venv/bin/python labs/02-serve/serve.py --compare`)
chưa? Chất lượng khác nhau thế nào?

Q2 nhỏ hơn 0.11 GB (22%) và ở lần chạy này decode nhanh hơn 1.19×: 110.2 so với 92.6
tok/s. Tôi hỏi cùng một câu Goodput@SLO trên cả hai bản và cả hai đều trả lời đúng
trọng tâm. Vì vậy ở lần đo này Q2 là lựa chọn tốt hơn về latency và dung lượng; chênh
lệch với lần đo trước cho thấy cần xem benchmark ngắn như một snapshot có biến thiên.

---

## 3. Serving under load  *(rubric 8, 9, 10 — 20 điểm)*

> Từ `benchmarks/02-server-results.md` (`make load-report`).

| Users | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|--:|--:|--:|--:|--:|--:|--:|
| 10 | 2.50 | 2600 | 4200 | 4300 | 6.8 | 0.0% |
| 50 | 2.56 | 17000 | 20000 | 21000 | 38.7 | 0.0% |

- **Offered load tăng 5×, throughput thực tăng:** 1.02×
- **P95 tăng:** 4.76×
- **Effective concurrency ở 50 users:** 38.7 so với `--parallel` = 4 slots

**Peak `llamacpp:n_busy_slots_per_decode`** (từ `make metrics` khi `make load-50` đang
chạy): 3.96 / 4 slots

**Saturation reading** (≤ 80 chữ): server của bạn bão hoà ở đâu, và **bằng chứng nào**
thuyết phục bạn? Nếu P95 tăng nhanh hơn RPS thì phần latency thêm đó là queue time hay
compute time — bạn biết bằng cách nào? Nếu bạn phải nâng goodput@SLO, bạn sẽ đổi knob
nào **trước**, và vì sao knob đó?

Server đã saturated ở 50 users: RPS chỉ tăng 1.02× khi offered load tăng 5×, trong khi
P95 tăng 4.76× lên 20 s. Effective concurrency 38.7 vượt xa 4 slot, metrics đồng thời
có peak 3.96 busy slots và 45 deferred request; đây là queue time. Tôi sẽ thử tăng
`--parallel` trước, nhưng chỉ giữ nếu goodput@SLO tăng mà P95 không tiếp tục phình lên.

---

## 4. Integration  *(rubric 12, 13 — 15 điểm)*

> Từ `make pipeline`. Nói thật cái nào real, cái nào stub — stub **không** mất điểm.

| Day | Piece | Real hay stub? |
|---|---|---|
| N16 Cloud/IaC | stub | không triển khai trong lab này |
| N17 Data pipeline | stub | không triển khai trong lab này |
| N18 Lakehouse | stub | không triển khai trong lab này |
| N19 Vector + features | stub | `TOY_DOCS` + keyword overlap |
| N20 Serving | `llama-server` | real |

**Latency split** (mean của 3 query, từ output của `pipeline.py`):

- embed: 0.0 ms (stub)
- retrieve: 0.0 ms (stub)
- llm: 1311.4 ms
- **stage chiếm nhiều nhất:** llm (100% của total)

**Reflection** (≤ 60 chữ): bottleneck ở đâu? Có khớp với kỳ vọng của bạn không? Nếu
phải giảm latency của pipeline này 2×, bạn sẽ tấn công vào đâu?

LLM là bottleneck, đúng kỳ vọng vì embedding và retrieval hiện là stub. Nếu cần giảm
latency 2×, tôi sẽ giảm decode LLM (giới hạn output tokens hoặc thử model/runtime nhanh
hơn); tối ưu stage 0.0 ms không làm tổng latency giảm đáng kể.

---

## 5. The single change that mattered most  *(rubric 11 — 10 điểm)*

> **Phần quan trọng nhất của report.** Không cần bonus track: `make tune` đã cho bạn
> một before/after thật (`benchmarks/01-tuning-tg128.md`). Đổi quantization,
> `LAB_N_CTX`, hay `--parallel` rồi đo lại cũng được.

**Change:** hạ số threads decode từ 8 xuống 4

```
before:  109.1 tok/s (`-t 8`)
after:   113.5 tok/s (`-t 4`)
speedup: 1.04×
```

**Tại sao nó work** (1–2 đoạn — đây là phần grader đọc kỹ nhất):

_Giải thích như đang nói với bạn ngồi cạnh. Bám vào **cơ chế**, không phải "vibes":
memory bandwidth? vector width? cache residency? scheduling? queueing? Nếu kết quả
**khác** với kỳ vọng từ deck — nói rõ, và giải thích vì sao. Grader thưởng điểm cho
lập luận đúng về một kết quả bất ngờ, hơn là một con số đẹp không được giải thích._

Knee nằm ở 4 threads, không phải 8 physical cores. Decode của model nhỏ này chủ yếu bị
giới hạn bởi memory bandwidth: sau 4 workers, các thread thêm vào tranh cùng bandwidth
thay vì tăng công việc hữu ích. Điều này thấy rõ ở `-t 16`, nơi oversubscription tăng
scheduling/cache pressure và throughput rơi còn 84.8 tok/s. Vì vậy tôi dùng 4 threads
cho server load test.

---

## 6. Bonus  *(optional — tối đa 10 điểm)*

> Bỏ trống nếu không làm. Xem `docs/bonus/README.md`. Đừng làm hết — **một** finding sâu
> ăn điểm hơn năm bảng nông.

**Đã làm:** _<B1 build-compare / B2 sweep nào / B4 challenge nào / B5 lựa chọn nào>_

**Numbers:**

```
before:  <số>
after:   <số>
speedup: <X.Y>×
```

**Điều này nói lên gì mà deck chưa nói:**

_(để trống nếu bạn không làm phần này)_

---

## 7. Điều làm bạn ngạc nhiên nhất  *(optional)*

_(1–2 câu. Không bắt buộc, nhưng grader đọc hết.)_

_(để trống nếu bạn không làm phần này)_

---

## 8. Self-check trước khi push

- [ ] `hardware.json` committed
- [ ] `models/active.json` committed
- [ ] `benchmarks/01-quickstart-results.md` committed (`make bench`)
- [ ] `benchmarks/01-tuning-tg128.md` committed (`make tune`)
- [ ] `benchmarks/02-server-results.md` committed (`make load-report`)
- [ ] `benchmarks/02-server-batching-u50.md` hoặc `-metrics-u50.csv` committed (`make metrics`)
- [ ] `benchmarks/locust-10_stats.csv` + `locust-50_stats.csv` committed (`make load-10` / `load-50`)
- [ ] `benchmarks/03-integration-results.md` committed (`make pipeline`)
- [ ] Mọi section **"required — replace this line"** trong các file `benchmarks/*.md`
      đã được thay bằng nhận xét của bạn
- [ ] 5 screenshots trong `submission/screenshots/`
- [ ] `make verify` → **exit 0**
- [ ] Repo tên đúng mẫu `K4-L3-DAY20-HoVaTen-MSSV-ModelServing` (xem `docs/SUBMISSION.md`)
- [ ] Repo GitHub ở chế độ **public**
- [ ] Đã push và paste public URL vào VinUni LMS **trước 23:59 (UTC+7) ngày làm lab**
- [ ] **Không** commit `models/*.gguf`, `runtime/` hay `.env` (đã có trong `.gitignore`)

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Private → grader không
xem được → 0 điểm.

---

## 9. Khai báo sử dụng AI  *(xem `docs/RULES.md` §3)*

Codex được dùng để đọc hiểu repo, chạy các lệnh setup/probe và hỗ trợ theo dõi tiến
trình thực hiện lab. Các số liệu benchmark, load test và kết luận trong báo cáo chỉ
được điền từ output chạy thật trên máy.
