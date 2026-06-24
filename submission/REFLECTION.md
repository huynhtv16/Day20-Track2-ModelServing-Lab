# Reflection — Lab 20 (Personal Report)

**Họ Tên:** Huynh
**Cohort:** A20-k2
**Ngày submit:** 2026-06-24

---

## 1. Hardware spec (từ `00-setup/detect-hardware.py`)

- **OS:** Ubuntu/Linux 7.0.0-22-generic x86_64
- **CPU:** 11th Gen Intel(R) Core(TM) i5-11400H @ 2.70GHz
- **Cores:** 12 physical / 12 logical
- **CPU extensions:** AVX2, AVX-512
- **RAM:** 14.8 GB
- **Accelerator:** NVIDIA GeForce GTX 1650 Max-Q, 4096 MiB
- **llama.cpp backend đã chọn:** Vulkan (`Vulkan1`, NVIDIA)
- **Recommended model tier:** Qwen2.5-1.5B-Instruct Q4_K_M from probe; final local model used TinyLlama-1.1B Q4_K_M/Q2_K for reliable download and 4GB VRAM fit.

**Setup story**:

Probe đề xuất CUDA, nhưng máy có driver CUDA runtime mà thiếu `nvcc`/CUDA Toolkit để build. Mình chuyển sang native llama.cpp Vulkan, cài `libvulkan-dev`, `glslc`, `glslang`, build `llama-server`, rồi ép device `--device Vulkan1` để chạy trên GTX 1650 thay vì iGPU Intel.

---

## 2. Track 01 — Quickstart numbers (từ `benchmarks/01-quickstart-results.md`)

| Model | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode rate (tok/s) |
|---|--:|--:|--:|--:|--:|
| tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf | 801 | 51 / 68 | 8.1 / 8.4 | 297 / 327 / 334 | 123.4 |
| tinyllama-1.1b-chat-v1.0.Q2_K.gguf | 223 | 53 / 226 | 11.0 / 11.6 | 392 / 566 / 640 | 91.1 |

**Một quan sát** (≤ 50 chữ):

Q4_K_M vừa cho chất lượng tốt hơn vừa decode nhanh hơn trong run này. Q2_K load nhanh hơn nhưng TTFT P95 nhiễu hơn, nên Q4_K_M là lựa chọn hợp lý trên GTX 1650 4GB.

---

## 3. Track 02 — llama-server load test

| Concurrency | Total RPS | TTFB P50 (ms) | E2E P95 (ms) | E2E P99 (ms) | Failures |
|--:|--:|--:|--:|--:|--:|
| 10 | 1.83 | 4300 | 5600 | 6300 | 0 |
| 50 | 1.77 | 22000 | 29000 | 29000 | 0 |

**Batching observation**:

Ở concurrency 50, peak `llamacpp:n_busy_slots_per_decode` là 1.86, `requests_processing` là 2, và `requests_deferred` lên 48. Điều này cho thấy server giữ 2 slot decode đang bận và các request còn lại bị queue; P95 tăng mạnh vì GPU nhỏ đã bão hòa.

---

## 4. Track 03 — Milestone integration

- **N16 (Cloud/IaC):** stub: local native `llama-server` on `localhost:8080`.
- **N17 (Data pipeline):** stub: no external DAG/batch job connected.
- **N18 (Lakehouse):** stub: no Delta/Iceberg table connected.
- **N19 (Vector + Feature Store):** stub: in-memory `TOY_DOCS` keyword retrieval from `pipeline.py`.

**Nơi tốn nhiều ms nhất** trong pipeline:

- embed: 0 ms, because this skeleton does not run embedding.
- retrieve: 0.0 ms.
- llama-server: 3051.3 ms worst case, then 502.6 ms and 582.1 ms on the other two queries.

**Reflection** (≤ 60 chữ):

Bottleneck nằm hoàn toàn ở llama-server, đúng kỳ vọng vì retrieval chỉ là in-memory stub. Khi nối N19 thật, retrieval/embed mới đáng đo; hiện tại pipeline chủ yếu chứng minh OpenAI-compatible call, timings, và provenance context IDs.

---

## 5. Bonus — The single change that mattered most

**Change:** build native llama.cpp with Vulkan and run full GPU offload on NVIDIA using `--device Vulkan1 -ngl 99`.

**Before vs after**:

```text
before: ngl=0,  pp64=405.76 tok/s, tg32=56.99 tok/s
after:  ngl=99, pp64=602.26 tok/s, tg32=66.89 tok/s
speedup: ~1.48x prefill, ~1.17x decode
```

**Tại sao nó work**:

GTX 1650 có compute tốt hơn CPU cho phần matrix multiply của prefill, nên khi offload toàn bộ layer lên GPU, tốc độ prompt processing tăng rõ. Decode tăng ít hơn vì mỗi bước chỉ sinh một token và thường bị giới hạn bởi memory bandwidth, KV-cache access, và scheduling overhead.

Kết quả này cũng giải thích vì sao `nvidia-smi` đôi khi thấy GPU util thấp: các request ngắn có nhiều khoảng nghỉ giữa các burst compute, nhưng VRAM vẫn cho thấy `llama-server` đang nằm trên GPU.

---

## 6. (Optional) Điều ngạc nhiên nhất

Q2_K không nhanh hơn Q4_K_M trong quickstart run này. Trên laptop này, overhead/backend noise quan trọng hơn kích thước quant khi model đã đủ nhỏ để fit VRAM.

---

## 7. Self-graded checklist

- [x] `hardware.json` đã commit
- [x] `models/active.json` đã commit (hoặc paste path snapshot vào section 1)
- [x] `benchmarks/01-quickstart-results.md` đã commit
- [x] `benchmarks/02-server-results.md` (hoặc CSV từ `record-metrics.py`) đã commit
- [x] `benchmarks/bonus-*.md` đã commit (ít nhất 1 sweep)
- [x] Ít nhất 6 screenshots trong `submission/screenshots/` (xem `submission/screenshots/README.md`)
- [x] `make verify` exit 0 (chạy ngay trước khi push)- [ ] Repo trên GitHub ở chế độ **public**
- [x] Đã paste public repo URL vào VinUni LMS
