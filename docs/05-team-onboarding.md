# 5. Onboarding a New Support Engineer

## Goal

In week 1 the engineer can: run the full stack locally, reproduce any
ticket, identify which API zone and service owns a bug, and trace the
checkout money-path. Week 2 covers rules engines, CMS backends, and
marketplace onboarding.

## Week-1 path (in order — each step unlocks the next)

### 1. Run it locally (day 1)
- Backend: `cd shopizer && mvnw clean install`, then
  `cd sm-shop && mvnw spring-boot:run` → http://localhost:8080
  (create MySQL schema `SALESMANAGER` first).
- Admin: `cd shopizer-admin && npm install --legacy-peer-deps && ng serve`
  → http://localhost:4200, login `admin@shopizer.com / password`.
- Shop: `cd shopizer-shop-reactjs && npm i && npm run dev`
  → http://localhost:3000 (backend URL in `public/env-config.js`).
- **Exit criteria:** all three running; Swagger UI loads at
  `/swagger-ui.html`.

### 2. Place a test order (day 1–2)
Browse the shop → add to cart → checkout with Stripe test keys → confirm
in `/order-confirm`, then find the order in the admin (Orders) and in the
`Order*` tables. **Exit criteria:** can narrate Flow A (doc 03) from memory.

### 3. Learn the API zones (day 2–3)
Browse Swagger; internalise public vs `/auth/**` (customer JWT) vs
`/private/**` (admin JWT). Reproduce a 401 by calling a private endpoint
without/with an expired token. **Exit criteria:** correctly classifies any
failing request into a zone and knows which token it needs.

### 4. Trace the money path (day 3–4)
Read: `ShoppingCartApi` → `ShoppingCartFacadeImpl` → checkout →
order facade + payment modules → `Order`/`OrderTotal`/transaction rows.
Know where totals, tax, shipping quotes, and payment transactions persist.
**Exit criteria:** given a failed-order ticket, lists the exact tables and
logs to check.

### 5. Learn the catalogue model (day 4–5)
Read: `Product` / `ProductDescription` (per language) /
`ProductAvailability` / prices / images; create a product in admin and fetch
it via public `GET /products`. **Exit criteria:** explains "product exists
but invisible" (store scope, language row, availability, search index).

### 6. Own the ops checklist (day 5)
MySQL backups, `hbm2ddl` behaviour per environment, Elasticsearch index
rebuild, CMS storage backend, per-merchant payment credentials, log
locations (sm-shop logs + browser network tab against `APP_BASE_URL`).

## What to prioritise (ranked)

1. Local reproduction ability — no ticket is understood until reproduced.
2. JWT zone mental model — resolves the largest ticket class fastest.
3. Checkout money-path — where sev-1s live.
4. Catalogue data model — where "missing data" tickets live.
5. Everything else (Drools, CMS backends, marketplace) — week 2+.

## First-ticket suggestions

- Change a product price in admin, verify on the shop (touches zones 1–5).
- Place an order with a bad Stripe key, read the backend error end to end.
- Create a second-language product description, observe fallback behaviour.
