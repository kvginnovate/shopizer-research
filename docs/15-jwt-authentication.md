# JWT Authentication in Shopizer

How token-based authentication works across the backend (Spring Boot) and the
two frontends (React storefront, Angular admin). Sourced from
`shopizer` 3.2.7.

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

`JWTUser` wraps `UserDetails` (id, username, firstname, lastname, email,
authorities, enabled, lastPasswordResetDate).

## Authentication managers / providers / services

| Audience | Manager bean | Provider | Service (UserDetailsService) |
|----------|--------------|----------|------------------------------|
| Admin | `jwtAdminAuthenticationManager` | `JWTAdminAuthenticationProvider` | `JWTAdminServicesImpl` |
| Customer | `jwtCustomerAuthenticationManager` | `JWTCustomerAuthenticationProvider` | `JWTCustomerServicesImpl` (injected as `jwtCustomerDetailsService`) |

Providers use `PasswordEncoder` (BCrypt) to verify credentials.

## Route protection (security config)

`MultipleEntryPointsSecurityConfig` (`@EnableWebSecurity`, ordered adapters):

| Order | Adapter | Matcher | Rules |
|-------|---------|---------|-------|
| 1 | CustomerConfigurationAdapter | `/shop/**` | most public; `/shop/customer/**` requires `AUTH_CUSTOMER` |
| 2 | ServicesApiConfigurationAdapter | `/services/**` | `/services/public/**` permitAll; `/services/private/**` requires `AUTH` |
| 5 | UserApiConfigurationAdapter | `/api/v*/private/**` | `/private/login`, `/private/refresh`, OPTIONS permitAll; rest requires `AUTH` |
| 6 | CustomeApiConfigurationAdapter | `/api/v*/auth/**` | `/auth/refresh`, `/auth/login`, `/auth/register`, OPTIONS permitAll; rest requires `AUTH_CUSTOMER` |

Both API adapters `addFilterAfter(authenticationTokenFilter, BasicAuthenticationFilter.class)`.

## Per-request enforcement

`AuthenticationTokenFilter` (`OncePerRequestFilter`):

1. Sets CORS headers (`Access-Control-Allow-Methods/Origin/Headers/Credentials`).
2. Captures client IP into `UserContext`.
3. If URL contains `/api/v1/auth` → customer auth manager validates `Bearer` token.
4. If URL contains `/api/v1/private` or `/api/v2/private` → admin auth manager validates `Bearer` token.
5. `JWTTokenUtil.validateToken`: username matches, token not expired, token issued after last password reset.
6. `chain.doFilter`; `postFilter` closes `UserContext`.

Note: a `FACEBOOK_TOKEN ("FB ")` branch exists but is commented out (unused).

## Validation rules

`JWTTokenUtil.validateToken(token, userDetails)`:
- `username.equals(user.getUsername())`
- `!isTokenExpired(token)` (exp before now)
- `!isCreatedBeforeLastPasswordReset(created, user.getLastPasswordResetDate())`

So changing a password (which updates `lastPasswordResetDate`) invalidates
older tokens.

## Refresh

- Admin: `GET /api/v1/auth/refresh`
- Customer: `GET /api/v1/auth/customer/refresh`

Re-issues a token when `canTokenBeRefreshed` holds (not created before last
password reset, and either not expired or audience is mobile/tablet). A
200 s grace window exists (`GRACE_PERIOD`, `*WithGrace` variants). Note:
`canTokenBeRefreshedWithGrace` currently returns `true` unconditionally.

## Frontends

- **React** (`src/util/webService.js`): reads `token` from local storage, sets
  `config.headers.common['Authorization'] = 'Bearer ' + token`.
  Login/register store it via `setLocalData('token', response.token)`;
  logout clears it (`setLocalData('token', '')`).
- **Angular** (`auth.interceptor.ts`): `TokenService` holds the token; interceptor
  clones every request adding `Authorization: Bearer <token>`; on 401 it calls the
  refresh endpoint and retries with the new token. `auth.guard.ts` gates routes
  on `tokenService.getToken()`.

## Security notes (demo only)

- Secret is the literal string `aSecret` (weak, hardcoded) — replace for production.
- Refresh grace logic has a quirk (`canTokenBeRefreshedWithGrace` returns `true`).
- No refresh-token rotation; single 7-day token.
- Tokens are stateless; revocation relies on password reset (`lastPasswordResetDate`).
