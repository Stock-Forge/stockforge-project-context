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

---

## Day 5 — `stockforge-api` (pending — fill in at end of day)

- (No issues logged yet.)
