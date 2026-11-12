# 6. Docker Setup — Running the Whole App

How the full Shopizer stack was set up locally with Docker, step by step,
so anyone can reproduce it. Companion to the architecture (doc 02) and
onboarding (doc 05) guides.

## Why Docker (and not native builds)

The machine had Java 21 (Corretto), Node 24, and no Maven or MySQL —
all wrong vintages for these codebases (backend wants Java 11/17,
admin wants Node 12, shop wants Node 16). The official prebuilt images
embed the correct runtimes, and the backend image ships with an embedded
H2 database, so **no MySQL and no builds are needed**.

## Prerequisites

- Docker client + Docker Desktop installed (daemon must be running —
  `docker ps` should list containers, not an error).
- ~2 GB free disk for images (backend ~1 GB, shop ~150 MB, admin ~80 MB).
- Free ports: `8080` (backend), `8081` (admin), `8082` (shop).

## Step 1 — Start the Docker daemon (if needed)

If `docker ps` reports it cannot connect, launch Docker Desktop and wait
until `docker ps` succeeds (empty table is fine).

## Step 2 — Pull the official images

```bash
docker pull shopizerecomm/shopizer:latest               # Spring Boot backend
docker pull shopizerecomm/shopizer-admin:latest         # Angular admin (nginx)
docker pull shopizerecomm/shopizer-shop-reactjs:latest  # React shop (nginx)
```

## Step 3 — Start all three containers

```bash
docker run -d --name shopizer-backend -p 8080:8080 \
  shopizerecomm/shopizer:latest

docker run -d --name shopizer-admin \
  -e "APP_BASE_URL=http://localhost:8080/api" \
  -p 8081:80 \
  shopizerecomm/shopizer-admin:latest

docker run -d --name shopizer-shop \
  -e "APP_MERCHANT=DEFAULT" \
  -e "APP_BASE_URL=http://localhost:8080" \
  -p 8082:80 \
  shopizerecomm/shopizer-shop-reactjs:latest
```

What each env var does:

| Container | Variable | Meaning |
|-----------|----------|---------|
| `shopizer-admin` | `APP_BASE_URL=http://localhost:8080/api` | Backend REST base the admin UI calls (baked into nginx `env.js` at boot) |
| `shopizer-shop` | `APP_MERCHANT=DEFAULT` | Which store's catalogue/pricing the shop displays |
| `shopizer-shop` | `APP_BASE_URL=http://localhost:8080` | Backend REST base the shop calls (from `public/env-config.js` semantics) |

These URLs must be **browser-reachable** — the React/Angular apps run in the
visitor's browser, so `localhost` here means "the machine viewing the page",
which is why ports are published (`-p`) rather than using container networking.

## Step 4 — Wait for the backend to boot

Spring Boot takes ~30–90 s on first start (H2 schema creation + demo data).
Poll until it answers:

```bash
curl http://localhost:8080/api/v1/store/DEFAULT
```

A JSON store object (`"code":"DEFAULT","name":"Default store",…`) means ready.
`docker logs shopizer-backend --tail 5` should show
`Started ShopApplication in … seconds`.

## Step 5 — Verify the stack

```bash
# Admin login returns a JWT
curl -X POST http://localhost:8080/api/v1/private/login \
  -H 'Content-Type: application/json' \
  -d '{"username":"admin@shopizer.com","password":"password"}'

# Catalogue is populated
curl 'http://localhost:8080/api/v1/products?store=DEFAULT&lang=en&page=0&count=3'

# Cart flow works (use a real SKU from the products call, e.g. "table1")
curl -X POST http://localhost:8080/api/v1/cart \
  -H 'Content-Type: application/json' \
  -d '{"product":"table1","quantity":1}'
# → returns a cart "code"; read it back:
curl http://localhost:8080/api/v1/cart/<code>   # shows subtotal/total
```

Then in the browser:

| URL | What to try |
|-----|-------------|
| http://localhost:8082 | Shop: browse catalogue, add to cart (guest cart works) |
| http://localhost:8081 | Admin: log in `admin@shopizer.com` / `password`, open Orders/Catalogue |
| http://localhost:8080/swagger-ui.html | Backend: execute store/product endpoints live |

## Managing the stack

```bash
docker ps                                            # status of the 3 containers
docker logs -f shopizer-backend                      # backend logs
docker stop shopizer-backend shopizer-admin shopizer-shop     # stop
docker start shopizer-backend shopizer-admin shopizer-shop   # start again
docker rm -f shopizer-backend shopizer-admin shopizer-shop   # remove (data resets!)
```

## Things to know

- **Database is embedded H2** (file inside the backend container, per its
  `profiles/docker/database.properties`). No MySQL setup was needed — but
  removing the container **wipes all data**. For persistent/demo-stable data,
  run MySQL 8 separately (`CREATE DATABASE SALESMANAGER`, user `shopizer`)
  and point the backend at it via the `profiles/mysql` datasource properties.
- **First boot is slow, restarts are fast** (H2 file already initialised).
- **Admin "401 / logged out" in the browser** almost always means the admin
  container was started with a wrong `APP_BASE_URL` — recreate it with the
  exact value from Step 3.
- **Port conflicts:** if 8080–8082 are taken, change only the left side of
  `-p` (e.g. `-p 9090:80`) — but then `APP_BASE_URL` must use the new host
  port too.
- Native builds remain an option (see backend README: `mvnw clean install`
  + `spring-boot:run`) but require installing Java 11/17, Maven, MySQL, and
  Node 12/16 — considerably more setup than the three commands above.
