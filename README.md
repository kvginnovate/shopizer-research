# Shopizer — Interview Research Assignment

Engineering & support briefing on the
[Shopizer](https://github.com/shopizer-ecommerce/shopizer) open-source
e-commerce platform, prepared as a take-home assignment.

## How this answers the brief

| # | Assignment question | Where it's answered |
|---|---------------------|---------------------|
| 1 | Platform overview — what it does, who it's for, what problem it solves | [`docs/01-platform-overview.md`](docs/01-platform-overview.md) |
| 2 | Technical architecture — components, how they connect (diagrams included) | [`docs/02-technical-architecture.md`](docs/02-technical-architecture.md) |
| 3 | End-to-end — customer checkout + admin catalogue, user action to DB and back | [`docs/03-end-to-end-flows.md`](docs/03-end-to-end-flows.md) |
| 4 | Strengths & limitations — support/engineering view | [`docs/04-strengths-limitations.md`](docs/04-strengths-limitations.md) |
| 5 | Onboarding — getting a new support engineer productive | [`docs/05-team-onboarding.md`](docs/05-team-onboarding.md) |

Short on time? Read [`SHOPIZER-BRIEFING.md`](SHOPIZER-BRIEFING.md) — the
condensed five-part version. `docs/` is the full write-up.

## Methodology & sources

Surveyed directly from source code (not just marketing pages), using AI
tooling to accelerate the reading — per the brief, used as a research
assistant; all findings verified against the repos:

- [`shopizer/shopizer`](https://github.com/shopizer-ecommerce/shopizer) —
  Java Spring Boot backend, v3.2.7 (REST API + business logic)
- [`shopizer/shopizer-shop-reactjs`](https://github.com/shopizer-ecommerce/shopizer-shop-reactjs) —
  React 16 + Redux customer storefront, v3.0.0
- [`shopizer/shopizer-admin`](https://github.com/shopizer-ecommerce/shopizer-admin) —
  Angular 11 + Nebular admin console

Key claims are grounded in file paths and class names cited inline
(e.g. `ShoppingCartFacadeImpl`, `ProductApi`, `auth.interceptor.ts`) and
cross-checked between the backend controllers and the frontend API clients.
