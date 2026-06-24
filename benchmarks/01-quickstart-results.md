# 01 — Quickstart Results

Settings: `n_threads=6`, `n_ctx=2048`, `n_batch=512`, `n_gpu_layers=99`.

| Model | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode rate (tok/s) |
|---|---:|---:|---:|---:|---:|
| tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf | 801 | 51 / 68 | 8.1 / 8.4 | 297 / 327 / 334 | 123.4 |
| tinyllama-1.1b-chat-v1.0.Q2_K.gguf | 223 | 53 / 226 | 11.0 / 11.6 | 392 / 566 / 640 | 91.1 |

## Observations

- Q4_K_M decoded faster than Q2_K on this Vulkan/GTX 1650 run: 123.4 tok/s vs 91.1 tok/s.
- Q2_K loaded faster and used a smaller file, but its P95 TTFT was noisier on this run.
- For this machine, Q4_K_M is the better default because it keeps better quality and still fits comfortably in 4GB VRAM.
