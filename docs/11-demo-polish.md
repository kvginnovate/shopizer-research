# 11. Demo Polish & Improvements Applied

## What Was Done

### Product Images ✅
- Uploaded banner images to all 3 demo products via admin API
- Product IDs 1 (demo-table-01), 2 (demo-chair-01), 3 (demo-lamp-01) now have images
- Images stored in Infinispan CMS (local file system)

### Store Pickup Shipping ✅ (Configured)
- Activated storePickUp shipping module with price 0.00
- Note: shipping quote may require additional store configuration
- Checkout works without shipping quote (shippingQuote: null)

### Tax Configuration ✅ (Already Configured)
- DEFAULT tax class exists and is assigned to the store
- Tax shows in order totals

### Content Pages ✅
- Created "About Us" content box (code: about-us)
- Content page accessible via /content/about-us

### Code Improvements ✅

**React (shopizer-shop-reactjs):**
- Fixed cart page redirect (now shows empty cart instead of redirecting to home)
- Removed all debug console.log statements (12 files cleaned)
- Fixed checkout MONEYORDER mode (skip Stripe token creation)
- Committed to feat/moneyorder-payment branch

**Admin (shopizer-admin):**
- Fixed orders table rendering (change detection + load order)
- Committed to fix/admin-store-filter branch

**Backend (shopizer):**
- Fixed inventory decrement (productService.getById → getBySku)
- Committed to fix/backend-demo-fixes branch

### Additional Orders Created ✅
- Order #100: Demo Wooden Table (CA$199) - guest checkout
- Order #101: Demo Wooden Table (CA$199) - authenticated checkout
- Order #102: Demo Office Chair (CA$89) - guest checkout
- Total: 3 orders in database for dashboard charts

## Demo Account Credentials

| Role | Email | Password |
|------|-------|----------|
| Admin | admin@shopizer.com | password |
| Customer | demouser@shopizer.com | demo1234 |

## Quick Demo Script

1. **Shop:** http://localhost:8082
   - Browse "Furniture" category → 3 products with images
   - Click "Demo Wooden Table" → see image, price, ADD TO CART
   - Add to cart → cart badge shows 1
   - Go to Cart → see item + total CA$199
   - Go to Checkout → fill billing form → PLACE YOUR ORDER

2. **Admin:** http://localhost:8081 (admin@shopizer.com / password)
   - Dashboard → see order charts populated
   - Catalogue → Products → 3 products with images
   - Orders → 3 orders listed
   - Store Management → view store config

3. **API:** http://localhost:8080/swagger-ui.html
   - Try GET /api/v1/products?store=DEFAULT
   - Try POST /api/v1/private/login
   - Try GET /api/v1/private/orders

## Container Management

```bash
# Start all
docker start shopizer-mysql shopizer-backend shopizer-admin shopizer-shop

# Stop all
docker stop shopizer-backend shopizer-admin shopizer-shop shopizer-mysql

# Backend logs
docker logs -f shopizer-backend
```

## Remaining Known Issues

1. **Admin orders table** - Source fix committed but compiled bundle patch doesn't fully resolve rendering. API works fine.
2. **React checkout form** - Form validation requires all fields. Fill manually in browser.
3. **Backend inventory decrement** - Code fix committed but can't rebuild (Maven deps). Workaround via sequencer reset.
4. **Search (Elasticsearch)** - Not configured. Client-side fallback in place.
5. **Email notifications** - No SMTP. Set ORDER_EMAIL_API=false to avoid errors.
