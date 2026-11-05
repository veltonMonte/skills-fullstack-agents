# Security Skill (AppSec & Core Controls)
Read before touching: auth, sessions, user input, DB/queries, crypto, uploads, network calls, secrets. Extends `lowTokensSkill.md`. Defensive only.
For Docker/K8s/CI supply-chain hardening or full Security Audit / Pentest mode: also read `docs/skills/references/security-infra-and-audit.md`.

## Mindset
- Secure by default, deny by default, fail closed, least privilege. Server is the only trust boundary (client input, headers, cookies, files, queues, webhooks, 3rd-party APIs, DB/Redis content, LLM output = untrusted).
- Insecure user request: warn in 1 line and implement the secure way (or ask). Never silently weaken.
- Sensitive features (auth, payments, uploads, multi-tenant, admin): threat model in max 3 lines (assets, attacker, entry points).
- Use vetted libs only (Nest guards + Passport, Spring Security, `jose`, `argon2`, Tink/Bouncy Castle). Out-of-scope vulnerability found: report in 1 line (`file:line`, issue); do not fix without asking.

## 1. Injection & Input Handling
- Allowlist schema validation at boundary (type, length, range, format, enum); reject invalid input (never "sanitize and continue"). Nest: global `ValidationPipe({ whitelist: true, forbidNonWhitelisted: true, transform: true })`. JVM: `@Valid`. React/BFF: `zod`.
- SQL: bound parameters only (`PreparedStatement`, JPA named params, Prisma `$queryRaw` tagged template; never `$queryRawUnsafe` or string concat). Dynamic table/column/`ORDER BY`: fixed allowlist map. Escape `LIKE` wildcards.
- NoSQL/Redis: expect scalars, reject objects/operators (`$ne`, `$gt`). Never build Redis commands/Lua/keys from raw input.
- OS commands: no shell (`execFile`/`spawn` with args array, `ProcessBuilder` list; never `shell: true` or `sh -c`).
- Paths & Archives: never join user input into paths; resolve + verify prefix or map ID -> file. Check zip slip, size, compression ratio.
- SSRF: allowlist scheme + host; resolve DNS and block private/loopback/link-local/metadata (`169.254.169.254`) ranges; disable/re-check redirects.
- Deserialization & XML: never `ObjectInputStream` or Jackson default typing on untrusted data; safe YAML loaders; disable XML DTD/XXE.
- Mass assignment & Prototype pollution: strict DTO allowlist (never bind body to entity; `role`, `ownerId`, `price`, `isAdmin` never from client). Block `__proto__`/`constructor`/`prototype`.
- ReDoS, CRLF, CSV, SSTI: no nested regex quantifiers (cap input length); strip CR/LF from headers/logs/emails; prefix CSV cells starting with `= + - @` with `'`; never render user input as template.
- Uploads: size limit, check magic bytes (not extension/Content-Type), random server filename outside webroot, no exec perms, re-encode images, separate domain/bucket, `Content-Disposition: attachment` for non-images.

## 2. Frontend & XSS
- Dangerous sinks: `dangerouslySetInnerHTML` (require strict `DOMPurify`), `innerHTML`, `eval`/`new Function`, user URLs in `href`/`src`/`action` (parse with `new URL()`, allow only `http`/`https`/`mailto`/`tel`), markdown/SVG uploads, SSR JSON in `<script>` (escape `<`).
- Hardening: strict CSP (nonce/hash, `object-src 'none'`, `base-uri 'none'`, `frame-ancestors`), `rel="noopener noreferrer"` on `target="_blank"`, exact `event.origin` on `postMessage`, sandboxed iframes, SRI on external scripts. No secrets or source maps in prod client bundle.

## 3. Authentication & Browser Storage
- Passwords: Argon2id (`m=19MiB, t=2, p=1` min) or bcrypt (`cost >= 12`, `<72 bytes`) / scrypt. Never MD5/SHA/plain.
- Endpoints (login/reset/OTP/signup): rate limit per account + IP with backoff; constant-time responses to prevent user enumeration (`timingSafeEqual` / `MessageDigest.isEqual`).
- Reset/magic tokens: CSPRNG `>=128 bits`, store hash only, single-use, short TTL; invalidate sessions on password change. MFA/re-auth for sensitive actions.
- Sessions & Cookies: rotate ID on login; invalidate server-side on logout; idle + absolute timeout. Cookie holds opaque CSPRNG ID (`>=128 bits`) or signed/AEAD payload (never raw JWT with PII/roles/prices). Flags: `HttpOnly; Secure; SameSite=Lax|Strict`, `__Host-` prefix (`Path=/`, no `Domain`), `<4KB`. Behind TLS proxy: enable `trust proxy` (Nest) / `forward-headers-strategy` (Spring). Expire with `Max-Age=0` on logout.
- JWT & OAuth: algorithm allowlist (reject `none`, block HS/RS confusion), verify signature/`exp`/`nbf`/`iss`/`aud`, access `<=15m` + rotating refresh with reuse detection, keys via KMS/env with `kid`. OAuth: Auth Code + PKCE, exact `redirect_uri`, validate `state` + `nonce`.
- Storage & WS: zero tokens/PII/secrets in `localStorage`/`sessionStorage`/`IndexedDB`/Service Worker cache/URLs/logs (redact `Cookie`, `Set-Cookie`, `Authorization`). Validate `Origin` and authenticate WebSocket handshake.

## 4. Authorization (IDOR / BOLA)
- Default deny: Nest global `APP_GUARD` + `@Public()`; Spring `anyRequest().authenticated()` last (covers REST, GraphQL, WS, gRPC, queues, cron, actuator, swagger).
- Object-level: scope every query by session owner/tenant (`WHERE id = :id AND tenant_id = :sessionTenant`). UUIDs are not authz.
- Function & Field level: role/permission check per handler (`@PreAuthorize`); return response DTOs with allowlist (`ClassSerializerInterceptor` + `@Exclude`), never raw entities. GraphQL: depth/complexity limits, introspection off in prod.
- Mandatory test: every auth/authz change requires 1 negative test (User A cannot access User B's resource; anon -> 401; wrong role -> 403).

## 5. API Hardening, Crypto & Concurrency
- API: exact CORS origin allowlist (never `*` with credentials); CSRF token or Origin/`Sec-Fetch-Site` check on cookie-authenticated state changes; Helmet/Spring security headers (`HSTS`, `nosniff`, `Cache-Control: no-store`); body/depth/page/rate limits (Redis throttler); generic client errors with correlation ID (no stack traces/SQL); webhook HMAC + timestamp replay window + idempotency key.
- Crypto: AEAD only (`AES-256-GCM` or `ChaCha20-Poly1305` with unique CSPRNG nonce); `HMAC-SHA-256+` / `Ed25519`; `TLS 1.2+` (prefer 1.3, never `rejectUnauthorized: false`); keys in KMS/Vault; mask/encrypt PII; leaked secret = rotate immediately.
- Logic & Concurrency: server computes price/discount/role/status using integer cents or `BigDecimal` (never float; reject negative/NaN/overflow). Prevent TOCTOU/double-spend via DB transaction + `FOR UPDATE` row lock or atomic conditional `UPDATE` + idempotency key. Enforce server-side state machine transitions. Fail closed using strict `===`.
- Redis & LLM: Redis private network + ACL/pass + TLS, disable `FLUSHALL`/`CONFIG`/`KEYS`/`EVAL`. Treat LLM output as untrusted (never eval/HTML/SQL/shell); enforce per-user rate/cost caps.
Priority: Security > user's request > everything else.
