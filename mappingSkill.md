# Mapping Skill — Syndetis_Project_Web Router
Goal: Load the minimum instructions required for each task in `Syndetis_Project_Web`. Never load all skills or scan all of `docs/`.
Base contract: `docs/skills/lowTokensSkill.md` rules apply to all tasks.

## 1. Procedure
1. Identify intent, target area (`frontend/`, `gateway/`, `services/`, `libs/`, `docker/`, `k8s/`), and action type (analyze, test, or edit).
2. Select 1 primary skill from the table below. Load a 2nd skill ONLY if the task genuinely requires two specialties.
3. Read only the selected skill(s) and strictly necessary code/spec files.
4. Bound affected files before editing; preserve unrelated code.
5. Run targeted checks on changed files only; report out-of-scope issues in 1 line without fixing.

## 2. Skill Selection Table
| Task Intent | Skill to Read | Optional Reference (Read only if needed) | Scope Boundary |
| :--- | :--- | :--- | :--- |
| UI, components, styling, responsive layout, a11y, UX | `docs/skills/frontendSkill.md` | `docs/skills/references/frontend-vibe-ban-and-inspiration.md` | `frontend/` & essential shared types |
| 3D scenes, cameras, WebGL, shaders, GSAP/scroll timelines | `docs/skills/motion3D.md` | — | Requested 3D/animated elements only (standard CSS transitions stay in `frontendSkill.md`) |
| Auth, sessions, credentials, input validation, API/infra security, audits | `docs/skills/securitySkill.md` | `docs/skills/references/security-infra-and-audit.md` | Specific security component/flow |
| Run, write, review or fix tests; bug repro; load/regression checks | `docs/skills/testingSkill.md` | — | Requested test scope; no functional/visual prod changes without permission |
| Reduce tokens, context bloat, or execution cost | `docs/skills/lowTokensSkill.md` | — | Workflow optimization only; do not alter product behavior |

## 3. Tasks Without a Dedicated Skill (Direct Code + Spec Doc)
- Backend & business rules: inspect only relevant files in `services/`, `gateway/`, `libs/`. Do NOT load `frontendSkill.md` or `testingSkill.md` just because backend code will be edited.
- Asaas, plans & subscriptions: `docs/assinaturas-asaas.md` and `docs/planos-e-assinaturas.md`.
- External integrations: `docs/api-provider.md` + corresponding service code.
- Gateway & rate limits: `docs/gateway-rate-limits.md`.
- Products & variations: `docs/product-variations.md`.
- Docker, K8s & deployment: `docker/`, `k8s/`, `docs/kubernetes-local.md`.
- Homologation: relevant docs in `docs/`; load `testingSkill.md` only if actively executing/writing tests.

## 4. Strict Scope Isolation
- Test != Modify: "test checkout" = execute and report tests; never change prod code, colors, or layouts unless asked to fix.
- Fix != Redesign: "fix login validation" = change only required logic; never touch unrelated styles or components.
- UI != Backend: never alter APIs, contracts, migrations, or infra for frontend convenience (explain if blocked).
- Unrelated broken test: report as a separate 1-line issue; never auto-fix.

## 5. Legitimate Combinations
- "Test signup form": `testingSkill.md` (read `frontend/` target files to identify test targets; do not modify UI).
- "Fix XSS in form": `securitySkill.md` (+ `frontendSkill.md` only if component structure changes).
- "Optimize 3D animation and benchmark": `motion3D.md` + `testingSkill.md`.
- "Create page and verify it works": `frontendSkill.md` (basic check is built-in; no need to load `testingSkill.md`).
