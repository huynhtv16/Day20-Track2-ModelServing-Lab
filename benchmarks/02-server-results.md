# 02 — llama-server Load Results

Server command used:

```bash
BONUS-llama-cpp-optimization/llama.cpp/build/bin/llama-server \
  -m models/TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF/tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf \
  --host 127.0.0.1 --port 8080 \
  --device Vulkan1 -ngl 99 \
  -t 6 --parallel 2 --cont-batching --ctx-size 2048 --metrics
```

GPU evidence: `nvidia-smi` showed `llama-server` using about 696 MiB on the NVIDIA GeForce GTX 1650 Max-Q while serving.

| Concurrency | Requests | Failures | RPS | P50 (ms) | P95 (ms) | P99 (ms) |
|---:|---:|---:|---:|---:|---:|---:|
| 10 | 107 | 0 | 1.83 | 4300 | 5600 | 6300 |
| 50 | 104 | 0 | 1.77 | 22000 | 29000 | 29000 |

## Metrics

- 10 users: `requests_processing` peaked at 2, `requests_deferred` peaked at 8, `n_busy_slots_per_decode` reached 1.76, and `tokens_predicted_total` increased from 1412 to 11540.
- 50 users: `requests_processing` stayed at 2, `requests_deferred` peaked at 48, `n_busy_slots_per_decode` reached 1.86, and `tokens_predicted_total` increased from 11540 to 21748.

## Observation

The native server was GPU-backed, but this laptop GPU has only 4GB VRAM and the server was started with `--parallel 2`. At 50 users, the queue filled up (`requests_deferred` near 48), so throughput stayed roughly flat while tail latency rose sharply.
