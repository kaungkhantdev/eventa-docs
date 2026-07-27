# Stage 3 — UX / UI Design

The UX/UI design is delivered as a **working prototype** (design-in-code), not static mockups.

## Document
- **[design-reference.md](design-reference.md)** — a thin **map/index** of the design: where it
  lives, a full screen inventory (grouped by area, routed, tagged to backlog epics), the key user
  flows, and a pointer to the design system. It indexes the prototype rather than re-documenting it.

## Where the design lives
- **[`../../eventa-web`](../../eventa-web)** — the React implementation: the full clickable screen
  set across the attendee portal, public landing pages, and admin console, with light/dark theming.
  This is the high-fidelity, interactive design spec.
- **[`../../eventa-ui-kit`](../../eventa-ui-kit)** — the static HTML/Tailwind design kit (visual
  source of truth: colours, typography, components).

To view it live, run `eventa-web` (`pnpm dev`, port 5180) and browse the routes in the reference.

**Maintenance rule:** the prototype is authoritative — update the code; keep `design-reference.md` as
a lightweight index so it never drifts.

Status: ✅ Documented (design delivered as the prototype; reference doc indexes it).
