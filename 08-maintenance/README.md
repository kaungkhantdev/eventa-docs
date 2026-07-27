# Stage 8 — Maintenance & Operations

Keeping the platform healthy and evolving after launch — the **operate/monitor** half of the DevOps
documentation set (the build/ship half is in [Stage 7 — Deployment](../07-deployment/)).

## Document
- **[devops-observability-sre.md](devops-observability-sre.md)** — Observability, SRE & DevSecOps:
  the three pillars (metrics/logs/traces via Prometheus / Grafana / Loki / OpenTelemetry / Sentry),
  **domain SLIs** (checkout, check-in, outbox lag, RabbitMQ queue depth), **SLOs & error budgets**,
  alerting & on-call, incident management with **blameless postmortems**, the shift-left **security
  gates + PCI/PDPA controls**, **backups / PITR / DR**, and a runbooks index.

The DevOps **overview, CI/CD, and infrastructure** docs are in
[Stage 7 — Deployment](../07-deployment/devops-architecture.md). Together they form the platform's
DevOps documentation, grounded in the
[software architecture](../04-architecture/software-architecture.md).

## What still belongs here (optional)
- Support process / SLAs, changelog / release notes, and the post-launch enhancement backlog
  (deferred `Could` stories from the [product backlog](../01-requirements-and-features/functional-requirements.md)).

Status: ✅ Observability / SRE / DevSecOps / DR documented.
