# Master Prompt — Shopizer Interview Demo Setup

> This document contains the complete prompt and step-by-step instructions to
> recreate the entire Shopizer interview demo from scratch. It covers research,
> fixes, Docker deployment, mock data, and documentation.

---

## INITIAL ASSIGNMENT PROMPT

```
Please explore the following open-source e-commerce platform:

git clone https://github.com/shopizer-ecommerce/shopizer-admin.git
git clone https://github.com/shopizer-ecommerce/shopizer-shop-reactjs.git
git clone https://github.com/shopizer-ecommerce/shopizer.git

I'd like you to look at the shopizer project and get familiarized with the overall project:

1. Platform Overview — What does Shopizer do? Who is it built for? What problem does it solve?
2. Technical Architecture — How is the system structured? Walk us through the key components and how they connect.
3. How It Works End-to-End — Pick one or two core user flows and explain what happens technically under the hood.
4. Strengths & Limitations — From a support and engineering perspective, what stands out as well-designed? Where might your team spend most of their time troubleshooting?
5. If You Were Onboarding Your Team — How would you get a new support engineer up to speed on this platform?

You are absolutely welcome — and encouraged — to use AI tools to assist your research.
```

---

## PHASE 1: RESEARCH & DOCUMENTATION

### Step 1: Clone Repositories
```bash
git clone https://github.com/shopizer-ecommerce/shopizer.git
git clone https://github.com/shopizer-ecommerce/shopizer-admin.git
git clone https://github.com/shopizer-ecommerce/shopizer-shop-reactjs.git
```

### Step 2: Survey Backend (Java Spring Boot)
- Read README.md for platform overview
- Check pom.xml for Maven module structure (sm-core-model, sm-core-modules, sm-core, sm-shop-model, sm-shop)
- Explore sm-shop/src/main/java/com/salesmanager/shop/store/api/v1/ for REST controllers
- Check sm-shop/src/main/java/com/salesmanager/shop/store/security/ for JWT auth
- Check sm-core/src/main/java/com/salesmanager/core/business/configuration/DataConfiguration.java for DB config
- Check sm-core/src/main/java/com/salesmanager/core/business/modules/integration/ for payment/shipping/tax modules
- Check application.properties for configuration

### Step 3: Survey Frontends
- React: package.json, src/App.js (routes), src/util/webService.js (API client), src/redux/ (state), src/pages/ (pages)
- Angular: package.json, src/app/app-routing.module.ts, src/app/pages/shared/services/crud.service.ts, src/app/pages/shared/interceptors/auth.interceptor.ts

### Step 4: Create Documentation
Create `docs/` folder with 11 files:
1. `01-platform-overview.md` — What/who/problem
2. `02-technical-architecture.md` — System map, modules, APIs, data layer, integrations
3. `03-end-to-end-flows.md` — Checkout + admin catalogue flows with diagrams
4. `04-strengths-limitations.md` — Engineering assessment
5. `05-team-onboarding.md` — Week-1 onboarding path
6. `06-docker-setup.md` — Docker deployment guide
7. `07-bug-fixes.md` — Two frontend bugs found and fixed
8. `08-demo-readiness.md` — Demo account, remaining items, quick script
9. `09-secrets-and-credentials.md` — All passwords, connection strings
10. `10-ux-gaps-and-improvements.md` — UX gaps and improvement priorities
11. `11-demo-polish.md` — Demo polish applied, quick demo script

Also create `SHOPIZER-BRIEFING.md` (condensed 5-part version) and `docs/README.md` (index).

---

## PHASE 2: DOCKER DEPLOYMENT

### Step 1: Start Docker Desktop
```bash
# Windows
"C:/Users/<user>/AppData/Local/Programs/DockerDesktop/Docker Desktop.exe" &
# Wait for docker ps to work
```

### Step 2: Pull Images
```bash
docker pull shopizerecomm/shopizer:latest
docker pull shopizerecomm/shopizer-admin:latest
docker pull shopizerecomm/shopizer-shop-reactjs:latest
docker pull mysql:8.0.42
```

### Step 3: Create Network
```bash
docker network create shopizer-net
```

### Step 4: Start MySQL
```bash
docker run -d --name shopizer-mysql --network shopizer-net \
  -e MYSQL_ROOT_PASSWORD=rootpw123 \
  -e MYSQL_DATABASE=SALESMANAGER \
  -e MYSQL_USER=shopizer \
  -e MYSQL_PASSWORD=shopizerpw123 \
  -p 3306:3306 mysql:8.0.42
```

### Step 5: Patch Backend Jar for MySQL
Extract jar from image, replace `BOOT-INF/classes/database.properties` with MySQL config, copy back.

db-mysql.properties:
```properties
db.jdbcUrl=jdbc:mysql://shopizer-mysql:3306/SALESMANAGER?autoReconnect=true&serverTimeZone=UTC&useUnicode=true&characterEncoding=UTF-8
db.user=shopizer
db.password=shopizerpw123
db.driverClass=com.mysql.cj.jdbc.Driver
hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
db.show.sql=false
db.preferredTestQuery=SELECT 1
db.schema=SALESMANAGER
hibernate.hbm2ddl.auto=update
db.initialPoolSize=5
db.minPoolSize=5
db.maxPoolSize=10
```

Patch script (Python):
```python
import zipfile, os
SRC = 'shopizer-original.jar'
TMP = 'shopizer-patched.jar'
TARGET = 'BOOT-INF/classes/database.properties'
with open('db-mysql.properties', 'rb') as f:
    new_data = f.read()
with zipfile.ZipFile(SRC, 'r') as zin, zipfile.ZipFile(TMP, 'w', zipfile.ZIP_DEFLATED) as zout:
    for item in zin.infolist():
        if item.filename == TARGET:
            continue
        zout.writestr(item, zin.read(item.filename))
    zout.writestr(TARGET, new_data)
os.replace(TMP, SRC.replace('original', 'patched'))
```

### Step 6: Start Backend
```bash
docker run -d --name shopizer-backend --network shopizer-net -p 8080:8080 shopizerecomm/shopizer:latest
docker cp shopizer-patched.jar shopizer-backend:/opt/app/shopizer.jar
docker network connect shopizer-net shopizer-backend
docker restart shopizer-backend
# Wait ~60s for Spring Boot to boot
```

### Step 7: Start Admin
```bash
docker run -d --name shopizer-admin -e "APP_BASE_URL=http://localhost:8080/api" -p 8081:80 shopizerecomm/shopizer-admin:latest
```

### Step 8: Start Shop
```bash
docker run -d --name shopizer-shop -e "APP_MERCHANT=DEFAULT" -e "APP_BASE_URL=http://localhost:8080" -p 8082:80 shopizerecomm/shopizer-shop-reactjs:latest
```

### Step 9: Verify
```bash
curl http://localhost:8080/api/v1/store/DEFAULT  # should return store JSON
curl -s -o /dev/null -w '%{http_code}' http://localhost:8081/  # 200
curl -s -o /dev/null -w '%{http_code}' http://localhost:8082/  # 200
```

---

## PHASE 3: MOCK DATA & CONFIGURATION

### Step 1: Configure Payment Module (MoneyOrder)
```bash
TOKEN=$(curl -s -X POST http://localhost:8080/api/v1/private/login -H 'Content-Type: application/json' -d '{"username":"admin@shopizer.com","password":"password"}' | python3 -c 'import json,sys; print(json.load(sys.stdin)["token"])')
curl -s -X POST http://localhost:8080/api/v1/private/modules/payment/ -H "Authorization: Bearer $TOKEN" -H 'Content-Type: application/json' -d '{"code":"moneyorder","active":true,"defaultSelected":true,"integrationKeys":{"address":"123 Demo Street, Montreal, QC"}}'
```

### Step 2: Create Demo Customer Account
```bash
curl -s -X POST 'http://localhost:8080/api/v1/customer/register?store=DEFAULT&lang=en' -H 'Content-Type: application/json' -d '{
  "userName": "demouser@shopizer.com",
  "password": "demo1234",
  "emailAddress": "demouser@shopizer.com",
  "gender": "M",
  "language": "en",
  "billing": {"country":"CA","stateProvince":"QC","firstName":"Demo","lastName":"User","address":"123 Demo Street","city":"Montreal","postalCode":"H2H2H2","phone":"5145559999"},
  "delivery": {"country":"CA","stateProvince":"QC","firstName":"Demo","lastName":"User","address":"123 Demo Street","city":"Montreal","postalCode":"H2H2H2","phone":"5145559999"}
}'
```

### Step 3: Create Product Categories
```bash
# Living Room (id=50), Bedroom (id=51), Office (id=52)
for cat in "living-room:Living Room" "bedroom:Bedroom" "office:Office"; do
  code="${cat%%:*}"; name="${cat##*:}"
  curl -s -X POST 'http://localhost:8080/api/v1/private/category?store=DEFAULT&lang=en' -H "Authorization: Bearer $TOKEN" -H 'Content-Type: application/json' -d "{\"code\":\"$code\",\"visible\":true,\"sortOrder\":1,\"descriptions\":[{\"language\":\"en\",\"name\":\"$name\",\"description\":\"$name furniture\",\"friendlyUrl\":\"$code\",\"title\":\"$name\"}]}"
done
```

### Step 4: Create Products with Images
Create 8 products (demo-table-01, demo-chair-01, demo-lamp-01, sofa-01, bookshelf-01, desk-01, chair-02, lamp-02) with categories and upload images.

### Step 5: Create Orders
Create 3-4 orders via guest checkout API to populate dashboard.

### Step 6: Configure Tax (Already Done)
DEFAULT tax class exists. No additional config needed.

---

## PHASE 4: CODE FIXES

### Fix 1: React Checkout.js (MONEYORDER mode)
File: `shopizer-shop-reactjs/src/pages/other/Checkout.js`
Change `onSubmitOrder` to skip Stripe token creation in MONEYORDER mode:
```javascript
if (window._env_.APP_PAYMENT_TYPE === 'MONEYORDER') {
  onPayment(data, null)
  return
}
```

### Fix 2: React Cart.js (empty cart instead of redirect)
File: `shopizer-shop-reactjs/src/pages/other/Cart.js`
- Remove always-redirect setTimeout
- Change catch block to set empty cart instead of redirecting

### Fix 3: React console.log removal
Remove all debug console.log from 12 files.

### Fix 4: Angular order-list.component.ts (change detection)
File: `shopizer-admin/src/app/pages/orders/order-list/order-list.component.ts`
- Inject ChangeDetectorRef
- Swap order: source.load() first, then loadingList=false, then cdr.detectChanges()

### Fix 5: Backend OrderServiceImpl (inventory decrement)
File: `shopizer/sm-shop/src/main/java/com/salesmanager/shop/store/controller/order/facade/OrderFacadeImpl.java`
- Change `productService.getById(orderProduct.getId())` to `productService.getBySku(orderProduct.getSku(), store)`

### Fix 6: Backend ImagesController (guard missing image)
File: `shopizer/sm-shop/src/main/java/com/salesmanager/shop/controller/ImagesController.java`
- Guard against missing static/not-found.png

---

## PHASE 5: DOCKER IMAGE REBUILD

### React (shop)
```bash
cd shopizer-shop-reactjs
docker build -t shopizer-shop-fixed .
docker rm -f shopizer-shop
docker run -d --name shopizer-shop -e "APP_MERCHANT=DEFAULT" -e "APP_BASE_URL=http://localhost:8080" -p 8082:80 shopizer-shop-fixed
```

### Admin (bundle patch only - build blocked by toolchain)
```bash
# Patch compiled bundle in running container
docker cp admin-dist/18.68c3e32d5f6ba64567a4.js shopizer-admin:/usr/share/nginx/html/18.68c3e32d5f6ba64567a4.js
```

### Backend (jar patch only - build blocked by Maven deps)
```bash
# Patch jar in running container
docker cp shopizer-patched.jar shopizer-backend:/opt/app/shopizer.jar
docker restart shopizer-backend
```

---

## PHASE 6: GIT & GITHUB

### Create Research Repo
```bash
gh repo create shopizer-research --public --description "Shopizer interview demo research"
```

### Push Documentation
```bash
cd shopizer-research
git init -b main
git add -A
git commit -m "Shopizer platform research: briefing + docs"
git push origin main
```

### Create PRs on Forks
- shopizer: fix/backend-demo-fixes → 3.2.7
- shopizer-admin: fix/admin-store-filter → main
- shopizer-shop-reactjs: feat/moneyorder-payment → main
- shopizer-shop-reactjs: feat/shop-polish → main

### Merge Order
1. shopizer PR #1
2. shopizer-admin PR #1
3. shopizer-shop-reactjs PR #2
4. shopizer-shop-reactjs PR #3

---

## CREDENTIALS SUMMARY

| Service | Username | Password |
|---------|----------|----------|
| Admin Console | admin@shopizer.com | password |
| Customer Demo | demouser@shopizer.com | demo1234 |
| MySQL root | root | rootpw123 |
| MySQL app | shopizer | shopizerpw123 |

## PORTS

| Service | Port |
|---------|------|
| Backend API | 8080 |
| MySQL | 3306 |
| Admin | 8081 |
| Shop | 8082 |

## QUICK DEMO SCRIPT

1. Open shop: http://localhost:8082
2. Browse "Furniture" → 8 products with images
3. Click "Demo Wooden Table" → "ADD TO CART"
4. Go to Cart → see item + total CA$199
5. Go to Checkout → fill billing form → "PLACE YOUR ORDER"
6. Open admin: http://localhost:8081 (admin@shopizer.com / password)
7. Go to Orders → see orders
8. Go to Catalogue → Products → see all 8 products with images
9. Go to Dashboard → see order charts populated
