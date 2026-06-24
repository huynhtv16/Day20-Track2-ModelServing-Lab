# Bonus — GPU Offload Sweep

Model: `tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf`

Backend: llama.cpp native Vulkan build on NVIDIA GeForce GTX 1650 Max-Q (`Vulkan1`).

| Mode | Backend | ngl | Device | pp64 tok/s | tg32 tok/s |
|---|---|---:|---|---:|---:|
| CPU-like path | Vulkan | 0 | host/no layer offload | 405.76 | 56.99 |
| GPU offload | Vulkan | 99 | Vulkan1 | 602.26 | 66.89 |

## Observation

Full GPU layer offload improved prompt processing from 405.76 to 602.26 tok/s (about 1.48x) and decode from 56.99 to 66.89 tok/s (about 1.17x). The larger gain is in prompt processing because prefill is more compute-heavy, while decode is often limited by memory bandwidth and per-token scheduling.
