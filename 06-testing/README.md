# Stage 6 — Testing

Verifies the software meets its requirements and is free of critical defects.

## What belongs here
- **Test plan** — scope, levels (unit / integration / e2e), environments, entry/exit criteria.
- **Test cases** — ideally one or more per `FR-*` requirement, using the Given/When/Then
  acceptance criteria already written in the SRS.
- **Traceability matrix** — `FR-<AREA>-<NNN>` → test case → pass/fail, proving every requirement
  is verified.
- **QA reports / defect log.**

Seed: every requirement in
[functional-requirements.md](../01-requirements-and-features/functional-requirements.md) already
carries Given/When/Then acceptance criteria — these convert directly into test cases (e.g.
`TC-EVT-010` verifies `FR-EVT-010`).

Status: ⬜ Not started.
