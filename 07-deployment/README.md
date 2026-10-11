# Stage 7 — Deployment

Releasing the product and the infrastructure that runs it — the **build/ship** half of the DevOps
documentation set (the operate/monitor half is in [Stage 8 — Maintenance](../08-maintenance/)).

## Documents
- **[devops-architecture.md](devops-architecture.md)** — DevOps **overview**: principles (CALMS +
  the Three Ways), the delivery lifecycle, the deployable services (web / api / worker / relay + the
  dedicated check-in pool), environments & promotion, the toolchain, and DORA metrics. **Start here.**
- **[devops-ci-cd.md](devops-ci-cd.md)** — CI/CD & release management: trunk-based branching + PR
  preview envs, CI stages & quality gates, **GitOps (Argo CD)** with rolling/**canary** deploys, gated
  zero-downtime **DB migrations**, feature flags, rollback, and versioning.
- **[devops-infrastructure.md](devops-infrastructure.md)** — Infrastructure as Code: **Terraform +
  Helm + Argo CD**, the **Kubernetes** topology (a Deployment per service, HPAs on all of them
  except the singleton `relay`, the check-in pool),
  managed PostgreSQL/Redis/RabbitMQ, secrets/config, scaling for on-sale & check-in spikes, and FinOps.

The runtime **observability, SRE, DevSecOps and DR** doc lives in
[Stage 8 — Maintenance](../08-maintenance/devops-observability-sre.md). Together these four documents
form the platform's DevOps documentation, grounded in the
[software architecture](../04-architecture/software-architecture.md).

Status: ✅ CI/CD + infrastructure + DevOps overview documented.
