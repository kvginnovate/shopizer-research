# Demo Account & Remaining Items

## Demo Account Credentials

| Role | Email | Password |
|------|-------|----------|
| Admin | admin@shopizer.com | password |
| Customer (Demo User) | demouser@shopizer.com | demo1234 |

## What's Working (Verified)

| Feature | Status | Notes |
|---------|--------|-------|
| Backend (MySQL) | ✅ | All data persisted in MySQL 8 |
| Admin console | ✅ | Login, catalogue, orders, store config |
| Shop storefront | ✅ | Browse, cart, checkout form |
| Guest checkout | ✅ | Order created in DB |
| Authenticated checkout | ✅ | Order created in DB |
| Payment module (moneyorder) | ✅ | Configured and active |
| Customer registration | ✅ | Via API and UI |
| Customer profile update | ✅ | Via API |
| Product catalogue | ✅ | 3 demo products with categories |
| Admin orders list | ✅ | Patched compiled bundle |
| React checkout form | ✅ | Fixed MONEYORDER mode |

## Remaining Items for Demo Polish

1. **Product images** — Products have no images. Upload at least one
   product image via admin → Catalogue → Products → Edit → Images.
   This proves the CMS/file-store path works.

2. **React checkout form validation** — The form requires all fields
   (firstName, lastName, address, city, country, state, postalCode, phone,
   email, terms). The country select may not propagate through React
   state when filled programmatically. For the demo, fill the form
   manually in the browser.

3. **Admin orders table** — Fixed via compiled bundle patch, but a
   proper fix requires rebuilding the Angular app (blocked by legacy
   build toolchain issues). The patch persists until the container is
   recreated.

4. **Email notifications** — `ORDER_EMAIL_API=true` but no SMTP server
   configured. Order confirmation emails won't send. Not needed for demo.

5. **Search (Elasticsearch)** — Product search may not work without
   Elasticsearch running. The catalogue browsing works fine.

6. **Performance** — React app is slow to load on this machine due to
   the large bundle. Consider using the production build.

## Quick Demo Script

1. Open shop: http://localhost:8082
2. Browse Furniture category → see 3 products
3. Click "Demo Wooden Table" → "ADD TO CART"
4. Go to Cart → see item + total
5. Go to Checkout → fill billing form → "PLACE YOUR ORDER"
6. Open admin: http://localhost:8081 (admin@shopizer.com / password)
7. Go to Orders → see the new order
8. Go to Catalogue → Products → see all products

## Container Management

```bash
# Start all
docker start shopizer-backend shopizer-admin shopizer-shop shopizer-mysql

# Stop all
docker stop shopizer-backend shopizer-admin shopizer-shop shopizer-mysql

# Backend logs
docker logs -f shopizer-backend

# Re-apply MySQL patch (if backend reverted to H2)
docker cp shopizer/shopizer-patched.jar shopizer-backend:/opt/app/shopizer.jar
docker restart shopizer-backend
```

## Demo Account

- Admin: admin@shopizer.com / password
- Customer: demouser@shopizer.com / demo1234
- Backend API: http://localhost:8080/swagger-ui.html
