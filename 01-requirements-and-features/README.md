# Stage 1 — Requirements & Features

Defines **what** Eventa must do and **why it matters** — owned by the Product Owner and written
as a product backlog.

## Documents
- **[functional-requirements.md](functional-requirements.md)** — the **product backlog**: 13
  epics broken into **161 user stories** (`As a <role>, I want <goal>, so that <benefit>`), each
  with business-observable acceptance criteria and a **MoSCoW** priority (88 Must / 58 Should /
  15 Could). The **Must** stories define the MVP; each epic opens with its business goal and
  success measures.
- **[non-functional-requirements.md](non-functional-requirements.md)** — **46 quality
  requirements** across 8 themes (performance, reliability, security, PDPA privacy,
  usability/accessibility, localization, scalability, supportability), written story-style with
  measurable business targets and MoSCoW priorities.

## How this flows into later stages
The backlog (the *what / why*) is elaborated into the technical design in
[Stage 4 — Architecture](../04-architecture/) (data model & ERD), verified against test cases in
[Stage 6 — Testing](../06-testing/), and released per the priorities set here.

Status: ✅ PO backlog + quality requirements drafted.
