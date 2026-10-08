# Eventa — SDLC Workspace

Documentation for **Eventa**, the event registration & management platform, organized by
the software development lifecycle. The folder structure follows the 8-stage process in
Hostinger's [How to Build Software](https://www.hostinger.com/tutorials/how-to-build-software)
guide — one folder per stage.

| # | Stage | Folder | Purpose | Status |
|---|-------|--------|---------|--------|
| 1 | Requirements & Features | [`01-requirements-and-features/`](01-requirements-and-features/) | *What* the software must do + *how* users interact | ✅ Backlog + story map |
| 2 | Project Plan | [`02-project-plan/`](02-project-plan/) | Phases, milestones, scope, timeline | ✅ Drafted |
| 3 | UX / UI Design | [`03-ux-ui-design/`](03-ux-ui-design/) | Wireframes, mockups, design system, user flows | ✅ Documented (prototype = `../eventa-web`) |
| 4 | Architecture | [`04-architecture/`](04-architecture/) | System & data architecture, ERD, API design | ✅ Architecture + data model |
| 5 | Development | [`05-development/`](05-development/) | Coding standards, module breakdown, build notes | ✅ Dev guide (prototype in `../eventa-web`) |
| 6 | Testing | [`06-testing/`](06-testing/) | Test plan, test cases, QA, traceability to FR-IDs | ✅ Plan + 340 cases designed |
| 7 | Deployment | [`07-deployment/`](07-deployment/) | Release plan, environments, CI/CD, infra | ✅ DevOps: CI/CD + IaC + overview |
| 8 | Maintenance | [`08-maintenance/`](08-maintenance/) | Monitoring, support, iteration, changelog | ✅ Observability / SRE / DevSecOps |

## Current contents
- **Stage 1** — `functional-requirements.md` (Product Owner backlog: 162 user stories across 13 epics, MoSCoW), `non-functional-requirements.md` (46 quality requirements), `user-story-map.md` (the backlog re-cut as a 16-activity journey backbone × 3 release bands; walking-skeleton check).
- **Stage 2** — `project-plan.md` (phased delivery plan, 15-sprint schedule, milestones, risks; MVP launch Dec 2026, v1.0 GA Mar 2027).
- **Stage 3** — `design-reference.md` (screen inventory, user flows & design-system index pointing at the `../eventa-web` prototype + `../eventa-ui-kit`).
- **Stage 4** — `software-architecture.md` (baseline SAD: NestJS modular monolith + RabbitMQ event-driven consumers + transactional outbox, synchronous checkout, C4 views, 13 ADRs), `entities.md` (relational data dictionary, 53 tables), `erd.md` (crow's-foot ERD).
- **Stage 5** — `development-guide.md` (monorepo layout, local setup, coding standards, build-a-feature playbook, Git workflow, DoD).
- **Stage 6** — `test-plan.md` (7-step cycle: strategy, environment, defect process, closure), `test-cases.md` (340 test cases across 13 epics, traced to all 162 user stories).
- **Stage 7** — `devops-architecture.md` (DevOps overview: CALMS/Three Ways/DORA), `devops-ci-cd.md` (CI/CD & release, GitOps + canary), `devops-infrastructure.md` (Terraform/Helm/Argo CD, Kubernetes).
- **Stage 8** — `devops-observability-sre.md` (observability, SLOs, incident mgmt, DevSecOps, DR).

## Editable diagrams (`.tldr`)

Every document containing Mermaid diagrams has a **tldraw companion of the same name** in the same
folder — `04-architecture/erd.md` → `04-architecture/erd.tldr` — with **one page per diagram**
(33 diagrams across 11 files). Open them offline at [tldraw.com](https://tldraw.com) (drag the file
in) or in any current tldraw app to edit the diagrams on a canvas.

Flowcharts and ERDs are real, editable shapes — boxes, labels, and arrows bound to their boxes, so
dragging a box keeps its connections. Gantt, sequence and state diagrams have no shape equivalent, so
those pages embed the rendering as an image instead.

> The `.tldr` files are **generated from the Markdown**, which stays the source of truth. They do not
> update automatically when a ```mermaid fence changes, and edits made on the canvas do not flow back.
> They require tldraw v5 or newer.

The implemented **React front-end prototype** lives in the sibling folder `../eventa-web`; the
static HTML kit in `../eventa-ui-kit`. This `sdlc/` folder holds the *documentation* only.

_Last updated: 2026-08-05._
