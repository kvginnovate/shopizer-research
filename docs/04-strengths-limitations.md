# 4. Strengths & Limitations

An engineering + support assessment from reading the code.

## Strengths (what's well-designed)

1. **True headless split.** Storefront and admin are thin API consumers; all
   commerce state and rules live behind versioned REST. Either UI can be
   rebuilt or replaced without touching the backend.
2. **Thorough domain modelling.** Per-language descriptions, price vs
   availability separation, order-total pipeline, status history, multi-store
   merchants — real commerce edge cases are already in the schema.
3. **Plugin SPIs for money-moving parts.** Payments (Stripe, PayPal ×2,
   Braintree, BeanStream, money order), shipping (UPS, USPS, store pickup,
   weight/distance rules), and tax sit behind integration modules, plus
   Drools decision tables for pricing/shipping rules. New providers don't
   require core rewrites.
4. **Facade layer.** Controllers stay thin; `store/facade/*` owns DTO mapping
   and cross-service orchestration — the right place to read when tracing a
   bug.
5. **Fast local stand-up.** Docker images for all three pieces, default admin
   creds, demo store `DEFAULT`, Swagger out of the box.

## Limitations (where time will be spent troubleshooting)

| Area | Risk | Mitigation |
|------|------|------------|
| Dependency age | Angular 11, React 16, Spring Boot 2.5, Java 11 — all past support; CVEs, `‑‑legacy-peer-deps` friction, Node 12/16-only builds | Plan the upgrade path early; pin build images |
| Schema management | `hbm2ddl.auto=update`, no migrations (Liquibase/Flyway) — env drift and risky upgrades | Day-one backups + migration tooling before prod |
| Search/cache ops | Elasticsearch + Infinispan assumed but optional; silent degradation | Health-check both per deploy; know the index rebuild |
| Payment config | Credentials per merchant; webhook/test-mode mismatches | Per-merchant credential checklist; test-mode drills |
| Auth confusion | `AUTH` (admin) vs `AUTH_CUSTOMER` roles, token expiry → silent logout | Memorise the three API zones (doc 02) |
| Multi-store leakage | `DEFAULT` vs merchant codes scoping carts/products/config | Always assert store code in bug reports |
| CMS/media | Image uploads depend on file/S3/GCS backend permissions | Verify storage backend in every environment |
| Docs | README-level only; truth is Swagger + code | Read facades, not wikis |

## Support hotspot forecast

1. Checkout/payment failures (provider credentials, webhooks, totals drift).
2. "Product/order missing" (wrong store, missing language row, unindexed search).
3. Auth/401 loops (expired token, wrong zone, role mismatch).
4. Environment issues (schema drift, ES/Infinispan health, CMS permissions).
5. Build issues (legacy Node/Angular/React toolchains on new machines).
