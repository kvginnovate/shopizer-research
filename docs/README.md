# Shopizer Documentation

Start here. This set documents the Shopizer open-source e-commerce platform
as surveyed from source (backend 3.2.7, shop-react 3.0.0, Angular admin).

| # | Document | Answers |
|---|----------|---------|
| 1 | [01-platform-overview.md](01-platform-overview.md) | What is Shopizer? Who is it for? What problem does it solve? |
| 2 | [02-technical-architecture.md](02-technical-architecture.md) | How is the system structured? Modules, APIs, data layer, integrations |
| 3 | [03-end-to-end-flows.md](03-end-to-end-flows.md) | Customer checkout + admin catalogue, step by step through the code |
| 4 | [04-strengths-limitations.md](04-strengths-limitations.md) | What's well-designed, where support time goes |
| 5 | [05-team-onboarding.md](05-team-onboarding.md) | How to get a new support engineer productive |
| 6 | [06-docker-setup.md](06-docker-setup.md) | Running the whole stack in Docker, reproducibly |
| 7 | [07-bug-fixes.md](07-bug-fixes.md) | Bug fixes and end-to-end verification |
| 8 | [08-demo-readiness.md](08-demo-readiness.md) | Demo account, remaining items, quick demo script |

**Upstream repositories surveyed** (linked, not vendored):

- [`shopizer/shopizer`](https://github.com/shopizer-ecommerce/shopizer) — Java Spring Boot backend (REST API + business logic)
- [`shopizer/shopizer-shop-reactjs`](https://github.com/shopizer-ecommerce/shopizer-shop-reactjs) — React customer storefront
- [`shopizer/shopizer-admin`](https://github.com/shopizer-ecommerce/shopizer-admin) — Angular admin console

**Fastest path to running it:** backend `mvnw clean install` then
`sm-shop: mvnw spring-boot:run` (http://localhost:8080, Swagger at
`/swagger-ui.html`), admin `ng serve` (:4200, `admin@shopizer.com / password`),
shop `npm run dev` (:3000). Docker images exist for all three
(`shopizerecomm/shopizer`, `shopizerecomm/shopizer-admin`,
`shopizerecomm/shopizer-shop-reactjs`).
