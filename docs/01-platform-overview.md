# 1. Platform Overview

## What Shopizer is

Shopizer is an **open-source, headless e-commerce platform**: a Java
(Spring Boot) REST backend with two reference frontends — a React customer
storefront and an Angular admin console. The backend is the product; either
UI can be replaced by any HTTP client.

- Backend: `shopizer/`, v3.2.7, Java 11, Spring Boot 2.5.12
- Storefront: `shopizer-shop-reactjs/`, v3.0.0, React 16 + Redux
- Admin: `shopizer-admin/`, Angular 11 + Nebular

## What it does (capabilities)

| Domain | Includes |
|--------|----------|
| Catalogue | Products, categories, manufacturers, variants, attributes, pricing, inventory, reviews |
| Cart & checkout | Guest + authenticated carts, promotions, shipping quotes, tax, order totals |
| Payments | Pluggable modules: Stripe, PayPal (Express + REST), Braintree, BeanStream, money order |
| Orders | Order creation, totals, status history, shipping, payment transactions |
| Customers | Registration, login (JWT), profiles, addresses, password reset |
| Merchants/stores | Multi-store support, merchant onboarding/signup, per-store config |
| Content | CMS pages, content blocks, newsletters, contact |
| Search | Product search via Elasticsearch/OpenSearch integration |

All of this is exposed as **versioned REST APIs** (`/api/v1`, `/api/v2`),
self-documented through Swagger UI at `/swagger-ui.html`.

## Who it is built for

- **Small-to-mid retailers** needing a self-hosted store without SaaS lock-in.
- **Marketplace operators**: stores/merchants are first-class concepts, so
  multiple sellers can be served from one deployment.
- **SI / dev teams** comfortable operating a Spring + MySQL stack who want a
  customisable commerce core instead of building cart/checkout/orders from
  scratch.

## What problem it solves

Building transactional commerce (pricing rules, tax/shipping calculation,
payment authorisation/capture, inventory decrement, order lifecycle) is
undifferentiated heavy lifting. Shopizer provides that core, already modelled
for real edge cases (per-language descriptions, price/availability split,
order-total pipeline), with payment/shipping/tax behind plugin SPIs — so a
team starts from a working shop and admin and spends effort on theme,
integrations, and business rules.

## Positioning in one paragraph

Think of Shopizer as a self-hosted, Java-based equivalent of the headless
commerce pattern (backend-for-frontends): one Spring Boot monolith owns all
commerce state and rules in MySQL, exposes it over REST, and ships two
working clients as starting points. It trades SaaS convenience for full code
and data ownership.

## License & community

Open source (see `LICENSE.md` in each repo). Public demo, Slack community,
and docs site are linked from the backend README. Swagger on a running
instance is the authoritative API reference.
