# 03 — Milestone Integration Notes

Command:

```bash
.venv/bin/python 03-milestone-integration/pipeline.py
```

| Query | Retrieved contexts | Retrieve (ms) | LLM (ms) | Total (ms) |
|---|---|---:|---:|---:|
| Why is goodput more useful than throughput? | n20-paged, n20-radix, n20-disagg | 0.0 | 3051.3 | 3051.4 |
| What problem does PagedAttention actually solve? | n20-paged, n20-radix, n20-disagg | 0.0 | 502.6 | 502.7 |
| When should I think about disaggregated serving? | n20-disagg, n20-paged, n20-radix | 0.0 | 582.1 | 582.1 |

## N16-N19 Status

- N16: stub, local-only native `llama-server` on `localhost:8080`.
- N17: stub, no external batch pipeline.
- N18: stub, no lakehouse table connected.
- N19: stub, `TOY_DOCS` in-memory retrieval with keyword overlap.

The integration proves the OpenAI-compatible serving call and response provenance format. The retrieval layer is intentionally still the lab skeleton stub.
