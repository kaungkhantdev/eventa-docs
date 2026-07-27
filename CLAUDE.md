# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

The **SDLC documentation** for **Eventa**, an event registration & management platform. It is **plain
Markdown only** — no application code, no build system — organized into **eight lifecycle-stage folders**
(`01-requirements-and-features` … `08-maintenance`), one per stage. `README.md` is the authoritative
index (a stage/status table + a list of the current documents); read it first. The actual product code
lives in **sibling** repositories: `../eventa-web` (the React 19 prototype, has its own CLAUDE.md) and
`../eventa-ui-kit` (the static HTML/Tailwind design kit).

## Commands / tooling

There is **no build, lint, or test tooling** — changes are plain Markdown edits committed with `git`.
The docs rely on two things a viewer must support and that you must keep valid **by hand** (there is no
committed checker):
- **Mermaid diagrams** (```mermaid fences) — C4 container/context, crow's-foot ERDs, state machines,
  sequence, gantt, flowcharts. They render on GitHub / VS Code. After editing one, sanity-check that
  entity/attribute braces balance and the diagram header is valid.
- **Cross-links** — relative paths between docs (e.g. `../04-architecture/entities.md`) plus
  **in-document anchors** in the large consolidated files. When you move or rename a doc, fix the
  inbound relative links **and** the `README.md` index.

## How the documents interlock (the part that spans files)

The value is the **traceability spine**, not any single file:

```
user story (US-*)  →  architecture (software-architecture.md + entities.md/erd.md)
                   →  test case (TC-*)  →  deploy & operate (DevOps, stages 07–08)
```

Change one link and the others should follow. Some specifics that require reading several files to grasp:

- **Requirements are a product backlog, not a flat list.** `01-.../functional-requirements.md` holds
  **13 epics (`E1`–`E13`) → 161 user stories** in Product-Owner voice (`As a … I want … so that …`) with
  MoSCoW priorities; `non-functional-requirements.md` holds the quality requirements (`NFR-*`).
- **Architecture is layered:** `software-architecture.md` is the baseline SAD (NestJS modular monolith +
  RabbitMQ consumers + **transactional outbox**, synchronous checkout; C4 views + **ADRs**). `entities.md`
  is the schema **source of truth** (47 tables); `erd.md` is **derived from it** — keep them consistent
  (e.g. the `outbox_events` / `seat_holds` / `webhook_events` tables exist specifically because SAD
  patterns require them). If you touch one, update the other and their shared counts.
- **Tests trace to stories:** `06-testing/test-cases.md` has **333 `TC-*` cases** covering **all 161
  stories**; `test-plan.md` is the strategy/process. Each test case names the `US-*` it verifies.
- **DevOps is deliberately split across stages 07 and 08**, not under `04-architecture`: `07-deployment`
  holds `devops-architecture.md` (overview) + `devops-ci-cd.md` + `devops-infrastructure.md`;
  `08-maintenance` holds `devops-observability-sre.md`. These four cross-reference each other **across the
  two folders** — preserve those relative paths.

## ID & anchor conventions (stable references — never renumber)

- `US-<AREA>-<n>` user stories · `TC-<AREA>-<n>` test cases — same **AREA** codes:
  `DISC, PAGE, ACC, DASH, EVT, TKT, PROG, REG, MTG, FIN, RPT, MSG, SET` (one per epic E1–E13).
- `NFR-<THEME>-<n>` quality requirements · `ADR-<n>` architecture decisions (in `software-architecture.md`).
- The big consolidated docs use explicit HTML anchors for in-document links: **`<a id="epic-eNN">`** in the
  backlog and **`<a id="tc-eNN">`** in the test cases (with a table-of-contents up top). Link to a section
  via its anchor, and add a matching `<a id="…">` when you add a section.

## Conventions that matter when editing

- **Counts are repeated** in doc headers and the root `README.md` (161 stories · 333 test cases · 47 tables
  · 13 epics/ADRs · MoSCoW splits). When a count changes, update every place — `README.md` is the index to
  keep current.
- **Crow's-foot notation in ERDs:** nullable FKs use `|o--o{`, non-null FKs use `||--o{` — stay consistent
  across the master diagram, the domain views, and the relationship matrix.
- **Authoring voice is in-role per stage** — Product Owner (backlog), Project Manager (plan), Solution
  Architect (SAD/DevOps), QA (tests). Match the voice of the doc you're editing.
- Each stage folder has its own `README.md` describing that stage's documents and status — keep it and the
  root index in sync when adding or renaming a document.
