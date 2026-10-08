# Stage 1 — Requirements & Features

Defines **what** Eventa must do and **why it matters** — owned by the Product Owner and written
as a product backlog.

## Documents
- **[functional-requirements.md](functional-requirements.md)** — the **product backlog**: 13
  epics broken into **162 user stories** (`As a <role>, I want <goal>, so that <benefit>`), each
  with business-observable acceptance criteria and a **MoSCoW** priority (90 Must / 57 Should /
  15 Could). The **Must** stories define the MVP; each epic opens with its business goal and
  success measures.
- **[non-functional-requirements.md](non-functional-requirements.md)** — **46 quality
  requirements** across 8 themes (performance, reliability, security, PDPA privacy,
  usability/accessibility, localization, scalability, supportability), written story-style with
  measurable business targets and MoSCoW priorities.
- **[user-story-map.md](user-story-map.md)** — a **derived view** of the same backlog, re-cut by
  *when the user meets each story* instead of by feature area: a **16-activity journey backbone**
  across four acts (both personas on one axis, with the organizer→attendee handoff visible), sliced
  into **three release bands** keyed to the plan's M3 / M4 / M5 gates. Confirms the MVP is a complete
  walking skeleton for the money path, and flags the three activities that ship empty at launch
  (promotion, partner meetings, reporting & feedback). Holds no titles, dates or totals — only
  `US-*` ids — so it stays cheap to maintain.

## How this flows into later stages
The backlog (the *what / why*) is elaborated into the technical design in
[Stage 4 — Architecture](../04-architecture/) (data model & ERD), verified against test cases in
[Stage 6 — Testing](../06-testing/), and released per the priorities set here. The
[story map](user-story-map.md) is what [Stage 2 — Project Plan](../02-project-plan/) slices its
phases from; where an epic straddles two phases, the map is the authority on which stories fall
either side.

Status: ✅ PO backlog + quality requirements + story map drafted.
