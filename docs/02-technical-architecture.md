# 2. Technical Architecture

## System map

```
shopizer-shop-reactjs (React 16 + Redux, :3000) ──┐
                                                  ├─ REST /api/v1 | /api/v2 ── sm-shop (Spring Boot, :8080)
shopizer-admin (Angular 11 + Nebular, :4200) ─────┘                                        │
                                                                                          ├─ sm-shop-model (REST DTOs)
                                                                                          ├─ sm-core (services, repositories, security, search, CMS)
                                                                                          ├─ sm-core-model (JPA entities)
                                                                                          └─ sm-core-modules (payment / shipping / tax SPIs)
                                                                                                                   │
                                         MySQL (SALESMANAGER, hbm2ddl=update) ◄── JPA / Hibernate ──────────────────┤
                                         Elasticsearch / OpenSearch (product search) ◄── SearchService ────────────┤
                                         Infinispan / ehcache + CMS (file / S3 / GCS) ◄── cache + content ──────────┘
```

## Backend modules (`shopizer/`, Maven multi-module)

| Module | Role | Key packages |
|--------|------|--------------|
| `sm-core-model` | JPA domain entities, enums, exceptions | `core.model.*` (catalog, customer, order, payments, shipping, tax, merchant, user, content, shoppingcart, …) |
| `sm-core-modules` | Pluggable integration SPIs | `core.modules.integration.*` — payment impls (PayPal, Stripe, Braintree, BeanStream), shipping (UPS, USPS, store pickup, weight/distance rules), Drools tables |
| `sm-core` | Business services, Spring Data repos, config, CMS, search, email | `business.services.*`, `business.repositories.*`, `business.modules.*` |
| `sm-shop-model` | REST DTOs + validation | `shop.model.*` |
| `sm-shop` | The runnable Spring Boot app | `shop.store.api.*` (~60 `*Api` controllers under `v1`, product variants under `v2`), `shop.store.facade.*`, `shop.store.security.*`, admin controllers, Swagger config |

Build & run: `./mvnw clean install`, then `cd sm-shop && ./mvnw spring-boot:run`.
Docker: `sm-shop/Dockerfile` → `shopizerecomm/shopizer:latest`
(`docker run -p 8080:8080 shopizerecomm/shopizer:latest`).

## API & security design

- **Versioning:** URL path (`/api/v0`, `/api/v1`, `/api/v2`). Legacy `/shop/**`
  and `/services/**` routes still exist but are deprecated.
- **Three security zones** (`MultipleEntryPointsSecurityConfig`):
  - public catalogue/cart/content endpoints — no auth;
  - `/api/v*/auth/**` — customer JWT (`AUTH_CUSTOMER` role);
  - `/api/v*/private/**` — admin JWT (`AUTH` role).
- **Mechanism:** Spring Security + JWT (jjwt): `AuthenticationTokenFilter`,
  `JWTTokenUtil`, providers under `store/security/{admin,customer,user}`.
  Login: `/v1/private/login` (admin), `customer/login` (shopper);
  refresh: `/v1/auth/refresh`.
- **Controller layout** mirrors domains:
  `api/v1/{catalog,category,product,shoppingCart,order,customer,payment,
  shipping,tax,content,search,store,user,configurations,…}`.

## Data layer

- **MySQL 8**, schema `SALESMANAGER`, via JPA/Hibernate
  (`MySQL5InnoDBDialect`, `hbm2ddl.auto=update` — schema auto-evolves, no
  Liquibase/Flyway; creation SQL lives under
  `sm-core/src/test/resources/sql/`).
- **Search:** `SearchService` backed by the Shopizer search module,
  configured against Elasticsearch/OpenSearch.
- **Caching:** Infinispan 9.4 (+ ehcache), `@Cacheable` on hot paths;
  CMS content served from file / Infinispan / AWS S3 / GCS.
- **Rules engine:** Drools 7.32 decision tables for pricing/shipping logic.

## Frontends

**Storefront** (`shopizer-shop-reactjs`): Create-React-App, React 16,
React-Router 5, Redux 4 + thunk (+ multilanguage, localstorage-simple),
axios 0.21, Bootstrap 4.5, Stripe Elements. API client
`src/util/webService.js` (base URL from `public/env-config.js`:
`APP_BASE_URL` + `/api/v1/`, Bearer interceptor). Cart id persisted in a
`{MERCHANT}_shopizer_cart` cookie. Node 16; nginx-based Dockerfile.

**Admin** (`shopizer-admin`): Angular 11 + Nebular (auth/theme/security),
HttpClient, ngx-translate, ng2-smart-table/echarts. Routing:
`/auth` (lazy) + guarded `/pages/*` (orders, catalogue, customers,
store/shipping/payment/tax/content …). `crud.service.ts` on
`environment.apiUrl`; `auth.interceptor.ts` attaches the Bearer token
(localStorage) and logs out on 401. Node 12; nginx Docker images.

## Request path (typical)

Browser → React/Angular → axios/HttpClient (+Bearer) → `sm-shop` controller
→ facade (DTO mapping, orchestration) → `sm-core` service → JPA repository
→ MySQL; search/cache/CMS/payment-provider calls fan out from services and
integration modules → JSON back up the same chain.
