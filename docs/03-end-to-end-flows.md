# 3. How It Works End-to-End

Two core flows, traced to the actual code.

## Flow A — Customer places an order (storefront)

```
Shopper            React shop                    sm-shop backend                     MySQL / providers
  │  add to cart      │  cartActions.js               │                                     │
  │──────────────────▶│ POST /api/v1/cart             │                                     │
  │                   │──────────────────────────────▶│ ShoppingCartApi →                   │
  │                   │                               │ ShoppingCartFacadeImpl → persists   │──▶ ShoppingCart + items
  │                   │◀──────── cart code ───────────│                                     │    (code → cookie)
  │  checkout         │  Checkout.js                  │                                     │
  │──────────────────▶│ GET /cart/{code}              │──▶ cart + items ───────────────────▶│
  │                   │ GET auth/customer/profile     │──▶ customer/address ───────────────▶│
  │                   │ GET shipping / total/         │──▶ ShippingQuoteService +          │
  │                   │                               │    order-total modules + Drools ───▶│    quotes, tax, totals
  │  pay (Stripe)     │  Stripe Elements → token      │                                     │
  │──────────────────▶│ POST cart/{id}/checkout       │──▶ payment module authorise/capture │──▶ Stripe/PayPal API
  │                   │  (address + payment token)    │──▶ Order*, OrderTotal, transaction  │──▶ Order rows, inventory ↓
  │                   │◀──────── order id ────────────│                                     │
  │                   │  deleteAllFromCart → /order-confirm + toast                         │
```

- **Files:** `src/redux/actions/cartActions.js`, `src/pages/other/Checkout.js`,
  `src/util/webService.js`, `src/util/constant.js` (endpoint catalogue);
  backend `store/api/v1/shoppingCart/ShoppingCartApi.java`,
  `store/facade/shoppingCart/ShoppingCartFacadeImpl.java`,
  `business/services/order/*`, `business/modules/integration/payment/impl/*`.
- **Failure modes to know:** expired cart code (cookie), totals mismatch
  (rules changed mid-checkout), payment authorisation failure (provider
  credentials per merchant), inventory shortfall at order creation.

## Flow B — Admin manages the catalogue (admin console)

```
Admin              Angular admin                 sm-shop backend                     MySQL / CMS / search
  │  login            │  auth.service.ts              │                                     │
  │──────────────────▶│ POST /v1/private/login        │──▶ admin auth → JWT ───────────────▶│    users/roles
  │                   │◀──────── Bearer token ────────│   (TokenService → localStorage)     │
  │  create product   │  catalogue pages →            │                                     │
  │──────────────────▶│ POST /private/product         │──▶ ProductApi → product facade/     │
  │                   │ (+ images via ProductImageApi) │    service → JPA                    │──▶ Product, descriptions
  │                   │                               │                                     │    (per language), prices,
  │                   │                               │    CMS store                        │    availability, images
  │                   │◀──────── product id ──────────│    search index                     │──▶ ES index, cache
  │  verify           │  GET /products, /product/{id} (public) ──▶ same rows ◀─────────────│
```

- **Files:** `src/app/pages/auth/services/auth.service.ts`,
  `src/app/pages/shared/services/crud.service.ts`,
  `src/app/pages/shared/interceptors/auth.interceptor.ts`;
  backend `store/api/v1/product/ProductApi.java`
  (`/private/product` + `/auth/*` + public `/products`),
  `ProductImageApi`, `ProductPriceApi`, `ProductInventoryApi`.
- **Failure modes to know:** 401s (expired token — interceptor logs out),
  missing per-language descriptions ("product exists but name blank"),
  image upload failures (CMS backend permissions), product invisible on the
  shop (not indexed / not available for that store).

## Cross-cutting notes

- Auth is always `Authorization: Bearer <token>` from localStorage; zone
  (`private` vs `auth` vs public) determines which role is required.
- Store/merchant code (`DEFAULT` unless multi-store) scopes carts, products,
  and config — most "data missing" tickets are wrong-store bugs.
- Swagger UI on the running backend (`/swagger-ui.html`) is the authoritative
  endpoint reference; `constant.js` (shop) and the 148-endpoint usage list in
  admin `src/` are the client-side mirrors.
