# Project memory - <project name>

## Verified facts
<!-- Checked against reality (a query, a test, a doc). Stop guessing about these. Include how it was verified. -->
- prc is in dollars, not cents. Verified via SELECT MIN(prc), MAX(prc) FROM trades.
- Test database uses sandbox keys; production uses real keys via env.

## General rules
<!-- Distilled lessons that apply beyond the specific case. Consult before re-deriving. -->
- When querying time-bucketed metrics, always include timezone (default UTC mismatches).
- For migrations, never ALTER tables >1M rows without batching.

## Open failures
<!-- Documented failures awaiting investigation. Include hypothesis and reproduction steps. -->
- 2026-06-09: tests/e2e/checkout flakes ~1 in 50 runs. Hypothesis: webhook race.
  Reproduction steps in debug/checkout-flake.md.

## Lessons learned
<!-- Post-mortem distillations worth keeping. Promote the general ones to skills. -->
- CI runners fail TLS 1.2 in PowerShell. Always shell out to bash.

## Last session
<!-- Resume pointer: timestamp, what happened, what's next. Overwrite each session. -->
2026-06-10 03:30 UTC - 7 failures classified, 3 fixes drafted, 4 escalated.
Next: verify the auth middleware fix against production load.
