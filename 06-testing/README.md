# Stage 6 — Testing

Verifies the software meets its requirements — owned by QA, following the 7-step testing cycle.

## Documents
- **[test-plan.md](test-plan.md)** — the master **test plan**: requirement analysis (testability +
  ambiguities), strategy (levels & types: functional, usability, performance, security, compatibility,
  accessibility, localization, regression, UAT), schedule aligned to the project-plan milestones,
  entry/exit criteria, **test environment setup**, the **defect-management process** (severity,
  priority, lifecycle diagram, report template), and the **test-closure / summary-report** template.
- **[test-cases.md](test-cases.md)** — **333 test cases** across 13 epics, each derived from a
  story's Given/When/Then acceptance criteria and traced to its `US-*` id (priority breakdown:
  179 High / 133 Medium / 21 Low). Includes a coverage table proving **all 161 user stories** have
  test cases.

## Traceability
Every test case → `US-*` story → (via the [backlog](../01-requirements-and-features/functional-requirements.md))
its `FR-*` requirement. This closes the loop: requirement → design → test.

## Current status
Test **design** is complete (plan + cases). **Execution, defect logging, and closure** happen during
development once builds are testable (the backend isn't built yet) — the front-end prototype can be
exercised now for early functional/usability/accessibility feedback.

Status: ✅ Test plan + test cases designed (execution pending a testable build).
