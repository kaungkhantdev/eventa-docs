# Eventa — SDLC Workspace

Documentation for **Eventa**, the event registration & management platform, organized by
the software development lifecycle. The folder structure follows the 8-stage process in
Hostinger's [How to Build Software](https://www.hostinger.com/tutorials/how-to-build-software)
guide — one folder per stage.

| # | Stage | Folder | Purpose | Status |
|---|-------|--------|---------|--------|
| 1 | Requirements & Features | [`01-requirements-and-features/`](01-requirements-and-features/) | *What* the software must do + *how* users interact | ✅ Drafted |
| 2 | Project Plan | [`02-project-plan/`](02-project-plan/) | Phases, milestones, scope, timeline | ⬜ Not started |
| 3 | UX / UI Design | [`03-ux-ui-design/`](03-ux-ui-design/) | Wireframes, mockups, design system, user flows | 🟡 Prototype exists in `../eventa-web` |
| 4 | Architecture | [`04-architecture/`](04-architecture/) | System & data architecture, ERD, API design | ✅ Data model drafted |
| 5 | Development | [`05-development/`](05-development/) | Coding standards, module breakdown, build notes | 🟡 Front-end prototype in `../eventa-web` |
| 6 | Testing | [`06-testing/`](06-testing/) | Test plan, test cases, QA, traceability to FR-IDs | ⬜ Not started |
| 7 | Deployment | [`07-deployment/`](07-deployment/) | Release plan, environments, CI/CD, infra | ⬜ Not started |
| 8 | Maintenance | [`08-maintenance/`](08-maintenance/) | Monitoring, support, iteration, changelog | ⬜ Not started |

## Current contents
- **Stage 1** — `functional-requirements.md` (Product Owner backlog: 161 user stories across 13 epics, MoSCoW), `non-functional-requirements.md` (46 quality requirements).
- **Stage 4** — `entities.md` (relational data dictionary, 44 tables), `erd.md` (crow's-foot ERD).

The implemented **React front-end prototype** lives in the sibling folder `../eventa-web`; the
static HTML kit in `../eventa-ui-kit`. This `sdlc/` folder holds the *documentation* only.

_Last updated: 2026-07-23._
