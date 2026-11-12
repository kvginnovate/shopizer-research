# Shopizer — Engineering & Support Briefing

## 1. Platform Overview

**What it is.** Shopizer is an open-source, Java-based **headless e-commerce platform**: a Spring Boot REST backend plus two reference frontends (a React customer storefront and an Angular admin console). Version surveyed: backend 3.2.7 (Java 11 / Spring Boot 2.5.12), shop-react 3.0.0, shopizer-admin (Angular 11).

**What it does.** Out of the box it covers catalogue (products, categories, manufacturers, variants, pricing, inventory), shopping cart, checkout with payment/shipping/tax calculation, orders, customers, stores/merchants, content pages, and promotions — exposed as versioned REST APIs (`/api/v1`, `/api/v2`) documented via Swagger at `/swagger-ui.html`.

**Who it's for.** Small-to-mid retailers and SI teams that want a self-hosted, customisable alternative to SaaS carts (Shopify-style), especially multi-merchant/marketplace setups (stores, merchant onboarding exist as first-class concepts). Developers comfortable with Spring + MySQL operations.

**Problem it solves.** Gives you a full transactional commerce core (cart → totals → payment → order) with pluggable payment/shipping/tax modules, so a team can launch a storefront and admin without building pricing, checkout, or order management from scratch.

## 2. Technical Architecture

Three repos, one deployable backend:

```
shopizer-shop-reactjs (React 16 + Redux) ──┐
                                            ├─ REST /api/v1|v2 ── sm-shop (Spring Boot app)
shopizer-admin (Angular 11 + Nebular) ─────┘                         │
                                                                     ├─ sm-shop-model (REST DTOs)
                                                                     ├─ sm-core (services, repos, security, search, CMS)
                                                                     ├─ sm-core-model (JPA entities)
                                                                     └─ sm-core-modules (payment/shipping/tax SPIs)
                                                                              │
                        MySQL (SALESMANAGER, hbm2ddl=update) ◄── JPA/Hibernate ┤
                        Elasticsearch/OpenSearch (product search) ◄───────────┤
                        Infinispan/ehcache + CMS (file/S3/GCS) ◄───────────────┘
```

- **Backend monolith, Maven multi-module** (`shopizer/pom.xml`): `sm-core-model` (JPA entities per domain: catalog, customer, order, payments, shipping, tax, merchant, shoppingcart…), `sm-core` (business services + Spring Data repositories), `sm-core-modules` (integration SPIs: PayPal, Stripe, Braintree, BeanStream, MoneyOrder; UPS/USPS/store-pickup; Drools rules), `sm-shop-model` (API DTOs), `sm-shop` (controllers `store/api/v1/*Api` × ~60, facades `store/facade/*`, security, Swagger). Run: `mvnw clean install`, then `sm-shop: spring-boot:run` → :8080; Docker `shopizerecomm/shopizer:latest`.
- **Security:** URL-versioned APIs; `/private/**` = admin JWT (`AUTH` role), `/auth/**` = customer JWT (`AUTH_CUSTOMER`); public catalogue/cart endpoints open; login/refresh at `/v1/private/login`, `/v1/auth/refresh`, `customer/login`. Both frontends attach `Bearer` tokens via interceptors, token in localStorage.
- **Storefront** (`shopizer-shop-reactjs`): CRA + Redux + redux-thunk, axios client (`util/webService.js`, base from `public/env-config.js`: `APP_BASE_URL` + `/api/v1/`), routes for category/product/search/cart/checkout/order-confirm/my-account. Stripe Elements for cards. Cart id in a `{MERCHANT}_shopizer_cart` cookie.
- **Admin** (`shopizer-admin`): Angular + Nebular, `pages/` modules (orders, catalogue, customers, store/shipping/payment/tax/content), shared `crud.service.ts` against `environment.apiUrl`, Nebular auth, `ng serve` → :4200, default login `admin@shopizer.com / password`.

## 3. How It Works End-to-End

**Flow A — Customer places an order (storefront).**
1. `cartActions.js` creates/updates a cart: `POST /api/v1/cart` (body: items, store `DEFAULT`) → backend `ShoppingCartApi` → `ShoppingCartFacadeImpl` persists `ShoppingCart` + items; cart code returned and cookied.
2. `Checkout.js` loads cart (`GET /cart/{code}`), customer profile (`auth/customer/profile`), shipping options and order totals (`shipping`, `total/` endpoints; server-side `ShippingQuoteService` + order-total modules + Drools pricing rules).
3. Customer pays (Stripe token via Stripe Elements) → `POST cart/{id}/checkout` with address + payment token → `OrderFacade`/payment modules authorise/capture (Stripe/PayPal/…), persist `Order`, `OrderTotal`, payment transaction rows, decrement inventory, clear cart (`deleteAllFromCart`), route to `/order-confirm`. Failures surface via `react-toast-notifications`.

**Flow B — Admin manages the catalogue (admin console).**
1. Admin logs in: `POST /v1/private/login` → JWT stored by `TokenService`; `auth.interceptor.ts` attaches it to every `/v1/private/*` call.
2. Create product: catalogue UI → `POST /private/product` (or `/v2/private/product/*`) → `ProductApi` → product facade/service → JPA `Product`, `ProductDescription` (per-language), `ProductAvailability`/price rows in MySQL; images via `ProductImageApi` → CMS (local file/Infinispan or S3/GCS).
3. Storefront reads it back via public `GET /products`, `/product/{id}`, `/products/{id}` — indexed for `search/` (Elasticsearch) and cached (Infinispan/ehcache).

## 4. Strengths & Limitations

**Well-designed**
- True headless split: storefront and admin are thin API consumers; backend is fully Swagger-exposed — easy to replace either UI.
- Domain modelling is thorough (per-language descriptions, multi-merchant stores, price/availability separation, order-total pipeline) — real commerce edge cases are already modelled.
- Pluggable payment/shipping/tax modules behind SPIs, plus Drools for pricing/shipping rules — new providers don't require core rewrites.
- Docker images for all three pieces; default creds + demo data make a local stack fast to stand up.

**Where support/engineering time will go**
- **Staleness:** Angular 11 / React 16 / Spring Boot 2.5 / Java 11 are all past support; expect dependency CVEs, `npm install --legacy-peer-deps` friction, and Node 12/16-only build quirks. Upgrade path is the biggest latent cost.
- **Schema management:** `hbm2ddl.auto=update`, no Liquibase/Flyway — upgrades and multi-env drift are risky; first incident will be a schema surprise. Treat MySQL backups/migrations as day-one work.
- **Search/cache ops:** Elasticsearch + Infinispan are optional-but-assumed; misconfig here degrades search and performance silently. Health-check these in every deployment.
- **Troubleshooting hotspots:** checkout/payment webhook mismatches (Stripe/PayPal credentials per merchant), JWT expiry/role confusion (`AUTH` vs `AUTH_CUSTOMER`), multi-store data leaking across `DEFAULT` vs merchant codes, and CMS/file-store permission issues for product images.
- **Docs:** README-level run instructions only; API truth lives in Swagger + code. New engineers should read facades, not docs.

## 5. Onboarding a New Support Engineer

**Week-1 path (in order):**
1. **Run it locally first.** Backend (`mvnw clean install` → `sm-shop: spring-boot:run`, MySQL `SALESMANAGER`), then admin (`ng serve`, login `admin@shopizer.com/password`), then shop (`npm run dev`). Place a test order end-to-end with Stripe test keys. Goal: can reproduce any ticket locally.
2. **Learn the API map, not the UI.** Browse `/swagger-ui.html`; memorise the three security zones (public / `/auth/**` customer / `/private/**` admin) and which frontend calls which. 80% of "it doesn't work" tickets are wrong-zone/wrong-token issues.
3. **Trace the money path.** `ShoppingCartApi → ShoppingCartFacadeImpl → checkout → OrderFacade + payment modules → Order* tables`. Know where totals, tax, shipping quotes, and payment transactions are persisted — this is where sev-1s live.
4. **Learn the data model.** Core entities: `Product/Description/Availability`, `ShoppingCart`, `Order/OrderTotal`, `Customer`, `MerchantStore`; per-language description tables explain most "missing data" tickets.
5. **Own the ops checklist.** MySQL backups, `hbm2ddl` behaviour per env, Elasticsearch index rebuild, CMS storage backend, payment-provider credentials per merchant, log locations (`sm-shop` logs + browser network tab against `APP_BASE_URL`).

**Prioritise first:** local reproduction ability, JWT zone mental model, checkout money-path, then catalogue data model. Everything else (Drools rules, CMS backends, marketplace onboarding) is week 2+.
