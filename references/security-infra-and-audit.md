# Security Reference — Supply Chain, Docker/K8s & Audit Mode
Read ONLY when editing Dockerfiles, K8s manifests, CI pipelines, dependencies, or when the user requests a Security Audit / Pentest / Review.

## 1. Supply Chain & CI/CD
- Dependencies: verify exact package name/publisher before install (prevent typosquatting/hallucinated packages). Pin versions, commit lockfile, use `npm ci` / Gradle dependency verification. Audit changed deps (`npm audit`, OWASP dependency-check, Trivy).
- CI Hardening: `--ignore-scripts` where feasible; pin GitHub Actions/CI steps by full commit SHA; least-privilege workflow tokens; never use `pull_request_target` with untrusted code checkout; never pipe `curl | sh`; no secrets in build logs or artifacts.

## 2. Docker & Kubernetes Hardening
- Docker: multi-stage builds; minimal base image pinned by version/digest; non-root `USER`; no secrets in layers/`ARG`/`ENV` (use BuildKit secret mounts); `.dockerignore` excludes `.env`, `.git`, keys; no `--privileged` or `docker.sock` mount; read-only rootfs; drop all capabilities; `no-new-privileges`; define `HEALTHCHECK`.
- K8s `securityContext`: `runAsNonRoot: true`, `allowPrivilegeEscalation: false`, `readOnlyRootFilesystem: true`, drop `ALL` capabilities, `seccompProfile: RuntimeDefault`, Pod Security `restricted`, CPU/memory requests & limits.
- Cluster isolation: no `hostPath`/`hostNetwork`/`hostPID`; `automountServiceAccountToken: false` unless required; least-privilege RBAC (no `cluster-admin` or `*`); default-deny `NetworkPolicy` (DB/Redis reachable only from authorized pods); external/sealed Secrets encrypted at rest; Ingress TLS; internal services (`ClusterIP` only for DB/Redis/dashboards, never `LoadBalancer`/`NodePort`). Confirm cluster context before any state-changing command.

## 3. Logging & Detection
- Log security events (login ok/fail, authz deny, role/password/MFA changes, admin actions, rate-limit bursts) with user ID, IP, correlation ID. Strip CR/LF; never log passwords, tokens, keys, card numbers, or raw PII.

## 4. Audit Mode Procedure (When user asks for audit / security review / pentest / "está seguro?")
1. Map surface using file names + `rg` (never full file reads): routes, WS, queues, webhooks, cron, uploads, auth boundaries, data stores, outbound calls.
2. Trace untrusted input to dangerous sinks:
   `rg -n 'dangerouslySetInnerHTML|innerHTML|eval\(|new Function|queryRawUnsafe|executeRawUnsafe|child_process|shell: ?true|ObjectInputStream|readObject|enableDefaultTyping|yaml\.load|Math\.random|md5|sha1|rejectUnauthorized: ?false|origin: ?true|permitAll|csrf\(\)\.disable|nativeQuery|createStatement|jwt\.decode|localStorage|PRIVATE KEY' src/`
   Check every route/controller for missing auth guards or ownership (`tenantId`/`ownerId`) filters.
3. Prioritize: Authz/IDOR > Injection > Auth/Session > Secrets > SSRF/Deserialization/Upload > Crypto > Infra/Config.
4. Output format (no padding, concrete attack paths only, group duplicates):
   `[Critical|High|Med|Low] path:line | issue | 1-line attack path | minimal fix diff` (Tag: `confirmed` / `likely` / `needs-check`).
5. End with 1 line of coverage (`Checked: X | Not checked: Y`). Never say "system is secure"; say "no findings in checked scope". Fix only after user approval (Critical first).
