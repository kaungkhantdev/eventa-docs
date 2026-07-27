# Stage 5 — Development

Implementation — turning the architecture into working code.

## Document
- **[development-guide.md](development-guide.md)** — the engineering handbook: the recommended
  **polyrepo** layout (separate `eventa-web` / `eventa-api` / `eventa-worker` repos; no shared
  package — OpenAPI codegen for web↔api, Pact contract tests for api↔worker), **local setup**
  (docker-compose), **coding standards**, the **"build a feature" playbook** (synchronous-checkout
  vs outbox boundary, tenant scoping, idempotency), **migrations** (expand/contract), API & messaging
  conventions, **Git workflow**, Definition of Done, and shift-left security & observability.

## Already exists
- **[`../../eventa-web`](../../eventa-web)** — the React front-end prototype (runs today); its
  `CONVENTIONS.md` holds the frontend conventions.
- The backend (NestJS) is specified in [Stage 4](../04-architecture/software-architecture.md) and is
  built per this guide; delivery is scheduled in the [project plan](../02-project-plan/project-plan.md).

Status: ✅ Development guide written (front-end prototype in place; backend to be built).
