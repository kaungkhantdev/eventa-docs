# Stage 4 — Architecture

The technical blueprint: system architecture, data model, and API design that satisfy the
requirements.

## Documents
- **[entities.md](entities.md)** — the relational **data dictionary**: 44 PostgreSQL tables
  across 11 bounded contexts, with columns/types/keys/constraints/indexes, enumerated types,
  junction tables, and a full relationship summary. Multi-tenant (`organization_id`), money as
  integer satang, audit & soft-delete columns, order→ticket commerce model.
- **[erd.md](erd.md)** — the **Entity Relationship Diagram**: a master crow's-foot Mermaid
  diagram plus 11 per-domain views and a 76-row relationship matrix, derived directly from
  `entities.md`.

The data model is grounded in the SRS domain model — see
[§1 Domain Data Model](../01-requirements-and-features/functional-requirements.md#sec-01-domain-data-model).

## What still belongs here (optional)
- **System / deployment architecture** diagram (front-end, API, datastore, integrations).
- **API design** — REST/GraphQL endpoint catalog mapped to `FR-*` requirements.
- **Tech-stack decisions** / ADRs (architecture decision records).
- **Security architecture** (authN/authZ, PCI via Stripe, PDPA).

Status: ✅ Data model & ERD drafted. System/API architecture: ⬜ not yet.
