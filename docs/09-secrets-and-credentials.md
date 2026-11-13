# Demo Secrets & Credentials

> **Private repo only.** This document contains all credentials, passwords,
> connection strings, and configuration used for the Shopizer interview demo.
> Do not share outside the team.

## Application Credentials

### Admin Console (shopizer-admin)
| Field | Value |
|-------|-------|
| URL | http://localhost:8081 |
| Username | admin@shopizer.com |
| Password | password |
| Backend API | http://localhost:8080/api |

### Customer Storefront (shopizer-shop-reactjs)
| Field | Value |
|-------|-------|
| URL | http://localhost:8082 |
| Demo account email | demouser@shopizer.com |
| Demo account password | demo1234 |
| Guest checkout | Supported (no login required) |

### Backend API (Swagger)
| Field | Value |
|-------|-------|
| Swagger UI | http://localhost:8080/swagger-ui.html |
| Admin login endpoint | POST /api/v1/private/login |
| Customer login endpoint | POST /api/v1/customer/login |

## Database (MySQL 8.0.42)

| Field | Value |
|-------|-------|
| Host | localhost (Docker: shopizer-mysql) |
| Port | 3306 |
| Database | SALESMANAGER |
| Root password | rootpw123 |
| App user | shopizer |
| App password | shopizerpw123 |
| JDBC URL | jdbc:mysql://shopizer-mysql:3306/SALESMANAGER?autoReconnect=true&serverTimeZone=UTC&useUnicode=true&characterEncoding=UTF-8 |

## Docker Configuration

### Network
- Network name: `shopizer-net`

### Container Names & Ports
| Container | Image | Port | Notes |
|-----------|-------|------|-------|
| shopizer-backend | shopizerecomm/shopizer (patched jar) | 8080 | Spring Boot, patched for MySQL |
| shopizer-mysql | mysql:8.0.42 | 3306 | SALESMANAGER database |
| shopizer-admin | shopizerecomm/shopizer-admin (bundle patched) | 8081 | Angular + Nebular |
| shopizer-shop | shopizer-shop-fixed (rebuilt) | 8082 | React + Redux |

### Environment Variables

**Admin container:**
```
APP_BASE_URL=http://localhost:8080/api
```

**Shop container:**
```
APP_MERCHANT=DEFAULT
APP_BASE_URL=http://localhost:8080
```

**MySQL container:**
```
MYSQL_ROOT_PASSWORD=rootpw123
MYSQL_DATABASE=SALESMANAGER
MYSQL_USER=shopizer
MYSQL_PASSWORD=shopizerpw123
```

## Backend Configuration (db-mysql.properties)

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

## Payment Configuration

| Field | Value |
|-------|-------|
| Payment type | MONEYORDER (mock, no external credentials) |
| Payment module | moneyorder (configured and active) |
| Integration keys | address: "123 Demo Street, Montreal, QC" |
| Stripe test key | pk_test_TYooMQauvdEDq54NiTphI7jx (not used in MONEYORDER mode) |

## Files with Secrets (Not Committed)

| File | Location | Contents |
|------|----------|----------|
| db-mysql.properties | D:/Projects/shopizer/db-mysql.properties | MySQL connection credentials |
| shopizer-patched.jar | D:/Projects/shopizer/shopizer/shopizer-patched.jar | Backend jar with MySQL config baked in |
| env-config.js | Inside shop container | Frontend env config |
| .env | shopizer-shop-reactjs/.env | Frontend env vars |

## GitHub

| Field | Value |
|-------|-------|
| Account | kvginnovate |
| Research repo | https://github.com/kvginnovate/shopizer-research (public) |
| Backend fork | https://github.com/kvginnovate/shopizer |
| Admin fork | https://github.com/kvginnovate/shopizer-admin |
| Shop fork | https://github.com/kvginnovate/shopizer-shop-reactjs |

## Quick Start (All Commands)

```bash
# Start everything
docker start shopizer-mysql shopizer-backend shopizer-admin shopizer-shop

# Stop everything
docker stop shopizer-backend shopizer-admin shopizer-shop shopizer-mysql

# Re-apply MySQL patch (if backend reverted to H2)
docker cp shopizer/shopizer-patched.jar shopizer-backend:/opt/app/shopizer.jar
docker restart shopizer-backend

# Re-apply admin bundle patch (if container recreated)
docker cp C:/Users/Chokkar/AppData/Local/Temp/admin-dist/18.68c3e32d5f6ba64567a4.js \
  shopizer-admin:/usr/share/nginx/html/18.68c3e32d5f6ba64567a4.js
```

## Notes

- The backend Docker image (shopizerecomm/shopizer:latest) defaults to H2 embedded DB.
  The jar must be patched with MySQL config for production use.
- The admin container serves pre-built Angular files. Bundle patches are
  lost when the container is recreated.
- The shop container was rebuilt from source with all fixes.
- All passwords are demo-only and should be changed for production.
