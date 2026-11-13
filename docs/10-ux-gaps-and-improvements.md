# UX Gaps & Improvement Opportunities for Demo

## 1. Product Images Missing
**Gap:** All 3 demo products have no images. Product cards show broken image placeholders.
**Impact:** Storefront looks unfinished; product detail page has no visual.
**Fix:** Upload at least 1 product image per product via Admin → Catalogue → Products → Edit → Images. The CMS stores images in Infinispan (local) by default.

## 2. React Checkout Form Validation
**Gap:** The checkout form requires all fields (firstName, lastName, address, city, country, state, postalCode, phone, email, terms). The country select may not propagate through React state. The "PLACE YOUR ORDER" button does nothing if any field is invalid.
**Impact:** Demo checkout appears broken.
**Fix:** Pre-fill or simplify the form. Alternatively, reduce required fields for demo (make phone, company optional). Or add default values for demo products.

## 3. Admin Orders Table Not Rendering
**Gap:** The orders table header renders but no rows appear, even though the API returns 2 orders.
**Impact:** Admin "Orders" page looks empty during demo.
**Fix options:**
- Option A: Fix ng2-smart-table data binding (requires Angular rebuild - blocked by toolchain)
- Option B: Add a simple HTML table fallback in the template
- Option C: Show orders via the API/Swagger during demo instead

## 4. React App Performance
**Gap:** The React bundle is large (~2MB+). Page loads take 5-10 seconds on this machine.
**Impact:** Demo feels sluggish; browser automation times out.
**Fix:** 
- Enable code splitting (already has lazy routes but bundle is large)
- Remove unused dependencies (Nuvei, etc.)
- Use production build (npm run build) instead of dev server

## 5. No Product Search Results
**Gap:** The search page (`/search/:id`) may return no results because Elasticsearch is not configured.
**Impact:** Search feature appears broken.
**Fix:** Either configure Elasticsearch or modify SearchProduct.js to fall back to client-side filtering (already has fallback to all products when 0 results).

## 6. Cart Page Redirects to Home
**Gap:** The Cart page redirects to homepage if the cart fetch fails (stale cart cookie from backend restart).
**Impact:** After backend restart, cart appears empty.
**Fix:** Clear the cart cookie on backend restart, or handle 404 gracefully (show "cart is empty" instead of redirecting).

## 7. Missing Product Variants/Options
**Gap:** Products have no variants, options, or attributes configured. The demo products are simple single-SKU items.
**Impact:** Doesn't showcase the platform's variant/option capabilities.
**Fix:** Add at least one product with size/color variants via admin.

## 8. No Order Confirmation Email
**Gap:** `ORDER_EMAIL_API=true` but no SMTP server configured. Order confirmation emails won't send.
**Impact:** Email flow can't be demonstrated.
**Fix:** Configure a test SMTP (Gmail, Mailtrap) or set `ORDER_EMAIL_API=false` to avoid errors.

## 9. Missing Store Configuration
**Gap:** The store (DEFAULT) has basic info but no logo, theme, or custom branding.
**Impact:** Storefront header/footer show default template.
**Fix:** Upload a store logo and configure theme colors via admin → Store Management.

## 10. Admin Dashboard Charts
**Gap:** The admin dashboard has charts (orders, revenue) but they may show empty/no data.
**Impact:** Dashboard looks unpopulated during demo.
**Fix:** Create a few orders (already have 2) to populate the charts.

## 11. Content Pages Missing
**Gap:** The React app has content page routes but no content pages are configured.
**Impact:** "About Us", "Terms", etc. pages are empty.
**Fix:** Create at least one content page via admin → Content Management.

## 12. Shipping Configuration
**Gap:** Shipping modules (UPS, USPS, etc.) are configured but not active for the store. The shipping quote returns empty.
**Impact:** Checkout shipping section is empty.
**Fix:** Either activate a shipping module (store pickup) or use the "storePickUp" module which requires no external credentials.

## 13. Tax Configuration
**Gap:** Tax classes/rates may not be configured for the store.
**Impact:** Tax shows as $0.00 in order totals.
**Fix:** Configure a basic tax class via admin → Tax Management.

## 14. Multi-language Support
**Gap:** Only English is configured. The platform supports multiple languages.
**Impact:** Can't demonstrate i18n.
**Fix:** Not needed for demo, but good to know.

## 15. React App Console Errors
**Gap:** Console has warnings/errors (debug console.log, missing images).
**Impact:** Looks unprofessional if dev tools are open.
**Fix:** Remove debug console.log statements (already done in ProductDetail.js).

## Priority for Demo (Ranked)

### Must Fix (Demo Blockers)
1. **Product images** — Upload 1 image per product (5 min in admin)
2. **React checkout form** — Verify form submits with MONEYORDER (already fixed in code, needs manual test)
3. **Admin orders table** — Either fix rendering or use API/Swagger to show orders

### Should Fix (Demo Polish)
4. **Cart redirect** — Show "empty cart" instead of redirecting to home
5. **Store branding** — Upload logo, set theme color
6. **Create 2-3 more orders** — Populate admin dashboard charts
7. **Activate store pickup shipping** — Show shipping options in checkout

### Nice to Have
8. **Product variants** — Add size/color to one product
9. **Content page** — Create one "About Us" page
10. **Search fallback** — Already has client-side fallback
11. **Remove console.logs** — Clean up debug statements

## Quick Win Commands

```bash
# Activate store pickup shipping (via admin API)
curl -X POST http://localhost:8080/api/v1/private/modules/shipping/ \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer <admin-token>' \
  -d '{"code":"storePickUp","active":true}'

# Create a content page (via admin)
# Use admin UI → Content Management → Create

# Upload product image (via admin UI)
# Admin → Catalogue → Products → Edit → Images → Upload
```
