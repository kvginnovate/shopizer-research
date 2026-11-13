# Setup Prompt for New Machine

> **Copy-paste this entire prompt into any AI coding assistant on a new machine.**
> It contains everything needed to recreate the Shopizer interview demo from scratch.
> Estimated time: 2-3 hours (mostly waiting for builds/downloads).

---

## THE PROMPT (Copy everything below this line)

---

I need you to set up a complete end-to-end demo of the Shopizer open-source e-commerce platform for a technical interview. Follow these steps in order. All commands should be run in a terminal.

### PREREQUISITES
- Docker Desktop installed and running
- Git installed
- Python 3 installed
- Node.js (any version, for npm only)
- Java 11+ (optional, for source builds)

### STEP 1: CLONE REPOSITORIES
```bash
mkdir -p ~/shopizer-demo && cd ~/shopizer-demo
git clone https://github.com/shopizer-ecommerce/shopizer.git
git clone https://github.com/shopizer-ecommerce/shopizer-admin.git
git clone https://github.com/shopizer-ecommerce/shopizer-shop-reactjs.git
```

### STEP 2: DOCKER DEPLOYMENT

#### 2.1 Create Network
```bash
docker network create shopizer-net
```

#### 2.2 Start MySQL
```bash
docker run -d --name shopizer-mysql --network shopizer-net \
  -e MYSQL_ROOT_PASSWORD=rootpw123 \
  -e MYSQL_DATABASE=SALESMANAGER \
  -e MYSQL_USER=shopizer \
  -e MYSQL_PASSWORD=shopizerpw123 \
  -p 3306:3306 mysql:8.0.42
```
Wait 30s for MySQL to initialize.

#### 2.3 Patch Backend Jar for MySQL
Create `db-mysql.properties`:
```
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

Patch the jar:
```bash
docker create --name shopizer-extract shopizerecomm/shopizer:latest
docker cp shopizer-extract:/opt/app/shopizer.jar shopizer-original.jar
docker rm shopizer-extract

python3 -c "
import zipfile, os
SRC='shopizer-original.jar'
TMP='shopizer-patched.jar'
TARGET='BOOT-INF/classes/database.properties'
with open('db-mysql.properties','rb') as f: new_data=f.read()
with zipfile.ZipFile(SRC,'r') as zin, zipfile.ZipFile(TMP,'w',zipfile.ZIP_DEFLATED) as zout:
    for item in zin.infolist():
        if item.filename==TARGET: continue
        zout.writestr(item, zin.read(item.filename))
    zout.writestr(TARGET, new_data)
os.replace(TMP, 'shopizer-patched.jar')
print('patched')
"
```

#### 2.4 Start Backend
```bash
docker run -d --name shopizer-backend --network shopizer-net -p 8080:8080 shopizerecomm/shopizer:latest
docker cp shopizer-patched.jar shopizer-backend:/opt/app/shopizer.jar
docker network connect shopizer-net shopizer-backend
docker restart shopizer-backend
# Wait 90s for Spring Boot to boot
```

#### 2.5 Start Admin
```bash
docker run -d --name shopizer-admin -e "APP_BASE_URL=http://localhost:8080/api" -p 8081:80 shopizerecomm/shopizer-admin:latest
```

#### 2.6 Start Shop (Rebuild with Fixes)
First, apply code fixes to shopizer-shop-reactjs (see Step 4), then:
```bash
cd shopizer-shop-reactjs
docker build -t shopizer-shop-fixed .
docker run -d --name shopizer-shop -e "APP_MERCHANT=DEFAULT" -e "APP_BASE_URL=http://localhost:8080" -p 8082:80 shopizer-shop-fixed
```

#### 2.7 Verify
```bash
curl http://localhost:8080/api/v1/store/DEFAULT  # should return store JSON
curl -s -o /dev/null -w '%{http_code}' http://localhost:8081/  # 200
curl -s -o /dev/null -w '%{http_code}' http://localhost:8082/  # 200
```

### STEP 3: MOCK DATA & CONFIGURATION

#### 3.1 Admin Login
```bash
TOKEN=$(curl -s -X POST http://localhost:8080/api/v1/private/login -H 'Content-Type: application/json' -d '{"username":"admin@shopizer.com","password":"password"}' | python3 -c 'import json,sys; print(json.load(sys.stdin)["token"])')
echo "Token: $TOKEN"
```

#### 3.2 Configure Payment Module (MoneyOrder)
```bash
curl -s -X POST http://localhost:8080/api/v1/private/modules/payment/ -H "Authorization: Bearer $TOKEN" -H 'Content-Type: application/json' -d '{"code":"moneyorder","active":true,"defaultSelected":true,"integrationKeys":{"address":"123 Demo Street, Montreal, QC"}}'
```

#### 3.3 Create Demo Customer Account
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

#### 3.4 Create Product Categories
```bash
for cat in "living-room:Living Room" "bedroom:Bedroom" "office:Office"; do
  code="${cat%%:*}"; name="${cat##*:}"
  curl -s -X POST 'http://localhost:8080/api/v1/private/category?store=DEFAULT&lang=en' -H "Authorization: Bearer $TOKEN" -H 'Content-Type: application/json' -d "{\"code\":\"$code\",\"visible\":true,\"sortOrder\":1,\"descriptions\":[{\"language\":\"en\",\"name\":\"$name\",\"description\":\"$name furniture\",\"friendlyUrl\":\"$code\",\"title\":\"$name\"}]}"
  echo ""
done
```

#### 3.5 Create Products (via API)
Create products using POST /api/v1/private/product?store=DEFAULT&lang=en with JSON body containing sku, price, quantity, descriptions, categories.

Products to create:
- demo-table-01: Demo Wooden Table, $199, category 1 (Furniture)
- demo-chair-01: Demo Office Chair, $89, category 1
- demo-lamp-01: Demo Desk Lamp, $29, category 1
- sofa-01: Modern Sofa, $599, category 50 (Living Room)
- bookshelf-01: Wooden Bookshelf, $149, category 50
- desk-01: Office Desk, $299, category 52 (Office)
- chair-02: Ergonomic Chair, $249, category 52
- lamp-02: Floor Lamp, $79, category 50

#### 3.6 Upload Product Images
For each product, upload an image via:
```bash
curl -s -X POST "http://localhost:8080/api/v1/private/products/{id}/images?store=DEFAULT" -H "Authorization: Bearer $TOKEN" -F 'file=@image.jpg' -F 'defaultImage=true'
```

#### 3.7 Create Orders
Create 3-4 orders via guest checkout to populate dashboard.

#### 3.8 Configure Tax (Already Done)
DEFAULT tax class exists. No additional config needed.

### STEP 4: CODE FIXES

#### Fix 1: React Checkout.js (MONEYORDER mode)
File: shopizer-shop-reactjs/src/pages/other/Checkout.js
In onSubmitOrder function, add at the beginning:
```javascript
if (window._env_.APP_PAYMENT_TYPE === 'MONEYORDER') {
  onPayment(data, null)
  return
}
```

#### Fix 2: React Cart.js (empty cart instead of redirect)
File: shopizer-shop-reactjs/src/pages/other/Cart.js
- Remove the always-redirect setTimeout in fetchData
- Change catch block to set empty cart instead of redirecting

#### Fix 3: React console.log removal
Remove all debug console.log from 12 files:
- src/App.js
- src/pages/content/Content.js
- src/pages/other/Checkout.js
- src/pages/other/MyAccount.js
- src/pages/other/OrderDetails.js
- src/pages/other/RecentOrder.js
- src/pages/other/ResetPassword.js
- src/pages/product-details/ProductDetail.js
- src/redux/actions/cartActions.js
- src/redux/actions/userAction.js
- src/pages/other/LoginRegister.js

#### Fix 4: Angular order-list.component.ts (change detection)
File: shopizer-admin/src/app/pages/orders/order-list/order-list.component.ts
- Inject ChangeDetectorRef in constructor
- Reorder: source.load() first, then loadingList=false, then cdr.detectChanges()

#### Fix 5: Backend OrderServiceImpl (inventory decrement)
File: shopizer/sm-shop/src/main/java/com/salesmanager/shop/store/controller/order/facade/OrderFacadeImpl.java
- Change: productService.getById(orderProduct.getId()) → productService.getBySku(orderProduct.getSku(), store)

#### Fix 6: Backend ImagesController (guard missing image)
File: shopizer/sm-shop/src/main/java/com/salesmanager/shop/controller/ImagesController.java
- Guard against missing static/not-found.png

### STEP 5: END-TO-END VERIFICATION
Using Browser OS Neo MCP or manual browser:
1. Open shop: http://localhost:8082 → verify homepage loads
2. Browse category → verify products show with images
3. Click product → verify detail page
4. Add to cart → verify cart badge updates
5. Go to cart → verify items and total
6. Go to checkout → fill form → place order
7. Open admin: http://localhost:8081 → verify dashboard, products, orders
8. Check API: http://localhost:8080/swagger-ui.html

### STEP 6: DOCUMENTATION
Create docs/ folder with 13 files (see master prompt for details).
Create SHOPIZER-BRIEFING.md (condensed version).
Create docs/README.md (index).

### STEP 7: GITHUB
- Create public repo: shopizer-research
- Push all documentation
- Create PRs on forks
- Merge in order: backend → admin → shop (PR2) → shop (PR3)

### CREDENTIALS
| Service | Username | Password |
|---------|----------|----------|
| Admin | admin@shopizer.com | password |
| Customer | demouser@shopizer.com | demo1234 |
| MySQL root | root | rootpw123 |
| MySQL app | shopizer | shopizerpw123 |

### PORTS
| Service | Port |
|---------|------|
| Backend | 8080 |
| MySQL | 3306 |
| Admin | 8081 |
| Shop | 8082 |

### QUICK DEMO SCRIPT
1. Open shop: http://localhost:8082 → Browse Furniture → 8 products with images
2. Click "Demo Wooden Table" → "ADD TO CART"
3. Go to Cart → see item + total CA$199
4. Go to Checkout → fill billing form → "PLACE YOUR ORDER"
5. Open admin: http://localhost:8081 → Orders → see orders
6. Go to Catalogue → Products → see all 8 products with images
7. Go to Dashboard → see order charts

---

## NOTES FOR THE INTERVIEWER

- All work was done using AI tools (Claude, Copilot) as research assistants
- Every claim was verified against actual source code
- Two frontend bugs were found and fixed during live testing
- One backend bug was found (inventory decrement using wrong ID)
- The demo uses mock payment (MoneyOrder) — no external credentials needed
- All documentation is in the public GitHub repo
- The master prompt (docs/12-master-prompt.md) can recreate everything from scratch
