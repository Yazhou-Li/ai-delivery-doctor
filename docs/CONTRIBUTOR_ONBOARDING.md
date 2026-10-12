# Contributor Onboarding

A short, evidence-first guide to making your first contribution to AI
Delivery Doctor. The goal is not a big diff — it is a contribution that is
**deterministic, reviewable, and safe**.

## 1. The idea in one paragraph

AI Delivery Doctor produces **evidence**, not opinions. The deterministic
core records *what was actually observed* (a file exists, a port answers,
an executable is on the PATH). Model or Agent reasoning is welcome upstream
of that boundary — it can help decide *what should be verified* — but the
core never fabricates an observation. Every check makes one bounded
observation, is read-only by default, and carries a narrow PASS/WARN/FAIL
meaning. Contributions that respect that line are the kind we merge.

## 2. A minimal, local example to start from

The repository ships synthetic examples under `examples/` that need no
network and no credentials:

```bash
python -m pip install -e .
python -m unittest discover -s tests -v

# Validate the contract (schema + check shape)
aidoc validate --config examples/contributor-hello-world.example.json

# Run it and read the evidence
aidoc doctor --config examples/contributor-hello-world.example.json
```

`contributor-hello-world.example.json` is intentionally small: one required
`file` check against the repo's own `pyproject.toml`, one `executable`
check for `python`, and one *optional* `env` check for `OPENAI_API_KEY`.
Run it locally and you will see two PASS lines and one WARN for the
missing key — the WARN is the expected "not required" outcome, not a
failure. That is the whole contribution pattern in miniature: **a small
ordered chain of checks that traces one real outcome.**

Copy the pattern, keep the chain short, and read the evidence before you
share it.

## 3. What makes a useful contribution

Pick the smallest one of these that matches the change you want to make:

- **A new deterministic check type.** It must make one bounded
  observation, declare its inputs, emit a narrow PASS/WARN/FAIL, stay
  read-only, and not leak secrets. Pair it with a unit test in
  `tests/` (see `tests/test_checks.py`).
- **Safer evidence redaction.** If a detail can carry a host, path, or
  value that should not leave the machine, fix the sanitizer in
  `src/aidoc/checks.py` and add a regression test that asserts the
  redacted output (see how `sanitize_result` maps each check type to a
  fixed wording).
- **Documentation or a reproducible example.** Synthetic, local, and
  dependency-light: extend an `examples/*.example.json` fixture or a
  page under `docs/`. No real credentials, no private IPs, no production
  logs.
- **A regression test from a real delivery failure.** Reconstruct the
  failure as a synthetic fixture and assert the *observed* result — not
  the guessed root cause.

## 4. Boundaries that are not negotiable

These come from `AGENTS.md` and are enforced by review:

- **Never log secret values.** Probe env keys by name, not by value.
- **Never execute arbitrary shell from a contract.** Checks observe;
  they do not run user-supplied commands.
- **Network probes must be explicit.** Use the `tcp`/`http`/
  `openai-compatible` types against synthetic hosts in tests.
- **Do not mutate the target system** from diagnostic code.
- **Do not turn a failed probe into a root-cause claim.** A missing
  artifact is *reported as the first blocker*, not explained as "the
  cause of everything downstream."
- **Keep the core dependency-light and vendor-neutral.** Adapters for a
  specific framework, model provider, or cloud live *around* the core,
  never inside it.

## 5. Before you open a pull request

```bash
python -m pip install -e .
python -m unittest discover -s tests -v
aidoc validate --config examples/acceptance.example.json
aidoc doctor --config examples/acceptance.example.json
```

Run the project's own checks plus the ones for your example. Then answer,
in the PR description: what delivery problem it solves, what observable
fact it establishes, what changed, how you verified it, what the result
**does not** prove, and whether it kept privacy and read-only defaults.
