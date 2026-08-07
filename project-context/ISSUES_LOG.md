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
