# Stage 4 — Architecture

The technical blueprint: the software architecture plus the data model that satisfy the requirements.

## Documents
- **[software-architecture.md](software-architecture.md)** — the **baseline Software Architecture
  Document (SAD)**: goals & constraints, principles, C4 context & container views, the NestJS
  **module (bounded-context)** design, the **event-driven** topology (RabbitMQ + a NestJS consumer
  tier + the **transactional outbox**), runtime views (the **synchronous, transactional checkout**
  boundary; reliable eventing), deployment, cross-cutting concerns (payments/PCI, multi-tenancy,
  concurrency & overselling, PDPA, observability, security), the technology stack, **13 ADRs**, and
  an NFR → mechanism mapping.
- **[entities.md](entities.md)** — the **data architecture**: relational data dictionary, 47
  PostgreSQL tables across 11 bounded contexts, with keys, constraints, indexes, enums, junction
  tables. Multi-tenant, money-as-satang, order→ticket commerce model.
- **[erd.md](erd.md)** — the **Entity Relationship Diagram**: master crow's-foot Mermaid diagram +
  11 domain views + a 76-row relationship matrix, derived from `entities.md`.

## Stack (baseline)
React (SPA + SSR) · **NestJS** API (modular monolith) · **RabbitMQ** eventing + NestJS consumers ·
**transactional outbox** · PostgreSQL (+ RLS, FTS) · Redis (cache/sessions/idempotency) ·
Stripe + PromptPay · object storage/CDN. Checkout/inventory is **synchronous & transactional**;
side effects are **asynchronous** via RabbitMQ.

## How it relates
`software-architecture.md` (the *how the system is built*) sits on top of `entities.md`/`erd.md`
(the *how the data is shaped*), both realising the [product backlog](../01-requirements-and-features/functional-requirements.md)
and the [quality requirements](../01-requirements-and-features/non-functional-requirements.md).

## What still belongs here (optional)
- A detailed **API endpoint catalogue** (resources mapped to `US-*` / `FR-*`).
- Per-decision **ADR files** (the SAD summarises them; each can become its own record).

Status: ✅ Baseline software architecture + data model & ERD.
