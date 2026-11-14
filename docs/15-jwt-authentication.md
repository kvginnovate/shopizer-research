# JWT Authentication in Shopizer

How token-based authentication works across the backend (Spring Boot) and the
two frontends (React storefront, Angular admin).

## Configuration

`sm-core/src/main/resources/authentication.properties`:

```properties
authToken.header=Authorization
jwt.header=Authorization
jwt.secret=aSecret
jwt.expiration=604800
```

- Signing algorithm: **HS512** (symmetric, shared secret).
- Token lifetime: **604800 s = 7 days**.
- Header carrying the token: `Authorization: Bearer <token>`.

## Token shape

`JWTTokenUtil` (`sm-shop/.../store/security/JWTTokenUtil.java`) builds a JJWT
with claims:

| Claim | Value |
|-------|-------|
| `sub` | username |
| `aud` | `api` |
| `iat` | issued-at (now) |
| `exp` | issued-at + 7 days |

Signed with `SignatureAlgorithm.HS512` using `jwt.secret`.

## Login endpoints

| Audience | Endpoint | Controller |
|----------|----------|------------|
| Admin (users) | `POST /api/v1/private/login` | `AuthenticateUserApi` |
| Customer | `POST /api/v1/customer/login` | `AuthenticateCustomerApi` |
| Customer (register) | `POST /api/v1/customer/register` | `AuthenticateCustomerApi` |

Flow: `UsernamePasswordAuthenticationToken` → `JWT*AuthenticationManager`
(BCrypt password check via `JWT*AuthenticationProvider`) → reload `JWTUser` →
`jwtTokenUtil.generateToken(userDetails)` → returns `AuthenticationResponse { id, token }`.

`JWTUser` wraps `UserDetails` (id, username, authorities, lastPasswordResetDate).

## Per-request enforcement

`AuthenticationTokenFilter` (`OncePerRequestFilter`):

1. Sets CORS headers (`Access-Control-Allow-*`).
2. If URL contains `/api/v1/auth` → customer auth manager validates `Bearer` token.
3. If URL contains `/api/v1/private` or `/api/v2/private` → admin auth manager validates `Bearer` token.
4. `JWTTokenUtil.validateToken`: username matches, token not expired, token issued after last password reset.
5. `chain.doFilter`.

## Refresh

- Admin: `GET /api/v1/auth/refresh`
- Customer: `GET /api/v1/auth/customer/refresh`

Re-issues a token when `canTokenBeRefreshed` holds (with a 200 s grace window
via `GRACE_PERIOD`).

## Frontends

- **React** (`src/util/webService.js`): reads `token` from local storage, sets
  `config.headers.common['Authorization'] = 'Bearer ' + token`.
  Login/register store it via `setLocalData('token', response.token)`.
- **Angular** (`auth.interceptor.ts`): `TokenService` holds the token; interceptor
  clones every request adding `Authorization: Bearer <token>`; on 401 it calls the
  refresh endpoint and retries with the new token.

## Security notes (demo only)

- Secret is the literal string `aSecret` (weak, hardcoded) — replace for production.
- `canTokenBeRefreshedWithGrace` returns `true` unconditionally (logic quirk).
- No refresh-token rotation; single short-lived-ish 7-day token.
