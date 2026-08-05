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
- **`.tldr` companions** — every `.md` holding Mermaid has a sibling tldraw file of the same stem
  (`erd.md` → `erd.tldr`), one **page per diagram**, for editing the diagrams on a canvas offline.
  Flowcharts and ERDs are real tldraw shapes (boxes, labels, arrows bound to their boxes) laid out at
  mermaid's computed coordinates; gantt/sequence/state have no shape equivalent and embed the
  rendering as a base64 SVG instead. Files target **tldraw v5** (current) — older tldraw cannot open
  them, newer will migrate them up.
  **These are derived artifacts and will silently go stale**: editing a ```mermaid fence does *not*
  update the `.tldr`, and editing a `.tldr` does *not* update the Markdown. The Markdown is the
  source of truth; regenerate rather than hand-reconciling. There is no committed generator (this
  repo has no build tooling) — the conversion is a throwaway script, so budget for rebuilding it if
  the diagrams change substantially.
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
  **The epics are ordered by build dependency, not by feature area** — `E1` Accounts is first because
  it is the only epic that needs nothing, and `E6` Discover & Register comes sixth because it needs a
  published event (`E3`+`E4`) and a ticket model (`E5`) to exist first. Read them top to bottom and
  you have a viable build sequence.
- **`01-.../user-story-map.md` is a derived view of that backlog** — the same stories re-cut by a
  **16-activity journey backbone** (4 acts, both personas, organizer→attendee handoff) × **3 release
  bands** tied to the plan's M3/M4/M5 gates. Its invariant: **every `US-*` appears exactly once**
  (the doc carries the `diff` command that checks this — run it after touching the backlog). It
  deliberately stores no story titles, dates or MoSCoW totals, so it is *not* another place counts
  can drift; only add/remove/re-prioritise forces an edit. Two release-vs-priority exceptions are
  recorded in both the map and `02-.../project-plan.md` §4.1 (`US-REG-04` Should→R1;
  six E12 Must stories→R2) — change one, change the other.
- **Architecture is layered:** `software-architecture.md` is the baseline SAD (NestJS modular monolith +
  RabbitMQ consumers + **transactional outbox**, synchronous checkout; C4 views + **ADRs**). `entities.md`
  is the schema **source of truth** (53 tables); `erd.md` is **derived from it** — keep them consistent
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
  `DISC, PAGE, ACC, DASH, EVT, TKT, PROG, REG, MTG, FIN, RPT, MSG, SET` (one per epic, E1–E13).
  **These are the stable ids and must never be renumbered** — they are cited from the test cases, the
  plan's WBS, the story map and the service repos' commit history. Note the AREA code is *not* derived
  from the epic number: `E1` Accounts holds `US-ACC-*`, `E6` Discover holds `US-DISC-*`. Always resolve
  an epic by its AREA code, never by assuming `E<n>` ↔ the nth AREA.
- **Epic numbers `E1`–`E13` were renumbered once** (2026-07-30) to put the backlog in build-dependency
  order; the AREA codes and every `US-*`/`TC-*` id were untouched. Do not renumber them again — if the
  order needs to change, record the new sequence in `user-story-map.md` instead.
- `NFR-<THEME>-<n>` quality requirements · `ADR-<n>` architecture decisions (in `software-architecture.md`).
- The big consolidated docs use explicit HTML anchors for in-document links: **`<a id="epic-eNN">`** in the
  backlog and **`<a id="tc-eNN">`** in the test cases (with a table-of-contents up top). Link to a section
  via its anchor, and add a matching `<a id="…">` when you add a section.

## Conventions that matter when editing

- **Counts are repeated** in doc headers and the root `README.md` (161 stories · 333 test cases · 53 tables
  · 13 epics/ADRs · MoSCoW splits). When a count changes, update every place — `README.md` is the index to
  keep current.
- **Crow's-foot notation in ERDs:** nullable FKs use `|o--o{`, non-null FKs use `||--o{` — stay consistent
  across the master diagram, the domain views, and the relationship matrix.
- **Authoring voice is in-role per stage** — Product Owner (backlog), Project Manager (plan), Solution
  Architect (SAD/DevOps), QA (tests). Match the voice of the doc you're editing.
- Each stage folder has its own `README.md` describing that stage's documents and status — keep it and the
  root index in sync when adding or renaming a document.
