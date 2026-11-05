# Testing Skill
Read when writing/changing tests, asked for coverage/TDD, or reproducing a bug. Extends `lowTokensSkill.md`.
Goal: deterministic tests that fail when behavior breaks and pass when it doesn't.

## 1. Detect First (Syndetis_Project_Web Stack)
1. Check `package.json` / `build.gradle(.kts)`, test config, and ONE existing test file to match style, folder layout, and helpers.
2. Use existing runner and libs only (never mix or add runners without asking).
   - JS/TS: Jest (NestJS: `npx jest <file> --silent --noStackTrace`), Vitest (React: `npx vitest run <file> --reporter=dot`), Testing Library + `user-event`, Supertest, MSW, Playwright/Cypress (E2E).
   - JVM (Kotlin/Java): JUnit 5 (`@ParameterizedTest`) + `kotlin.test`/AssertJ/Kotest, MockK/Mockito (`./gradlew -q test --tests "<Class>"`), Spring slices (`@DataJpaTest`, `@WebMvcTest`), Testcontainers (`@ServiceConnection`, 1 shared container per suite).

## 2. Before Writing (Max 6 lines internal plan)
1. `rg` existing tests, factories, and fixtures to reuse.
2. Select cases protecting real behavior: happy path, boundaries (`0`, `1`, max, empty, `null`), invalid input, dependency failure, authz (owner vs other, anon `401`, wrong role `403`), concurrency/idempotency.
3. Pick lowest effective level:
   - Unit (no I/O): pure domain logic, mappers, validators.
   - Integration (real engine via Testcontainers / docker-compose): DB queries, transactions, locks, Redis TTL/atomicity. Never fake Postgres/Redis with SQLite/H2/in-memory maps for query or lock tests.
   - API / Component: HTTP pipeline with real guards/pipes (`ValidationPipe`), or UI component queried by `role`/`label`/`text`.
   - E2E: critical journeys only when requested or configured.

## 3. Quality Rules (Zero AI Test Slop)
- Derive expected values as hardcoded literals from requirements; never copy or recompute using production implementation logic.
- Assert observable outcomes (returned value, state persisted, HTTP status/body, DOM rendered, event emitted), never private methods, internal state, or mock call order.
- Mock only external boundaries you don't own (3rd-party HTTP, clock, payment gateway). `>3` mocks = wrong test level.
- 1 behavior per test, Arrange-Act-Assert, named `<unit> <does X> when <condition>`. No `if`/`for`/`try` inside test bodies (use `it.each` / `@ParameterizedTest`).
- Strict determinism: fake clock/timers, seeded random, condition waits (never fixed `sleep`/`setTimeout`), isolated per-test data, full teardown (close servers/DB pools).
- Never leave `.only`, `.skip`, `@Disabled`, commented tests, or console noise. Never weaken assertions to get green.
- Test reveals a real production bug: report in 1 line (`file:line`, expected vs actual). Fix prod code only if in scope; else ask.

## 4. Bugs, TDD & Critical Domain Matrices
- Bug fix: write failing regression test first -> confirm failure reason -> fix code -> report `Red-check: failed before, passes after`.
- Critical logic (auth, money, permissions, concurrency): perform a red-check (temporarily break 1 condition in prod code, verify test fails, revert).
- Domain checklists (apply when relevant):
  - Money: integer cents / `BigDecimal`; test `0`, negative, overflow, rounding.
  - Auth/Authz: User A cannot read/write User B's resource (IDOR), anon `401`, wrong role `403`, mass-assignment ignores `role`/`ownerId`, no leaked secrets/stack traces.
  - Concurrency (double-spend, coupons, stock, locks): fire `N` parallel calls against real DB/Redis and assert exact success count.
  - Webhooks: valid/invalid HMAC, tampered body, expired timestamp, replay idempotency (same event ID twice = processed once).
  - Uploads: size limit, magic-byte mismatch, path traversal filenames.

## 5. Run & Output
- Run changed test file only with quiet flags. Max 2 fix rounds on failure, then stop and report root cause.
- If Docker/DB is unavailable: stop and report `Not run: <dependency> unavailable. Run: <cmd>` (never swap in an in-memory fake to force a pass).
- Output test file(s) with paths, followed by max 3 lines: cases covered, assumptions/untested scope, and `Validated: <cmd -> N passed>`.
Priority: Security > user's request > test validity (a test that can't fail is worse than none) > speed > brevity.
