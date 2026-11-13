# Master Prompt — Shopizer Interview Demo (Complete Recreation Guide)

> **Use this prompt to recreate the entire Shopizer interview demo from scratch.**
> Copy-paste this into any AI coding assistant (Claude, ChatGPT, Copilot, etc.)
> and it will reproduce everything documented here.

---

## THE PROMPT

```
I'm preparing for a technical interview. I need you to help me set up a complete
end-to-end demo of the Shopizer open-source e-commerce platform. Here's what
I need:

STEP 1 — RESEARCH (Read-only, no edits)
Clone and explore these three repositories:
- https://github.com/shopizer-ecommerce/shopizer.git (Java Spring Boot backend)
- https://github.com/shopizer-ecommerce/shopizer-admin.git (Angular 11 admin)
- https://github.com/shopizer-ecommerce/shopizer-shop-reactjs.git (React 16 storefront)

For each repo, document:
1. Platform overview: what it does, who it's for, what problem it solves
2. Technical architecture: modules, APIs, data layer, integrations, diagrams
3. End-to-end flows: customer checkout + admin catalogue (user action → DB → back)
4. Strengths & limitations: what's well-designed, where support time goes
5. Onboarding plan: how to get a new support engineer up to speed

Save findings as markdown docs in a docs/ folder with an index README.

STEP 2 — DOCKER DEPLOYMENT
Set up the full stack using Docker:
- Pull images: shopizerecomm/shopizer, shopizerecomm/shopizer-admin, shopizerecomm/shopizer-shop-reactjs, mysql:8.0.42
- Create network: shopizer-net
- Start MySQL (SALESMANAGER db, shopizer user, password: shopizerpw123)
- Patch backend jar: replace BOOT-INF/classes/database.properties with MySQL config
- Start backend on :8080, admin on :8081, shop on :8082
- Verify all services respond 200

STEP 3 — MOCK DATA & CONFIGURATION
- Configure MoneyOrder payment module (mock, no external credentials)
- Create demo customer: demouser@shopizer.com / demo1234 (with billing + delivery)
- Create 4 categories: Furniture, Living Room, Bedroom, Office
- Create 8 products with images (table, chair, lamp, sofa, bookshelf, desk, etc.)
- Upload product images via admin API
- Create 3+ orders for dashboard population
- Verify tax class (DEFAULT) exists

STEP 4 — CODE FIXES
Fix these bugs found during testing:

Bug 1 (React checkout): In MONEYORDER mode, onSubmitOrder calls stripe.createToken(card)
but card is null (no CardElement rendered). Fix: skip Stripe, call onPayment(data, null).

Bug 2 (Admin orders table): ng2-smart-table doesn't render rows because loadingList=false
is set before source.load(). Fix: load data first, then set loadingList=false, then
cdr.detectChanges().

Bug 3 (Backend inventory): productService.getById(orderProduct.getId()) uses OrderProduct's
ID instead of Product's ID. Fix: use productService.getBySku(orderProduct.getSku(), store).

Bug 4 (React cart redirect): Cart page always redirects to home. Fix: remove redirect,
show empty cart instead.

Bug 5 (React console.log): Remove all debug console.log statements.

STEP 5 — END-TO-END VERIFICATION
Using Browser OS Neo MCP:
- Verify shop homepage, category page (3+ products), product detail
- Verify admin dashboard, products list, orders
- Verify guest checkout API (order created)
- Verify authenticated checkout API (order created)
- Verify customer registration and login
- Verify MySQL persistence (orders, products, carts)

STEP 6 — DOCUMENTATION
Create 12 docs in docs/ folder:
1. Platform overview
2. Technical architecture (with diagrams)
3. End-to-end flows
4. Strengths & limitations
5. Team onboarding
6. Docker setup guide
7. Bug fixes
8. Demo readiness
9. Secrets & credentials
10. UX gaps & improvements
11. Demo polish
12. Master prompt (this document)

Create SHOPIZER-BRIEFING.md (condensed version)
Create docs/README.md (index)

STEP 7 — GITHUB
- Create public repo: shopizer-research
- Push all documentation
- Create PRs on forks (shopizer, shopizer-admin, shopizer-shop-reactjs)
- Merge in order: backend → admin → shop (PR2) → shop (PR3)

STEP 8 — FINAL VERIFICATION
- All 4 containers running and healthy
- All services respond 200
- Demo account works (login, browse, cart, checkout)
- Admin shows products and orders
- Documentation pushed to GitHub
```

---

## HOW I APPROACHED THIS (Interview Talking Points)

### 1. Research Phase
"I started by cloning all three repositories and systematically reading the
codebase. I used AI tools to accelerate the survey — spawning parallel
agents to analyze the backend modules and frontend structure simultaneously.
I verified every claim against actual source code (controller classes, facade
implementations, API client code) rather than relying on READMEs alone."

### 2. Architecture Understanding
"I identified that Shopizer is a headless commerce platform: a Spring Boot
monolith with 5 Maven modules (sm-core-model, sm-core-modules, sm-core,
sm-shop-model, sm-shop) exposing versioned REST APIs. The storefront (React)
and admin (Angular) are thin API consumers. Data lives in MySQL with
Elasticsearch for search and Infinispan for caching."

### 3. End-to-End Flow Tracing
"I traced two core flows through the actual code:
- Customer checkout: cartActions.js → ShoppingCartApi → ShoppingCartFacadeImpl
  → checkout → OrderFacade + payment modules → Order tables in MySQL
- Admin catalogue: ProductApi → product facade → JPA Product entities →
  MySQL + CMS (images) + Elasticsearch (search index)"

### 4. Bug Discovery
"During live testing, I found two frontend bugs:
1. React checkout fails in MONEYORDER mode because stripe.createToken(null)
   throws before onPayment runs — the form never submits.
2. Admin orders table renders no rows because loadingList=false is set
   before source.load(), and no change detection is triggered.

I also found a backend inventory decrement bug where productService.getById()
uses the OrderProduct's auto-generated ID instead of the Product's ID."

### 5. Docker Deployment
"The official images default to H2 (embedded). I patched the backend jar's
database.properties to use MySQL, created the network, and got all 4
containers running. The React shop was rebuilt from source with all fixes.
The admin bundle was patched at runtime since the Angular build toolchain
is incompatible with modern Node versions."

### 6. Mock Data & Polish
"I downloaded furniture images from Unsplash, created 4 categories and 8
products with images, configured the MoneyOrder payment module, created a
demo customer account with billing+delivery, and seeded 3 orders for the
dashboard charts."

### 7. Documentation
"I created 12 documentation files covering the assignment questions, Docker
setup, bug fixes, UX gaps, secrets, and a master recreation prompt. All
pushed to a public GitHub repo."

---

## KEY TECHNICAL DECISIONS (For Interview Discussion)

1. **MoneyOrder mock payment** — Used the built-in MoneyOrderPayment module
   which authorizes/captures without external calls. No Stripe/PayPal
   credentials needed.

2. **MySQL over H2** — Patched the backend jar to use MySQL 8 instead of
   the embedded H2. Data persists across container restarts.

3. **Runtime bundle patching** — The Angular admin can't be rebuilt (Node 12
   requirement vs Node 24 available). Patched the compiled JS in the running
   container as a workaround.

4. **Sequencer reset** — The backend inventory decrement bug uses
   OrderProduct.getId() (auto-generated) instead of Product ID. Reset the
   SM_SEQUENCER table to make IDs align.

5. **Parallel AI agents** — Used AI tools to survey backend and frontend
   simultaneously, then verified findings against actual source code.

---

## TROUBLESHOOTING STORIES (For Interview)

### "Why doesn't checkout work?"
"The React checkout form requires all fields (firstName, lastName, address,
city, country, state, postalCode, phone, email, terms). In MONEYORDER mode,
the original code called stripe.createToken(card) where card is null (no
CardElement rendered). The fix skips Stripe and calls onPayment directly."

### "Why are there no orders in the admin?"
"The ng2-smart-table LocalDataSource.load() doesn't trigger Angular change
detection when loadingList is set to false before the load. The fix reorders
to load data first, then hide spinner, then detectChanges()."

### "Why does checkout fail with 'Unable to find Product with id X'?"
"The OrderServiceImpl.processOrder decrements inventory using
productService.getById(orderProduct.getId()). But orderProduct.getId() is
the auto-generated OrderProduct ID, not the Product ID. The fix uses
productService.getBySku(orderProduct.getSku(), store) instead."

### "Why is the React app slow?"
"The bundle is ~2MB with all dependencies. On this machine with Docker
containers running, page loads take 5-10 seconds. Production build
(npm run build) would help."

### "Why doesn't search work?"
"Product search requires Elasticsearch/OpenSearch which isn't configured.
The storefront has a client-side fallback to show all products when search
returns 0 results."

---

## REPO STRUCTURE

```
shopizer-research/          # Public GitHub repo (documentation)
├── SHOPIZER-BRIEFING.md    # Condensed 5-part briefing
├── docs/
│   ├── README.md           # Index
│   ├── 01-platform-overview.md
│   ├── 02-technical-architecture.md
│   ├── 03-end-to-end-flows.md
│   ├── 04-strengths-limitations.md
│   ├── 05-team-onboarding.md
│   ├── 06-docker-setup.md
│   ├── 07-bug-fixes.md
│   ├── 08-demo-readiness.md
│   ├── 09-secrets-and-credentials.md
│   ├── 10-ux-gaps-and-improvements.md
│   ├── 11-demo-polish.md
│   └── 12-master-prompt.md  # This file

shopizer/                   # Backend fork (Java Spring Boot)
shopizer-admin/             # Admin fork (Angular)
shopizer-shop-reactjs/      # Shop fork (React)
```

## GITHUB REPOS

- Research: https://github.com/kvginnovate/shopizer-research (public)
- Backend: https://github.com/kvginnovate/shopizer (fork)
- Admin: https://github.com/kvginnovate/shopizer-admin (fork)
- Shop: https://github.com/kvginnovate/shopizer-shop-reactjs (fork)
