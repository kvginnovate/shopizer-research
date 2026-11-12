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

**Repositories** (cloned alongside this docs set):

- `shopizer/` — Java Spring Boot backend (REST API + business logic)
- `shopizer-shop-reactjs/` — React customer storefront
- `shopizer-admin/` — Angular admin console

**Fastest path to running it:** backend `mvnw clean install` then
`sm-shop: mvnw spring-boot:run` (http://localhost:8080, Swagger at
`/swagger-ui.html`), admin `ng serve` (:4200, `admin@shopizer.com / password`),
shop `npm run dev` (:3000). Docker images exist for all three
(`shopizerecomm/shopizer`, `shopizerecomm/shopizer-admin`,
`shopizerecomm/shopizer-shop-reactjs`).
