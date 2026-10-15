# Example delivery contracts

Three synthetic, local-only contracts for running AI Delivery Doctor without any
live endpoint or credentials. All are `aidoc-v1`.

| File | What it demonstrates | Run with |
|---|---|---|
| [`acceptance.example.json`](acceptance.example.json) | Two required environment facts plus one optional key; all `PASS`/`WARN`, no blocker. | `python -m aidoc.cli doctor --config examples/acceptance.example.json` |
| [`broken.example.json`](broken.example.json) | One intentionally missing required artifact; `FAIL`, `FIRST BLOCKER` reported. Not a proven root cause. | `python -m aidoc.cli doctor --config examples/broken.example.json` |
| [`openai-compatible.example.json`](openai-compatible.example.json) | An optional local OpenAI-compatible endpoint check; `false`-required so a missing endpoint is `WARN`, not `FAIL`. | `python -m aidoc.cli doctor --config examples/openai-compatible.example.json` |

## Validation

Each example also passes the schema validator without running checks:

```bash
python -m aidoc.cli validate --config examples/acceptance.example.json
python -m aidoc.cli validate --config examples/broken.example.json
python -m aidoc.cli validate --config examples/openai-compatible.example.json
```

## Boundaries

No fixture in this directory contains secret values, private endpoints, private
IPs, or production logs. The `openai-compatible` example points at the loopback
placeholder `http://127.0.0.1:11434/v1`; it is not a live service.
