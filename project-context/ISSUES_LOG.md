# STOCKFORGE — ISSUES LOG

> Running log of every issue faced while building StockForge, one entry per issue,
> updated at the END OF EVERY DAY (mandatory, per `START_OF_DAY.md` Phase 3).
> Purpose: we never repeat the same mistake, and future AI sessions learn the pitfalls
> before touching code.
> **Format: every bug is a mini case study** (SRE-style, PROJECT_CONTEXT §25 habit 7):
> Symptom → Detection → Cause → Fix → Prevention → Production relevance.

---

## Issue entry template

```
### Day N — <short issue title>

- **Symptom:** what happened / what we saw (error, message, behavior)
- **Detection:** how we noticed it (test, log, curl, dashboard, gut)
- **Cause:** why it happened (root cause — "because", not "we did")
- **Fix:** what we changed to resolve it
- **Prevention:** how we stop it recurring (test, guard, config, process)
- **Production relevance:** what a real trading platform (Zerodha/Groww-grade) does for
  this same problem, or why this pitfall matters in production
```

---

## Day 7 — `stockforge-auth` (2026-08-11)

### Day 7 — Protected endpoints answered 403 to anonymous requests instead of 401

- **Symptom:** the new Day 7 tests failed: `/api/auth/me` and `/api/auth/admin/ping` with a
  missing, invalid, or expired token all returned **403** (expected 401).
- **Detection:** the test suite (`meRejectsMissingToken`, `meRejectsInvalidToken`,
  `meRejectsExpiredToken`, `adminEndpointRejectsAnonymous`) failed with "Status expected:<401>
  but was:<403>".
- **Cause:** Spring Security's 401 answer does not come from the filter chain we wrote — it comes
  from an **`AuthenticationEntryPoint`**, and the default one is installed by enabling
  httpBasic/formLogin. Once BOTH are disabled, no default entry point exists, and Spring falls
  back to `Http403ForbiddenEntryPoint`. Result: "not authenticated" was reported as "forbidden".
- **Fix:** configured an explicit entry point in `SecurityConfig`
  (`exceptionHandling(... authenticationEntryPoint(...)` answering
  `HttpServletResponse.SC_UNAUTHORIZED`). Tests went green; curl confirmed 401 live.
- **Prevention:** a test per "unauthenticated → 401" and "authenticated-but-denied → 403" case;
  remember the rule: disabling form/basic login removes the default entry point, so any stateless
  API must define its own 401 handler.
- **Production relevance:** 401 vs 403 is a real API contract (RFC 9110 semantics). Clients
  (and SLOs, and auth middleware) treat them differently — wrong statuses break token-refresh
  flows, logs, and security dashboards. Trading APIs must never leak "the resource exists but
  you're not allowed" to anonymous callers.

### Day 7 — Stale Day 6 auth JVM was still squatting port 8080 (recurrence)

- **Symptom:** fresh `spring-boot:run` failed: "Port 8080 was already in use".
- **Detection:** `Get-NetTCPConnection -LocalPort 8080 -State Listen` → a leftover `java`
  (old stockforge-auth) from the Day 6 session; it even answered `/api/auth/register`.
- **Cause:** the Day 6 server was never stopped; it kept the port after we moved on.
- **Fix:** stopped the stale process (verified it was ours via its own endpoints first),
  restarted the new build cleanly.
- **Prevention:** at the end of every verification, either stop the server or note in state
  that it stays up; before starting a run, check the port and identify the owner before
  killing. This is the SECOND time this exact recurrence happened (Day 6 logged it too).
- **Production relevance:** real deployments solve this with process supervision (systemd),
  health/readiness probes, and fixed ports per service — port collisions between microservices
  are a classic dev-to-prod drift signal.

### Day 7 — PowerShell→curl inline JSON mangling (recurrence)

- **Symptom:** live curl of `POST /api/auth/register` returned 400 Bad Request with a
  perfectly valid-looking inline body.
- **Detection:** the Day 0-6 lesson applied immediately: the inline JSON quotes were stripped
  by PowerShell 5.1 native-argument handling.
- **Cause:** same root cause as Day 6 — PowerShell mangles embedded quotes when passing
  arguments to native `curl.exe`.
- **Fix:** used file-based bodies (`curl.exe --data-binary "@body.json"`) — worked first try.
- **Prevention:** file bodies only (the committed `test-auth.ps1` pattern); never inline JSON.
  Note the pattern now recurred twice — bake "file bodies always" into the run/test playbook.
- **Production relevance:** API testing from shell scripts is fragile; real teams test from
  fixtures/frameworks (Postman collections, REST Assured) with a CI pipeline, so quoting
  quirks never reach a shared script.

---

## Day 6 — `stockforge-auth` (2026-08-08)

### Day 6 — Every auth request returned 403: two compounding causes (mangled curl body + secured /error)

- **Symptom:** manual testing reported `POST /api/auth/register` → **403** (expected 201). The
  auth service seemed to reject everything; `GET /api/auth/register` was also 403 and
  `/actuator/health` was 200.
- **Detection:** reproduced against a fresh instance on port 8081; the server log showed
  `HttpMessageNotReadableException: JSON parse error: Unexpected character ('e' ...)` —
  meaning the request REACHED the controller but the body was `{email:...}` (first `"` missing).
- **Cause (two layers):**
  1. **Tooling:** from PowerShell, `curl.exe -d '{"email":"..."}'` (and even `-d "{\"email\":...}"`)
     mangles the embedded quotes, so the server receives `{email:"..."}` — an unreadable JSON
     body, NOT a clean request. This is a PowerShell 5.1 native-argument quoting quirk, not a
     server bug.
  2. **Server:** when the controller throws (e.g. unreadable body → 400, or a 404), Boot
     forwards to `/error`. That path matched our `anyRequest().authenticated()`, and the
     anonymous error dispatch was re-rejected as **403** — so every genuine error was masked
     as "forbidden".
- **Fix:** permitted the error dispatch (`.requestMatchers("/error").permitAll()`) so real
  errors keep their real status (400/404/500). Then verified with **file-based** JSON bodies
  (`curl.exe --data-binary "@file.json"`), which PowerShell cannot mangle: full matrix
  register=201, duplicate=409, login=200, wrong-password=401, short-password=400. All 8 tests
  still pass.
- **Prevention:** never trust inline JSON in PowerShell→curl; use `--data-binary "@file"` (or
  the committed `test-auth.ps1`). Document that `/error` must be `permitAll` in every service's
  security config so errors are never masked. When a whole API seems to 403, check the server
  log for parse errors before suspecting the security rules.
- **Production relevance:** error-masking is a real production bug — if `/error` is secured,
  your 400s/404s/500s silently become 403s, breaking clients, alerts, and SLO error budgets.
  It also hid the real problem for a while: a bad client body looked like a server security
  failure. Real platforms pin error handling (permissive `/error`, a global exception handler)
  and their API tests send bodies from fixtures, not shell strings.

### Day 6 — False lead: suspected Spring Security 7 PathPatternRequestMatcher breakage

- **Symptom:** while chasing the 403, we suspected Spring Security 7.1 (Boot 4) had broken
  string `requestMatchers(...)` because the auth paths seemed to fall through to
  `anyRequest().authenticated()`.
- **Detection:** a web search surfaced issue spring-projects/spring-security#17808 — string
  matchers need a `PathPatternRequestMatcher.Builder` sharing Spring MVC's parser, which Boot
  was expected to auto-provide.
- **Cause:** investigation error — the observation "only /actuator/health is 200, everything
  else 403" was consistent with the mangled-body + secured-/error masking, NOT with broken
  matchers. The security rules were fine all along (proven later: clean-body requests got 201
  on the ORIGINAL code).
- **Fix:** added a `PathPatternRequestMatcherBuilderFactoryBean`, tested, saw no change, then
  found the real cause and **reverted** the bean.
- **Prevention:** reproduce with a clean request (file-based body) before blaming framework
  internals; change one variable at a time. The PathPatternRequestMatcher builder bean IS still
  the documented SS7/Boot wiring for custom parser setups — we just don't need it.
- **Production relevance:** misdiagnosis costs time. In real incidents this is why you capture
  the actual request/response bytes early (curl -v, access logs) — it separates "client sent
  garbage" from "server is broken" instantly.

### Day 6 — Spring Boot 4 validation annotations not on the classpath

- **Symptom:** `mvnw test` failed to compile: `cannot find symbol — class NotBlank/Email/Size/Valid`
  in `RegisterRequest`, `LoginRequest`, `AuthController`.
- **Detection:** the compiler errors in the test run.
- **Cause:** in Spring Boot 4 the web starter does NOT bundle the Jakarta validation API;
  `spring-boot-starter-validation` is a separate starter we hadn't added.
- **Fix:** added `spring-boot-starter-validation` to `pom.xml`; tests then compiled and passed.
- **Prevention:** after any scaffold, compile once before writing tests; if a standard
  annotation is missing, check for a dedicated starter (Boot splits concerns into starters).
- **Production relevance:** every real project hits "which starter/artifact owns this class"
  — that's what the dependency tree (`mvnw dependency:tree`) is for. CI compiles on every
  push so this is caught in seconds, not at deploy time.

### Day 6 — Tests shared one in-memory store → "expected 201 but was 409"

- **Symptom:** `duplicateEmailReturnsConflict`, `loginReturnsJwt`, `wrongPasswordReturnsUnauthorized`
  all failed at their first step: `Status expected:<201> but was:<409>` when registering.
- **Detection:** Surefire failures; the same email (`alice@example.com`) was used in 4 tests.
- **Cause:** `@SpringBootTest` reuses ONE Spring context (and therefore one `UserStore`) for
  the whole test class, so whichever test registered `alice` first left it there; later
  tests saw an existing user → 409. Tests were secretly order-dependent.
- **Fix:** gave each test its own unique email (data isolation per test).
- **Prevention:** never share mutable state or rely on test order. Use unique fixtures per
  test, or `@DirtiesContext`/`@Transactional` rollback when shared state is unavoidable.
- **Production relevance:** a test suite that passes only in a certain order is a time bomb
  — real suites run in parallel (Maven surefire `parallel`, CI sharding), and a "works in
  my run order" suite explodes there. Isolation is a correctness requirement, not a nicety.

### Day 6 — Leftover Day 5 JVM was squatting port 8080

- **Symptom:** `mvnw spring-boot:run` failed: "Port 8080 was already in use" / another
  process is listening on port 8080.
- **Detection:** the startup error, then `Get-NetTCPConnection -LocalPort 8080 -State Listen`
  → owning process PID 2080 = `com.stockforge.api.StockforgeApiApplication` (Day 5 server
  never fully stopped).
- **Cause:** the previous day's `spring-boot:run` JVM was left running.
- **Fix:** `Stop-Process -Id 2080 -Force`, confirmed the port was free, then started the
  auth server.
- **Prevention:** always stop `spring-boot:run` servers when done; when a port is busy,
  identify the owner before killing anything (never blindly `taskkill`).
- **Production relevance:** port conflicts are the local version of "two services fighting
  over a resource" — real platforms manage this with orchestration (K8s ports, service
  discovery) and never start a second replica without knowing what holds the port.

### Day 6 — start.spring.io Boot 4 web dependency id is `web`, not `webmvc`

- **Symptom:** the initial scaffold request failed: `Unknown dependency 'webmvc'` — Day 5
  used the `webmvc` id successfully, so the same id was reused.
- **Detection:** start.spring.io rejected the dependency id at URL-build time.
- **Cause:** Boot 4 generator renamed/consolidated the web starter ids; on Boot 4 the id is
  `web` (the `webmvc` alias seen on Day 5 was not accepted this time).
- **Fix:** requested `web` instead; scaffold succeeded.
- **Prevention:** when start.spring.io rejects a dependency id, query the generator's
  dependency metadata (or try the plain `web` id) before retrying blindly.
- **Production relevance:** dependency metadata drift between generator versions is exactly
  why real teams pin scaffold/config versions and lock dependencies — "it worked last week".

---

## Day 4 — `stockforge-web` scaffold (2026-08-07)

### Day 4 — Environment drift: Node version mismatched the state file

- **Symptom:** `CURRENT_STATE.md` claimed Node v24.14.1 was installed; `node --version`
  returned **v20.19.6**.
- **Cause:** state file was written before the actual environment was checked.
- **Fix:** recorded the real version in the Day 4 notes and the web README; no code impact.
- **Production relevance:** environment drift (dev vs prod versions) is a top cause of
  "works on my machine". Real platforms pin versions (`.nvmrc`, container images, IaC)
  and detect drift in CI. We learned to verify tooling before trusting docs.

### Day 4 — PowerShell one-liner quoting broke the dev-server verification

- **Symptom:** a single-line PowerShell command with nested quotes failed to parse
  (parser errors about `root` tokens) while trying to check the dev server.
- **Cause:** complex quoting of `"id=\"root\""` inside a one-liner; `cmd /c` style
  escaping issues in PowerShell 5.1.
- **Fix:** wrote the check as a proper `.ps1` script file and executed it with
  `powershell -File` — clean parse, verified HTTP 200 + `#root` present, then stopped the server.
- **Production relevance:** automation should be script files, not fragile one-liners —
  the same reason real platforms keep pipelines (Jenkinsfile / Actions YAML) as
  version-controlled files instead of copy-pasted commands.

### Day 4 — Stray npm error ENOENT when running `npm run dev` from the wrong folder

- **Symptom:** `npm run dev` failed: `ENOENT ... could not read package.json` at
  `D:\Jotiraditya(personalgit)\Stock-Forge\package.json`.
- **Cause:** the command was run from the root `Stock-Forge` folder (no `package.json`),
  not from inside `stockforge-web`.
- **Fix:** run inside `stockforge-web` (the repo owns its own `package.json`).
- **Production relevance:** build tooling must run in the repo root that owns the
  manifest; CI/CD does this by checking out and cd-ing into the exact repo directory.

---

## Day 3 — `stockforge-contracts` (2026-08-06)

### Day 3 — OpenAPI YAML wrapped in a Markdown fence is invalid YAML

- **Symptom:** the first version of `contracts/openapi.yaml` was written with a
  ```yaml fence around the content; YAML validation failed.
- **Cause:** a `.yaml` file must be pure YAML — Markdown fenced code blocks are a
  different format entirely.
- **Fix:** rewrote the file as pure YAML; validated with
  `python -c "import yaml; yaml.safe_load(open(...))"`.
- **Production relevance:** specs must be machine-parseable and are validated in CI
  (swagger-cli / schema registry). A spec that doesn't parse breaks code generation,
  docs, and contract tests — exactly why contract-first repos gate on validation.

---

## Day 5 — `stockforge-api` (2026-08-07)

### Day 5 — Toolchain gate: no JDK 21, no Maven, no admin on this device

- **Symptom:** tooling check for Day 5 found **Java 17.0.12** (stack wants JDK 21) and
  **no Maven**; and this device has no complete admin rights, so a normal JDK installer
  is not viable.
- **Cause:** environment on this device was never provisioned for the approved stack;
  the state file's tooling assumptions had not been verified.
- **Fix:** Maven is handled by the **Maven Wrapper** (`mvnw`), which pins its own version —
  no global install needed. Decision: do JDK 21 setup + the Day 5 build on the **personal
  PC (full admin)**. If needed later on a no-admin box, the Temurin 21 **portable ZIP**
  + user-level `JAVA_HOME`/`PATH` works without admin.
- **Production relevance:** this is exactly why real platforms pin toolchains (`.nvmrc`,
  container images, IaC) and run CI on clean runners. Until we containerize (Phase 12-13),
  JVM work happens on the provisioned machine. "Works on my machine" is a production
  incident class, not a joke.

### Day 5 — Antivirus/AMSI blocked a JDK download command (false positive)

- **Symptom:** a PowerShell command that created a tools dir and ran
  `curl.exe -L -o ... https://api.adoptium.net/...` was blocked: "This script contains
  malicious content and has been blocked by your antivirus software."
- **Cause:** AMSI/antivirus heuristics flagged the command (download URL + `-o` write)
  as suspicious — a false positive, not real malware.
- **Fix:** abandoned the download on this device; moved the JDK setup to the personal PC.
  For future: run security-sensitive downloads on the admin machine, or break into
  smaller, clearly-benchign steps (e.g. a `.ps1` script file).
- **Production relevance:** security tooling that false-positives on legitimate build
  steps is a known friction point in real CI; teams pin images / artifact feeds and
  maintain allowlists. It also reinforces why downloads belong on a provisioned machine.

### Day 5 — start.spring.io emitted a Spring Boot version that doesn't exist on Maven Central

- **Symptom:** first `.\mvnw test` failed: `Non-resolvable parent POM ...
  spring-boot-starter-parent:pom:4.1.0.RELEASE` — "Could not find artifact ... in central".
- **Detection:** the Maven build error itself; then we queried
  `https://repo.maven.apache.org/maven2/org/springframework/boot/spring-boot-starter-parent/maven-metadata.xml`
  and saw real versions are `4.1.0` (no `.RELEASE` suffix).
- **Cause:** `start.spring.io` labels the Boot 4 versions `4.1.0.RELEASE` in its metadata,
  but Maven Central publishes them as `4.1.0`. The generator's label ≠ the artifact version.
- **Fix:** changed `<version>4.1.0.RELEASE</version>` → `<version>4.1.0</version>` in `pom.xml`.
- **Prevention:** after any `start.spring.io` scaffold, verify the parent version resolves —
  check `maven-metadata.xml` or run `mvnw -N validate` before adding code.
- **Production relevance:** dependency-resolution failures are a classic CI day-1 problem;
  pinning exact, verified versions (lockfiles / Bill of Materials) prevents "it worked on
  my machine" drift. In real teams this is caught by a CI job the moment a branch is pushed.

### Day 5 — Spring Boot 4 relocated `@AutoConfigureMockMvc` (package + starter changed)

- **Symptom:** `HealthControllerTests` failed to compile: `package
  org.springframework.boot.test.autoconfigure.web.servlet does not exist`.
- **Detection:** `mvnw test` compiler error; then we downloaded
  `spring-boot-test-autoconfigure-4.1.0.jar` and listed it — no `web.servlet` package.
  Inspecting `spring-boot-starter-webmvc-test`'s POM pointed to the `spring-boot-webmvc-test`
  module, whose jar contains the class.
- **Cause:** Spring Boot 4 reorganized test support: per-application-type test starters
  (`spring-boot-starter-webmvc-test`), and MockMvc autoconfig moved to package
  `org.springframework.boot.webmvc.test.autoconfigure`.
- **Fix:** changed the import in the test to
  `org.springframework.boot.webmvc.test.autoconfigure.AutoConfigureMockMvc`
  and removed a needless extra `spring-boot-test-autoconfigure` dependency.
- **Prevention:** when upgrading a framework major version, search for moved packages before
  assuming imports; use the scaffold's own generated test as the source of truth.
- **Production relevance:** framework major upgrades are exactly where "small" breaking
  changes surface; real teams lean on the migration guide + a clean CI build. Same lesson
  as ADR 0002's Boot 3.x → 4.1.0 note.

### Day 5 — `run.log` committed by accident

- **Symptom:** `git status`/commit included `run.log` (the captured `spring-boot:run` output)
  inside the first commit.
- **Detection:** `git status` showed `?? run.log` before staging; after committing, a
  `git rm --cached` + `.gitignore` entry was needed.
- **Cause:** the log file was created in the repo root by the run-verification step and
  staged with `git add -A` before being cleaned up.
- **Fix:** `git rm --cached run.log`, appended `run.log` to `.gitignore`, committed the fix
  (`b0788bb`), pushed.
- **Prevention:** add generated/runtime files to `.gitignore` (or review `git status`
  carefully) before `git add -A`; the Spring Initializr `.gitignore` covers `target/` but
  not ad-hoc root log files.
- **Production relevance:** committing build/runtime artifacts bloats repos and can leak
  data; real repos have `.gitignore` + CI checks (e.g. "no unexpected files in root").

---

## Verification session — 2026-08-08 (full run of everything built so far)

### Verify — PowerShell blocks `npm` ("running scripts is disabled on this system")

- **Symptom:** running `npm install` in PowerShell failed with a `PSSecurityException` /
  `UnauthorizedAccess`: `File C:\Program Files\nodejs\npm.ps1 cannot be loaded because running
  scripts is disabled on this system`.
- **Detection:** immediate command failure; the error points at the `.ps1` wrapper that npm
  installs for PowerShell.
- **Cause:** the ExecutionPolicy on this Windows box is `Restricted`/`RemoteSigned`-not-applied,
  so PowerShell refuses to run any `.ps1` script (including npm's launcher). This is NOT an npm
  problem — `npm.cmd` (the batch launcher) is unaffected.
- **Fix:** use `npm.cmd install` / `npm.cmd run dev` in PowerShell, run from `cmd.exe`, or set
  `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` once in an admin PowerShell to allow
  signed/local scripts permanently.
- **Prevention:** remember the rule "PowerShell executes .ps1; cmd.exe executes .cmd"; use the
  `.cmd` variant for node/npm tooling unless the policy is changed.
- **Production relevance:** same class of issue as the curl-inline-JSON 403: the *tool layer*
  (shell/quoting/policy) can block a perfectly good app. Check the launcher before blaming the
  software. In CI the equivalent is "works on my machine" — pin tooling versions and run in a
  consistent environment.

### Verify — `stockforge-api` and `stockforge-auth` both bind port 8080 (documented, not a bug)

- **Symptom:** starting `stockforge-auth` after `stockforge-api` (or vice versa) fails with
  "port already in use", or the wrong service answers a health request.
- **Detection:** `Get-NetTCPConnection -LocalPort 8080 -State Listen` shows the survivor; the
  second Spring Boot instance fails to start.
- **Cause:** both services default to `server.port: 8080` in their `application.yml`, and they
  are separate processes — nothing starts the API automatically when auth starts.
- **Fix:** run only one Spring service at a time on :8080 (the intended local workflow today);
  later, Docker Compose gives each service its own port/container.
- **Prevention:** document ports per service (done in the Run & Test Playbook); when a service
  needs simultaneous local testing, pass `-Dserver.port=8081` (as the Day 6 debug instance did).
- **Production relevance:** in production each service gets its own port/container and service
  name — this local collision is purely a dev-machine artifact, but the lesson (know which
  process owns which port) is how port-conflict incidents are debugged everywhere.

---
