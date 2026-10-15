# Contributor Onboarding

A short, evidence-first onboarding guide for making a first contribution to
AI Delivery Doctor. Everything below uses synthetic, local fixtures only — no
external credentials, endpoints, or production logs — and preserves the
project's deterministic, reviewable design.

## What a contribution is

A good contribution makes it possible to answer, for one bounded delivery
question:

1. **What real delivery problem does it solve?**
2. **What observable fact does it establish?**
3. **What changed?**
4. **How was it verified?**
5. **What does the result *not* prove?**

Keep each contribution small and reviewable. If the requested behavior is
ambiguous, take the smallest reviewable interpretation and state the assumption.

## Local, minimal example

This repository ships three synthetic contracts in `examples/`:

| File | What it shows |
|---|---|
| [`examples/acceptance.example.json`](../examples/acceptance.example.json) | A passing self-check: two required environment facts plus one optional key. |
| [`examples/broken.example.json`](../examples/broken.example.json) | An intentionally failing required check, used to see how a blocker is reported. |
| [`examples/openai-compatible.example.json`](../examples/openai-compatible.example.json) | An optional local, OpenAI-compatible endpoint check that stays `false`-required. |

All three are synthetic. The `openai-compatible` example points at a loopback
placeholder (`http://127.0.0.1:11434/v1`) and is **not** a live endpoint.

## Evidence-first design, in one paragraph

AI Delivery Doctor separates *what a reader wants verified* from *what was
actually observed*. A check records one bounded observation — an executable
exists, a file can be read, an environment variable is set — and reports
`PASS`, `WARN`, or `FAIL` with a narrow meaning. A `FAIL` is a reported
transition, not a proven root cause. The deterministic core owns the observed
facts; reasoning about the system may help choose what to verify, but it never
rewrites what was observed. See [ARCHITECTURE](ARCHITECTURE.md) and
[PHILOSOPHY](PHILOSOPHY.md).

## Run the two smallest checks yourself

Install in an editable environment, then run the shipped fixtures:

```bash
python -m pip install -e .
python -m aidoc.cli doctor --config examples/acceptance.example.json
python -m aidoc.cli doctor --config examples/broken.example.json
```

Expected, on a machine with Python on PATH:

- `acceptance` reports `PASS` for the two required checks and `WARN` for the
  optional key, then `FIRST BLOCKER  none among required checks`.
- `broken` reports `FAIL` for the intentionally missing artifact, names it in
  `FIRST BLOCKER`, and adds the note that it is the first unsupported required
  transition — **not** a proven root cause.

To validate a contract without running it:

```bash
python -m aidoc.cli validate --config examples/acceptance.example.json
```

To write a starter contract of your own:

```bash
python -m aidoc.cli init --output my-check.json
```

## Useful documentation and tests contributions

Newcomer-friendly, high-signal work:

- Add a synthetic example contract that mirrors a real delivery question and
  documents what it proves and what it does not.
- Add a regression test in `tests/` from a delivery failure you have seen,
  using local fixtures only.
- Extend a check type and keep it read-only, deterministic, and secret-safe.
- Improve a doc page so a first-time reader can run the checks above.

## Boundaries: secrets and customer data

- Never include secret values, private customer endpoints, private IPs, or
  production logs in a fixture, test, or document.
- Keep checks read-only and never add arbitrary shell execution to a contract.
- Never turn a failed probe into a definitive root-cause claim; report the
  observed transition instead.
- Network probes must be explicit. Keep the core vendor-neutral and
  dependency-light.

The exact non-negotiable list is in [AGENTS.md](../AGENTS.md).
