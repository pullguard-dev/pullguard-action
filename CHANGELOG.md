# PullGuard changelog

All notable customer-visible changes. Earlier releases predate this
changelog; this file is the canonical record going forward.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Live release notes for the hosted scanner: [pullguard.dev](https://www.pullguard.dev).

## [Unreleased]

_Customer-visible changes already live on `:latest` but not yet bundled into a cut image tag. Pin a specific release below for change-controlled, reproducible scans._

---

## [1.2.5] — 2026-06-10

A security-depth release for Java. PullGuard now catches two classes of real vulnerability that a leading free SAST tool flagged but earlier versions missed — at PullGuard's zero-false-positive bar (it matches the genuine findings without that tool's noise). Pin it with `image-pin: v1.2.5`, or stay on `image-pin: 1` (re-pointed to v1.2.5).

### Added — security depth

- **Insecure temporary files (CWE-377).** Flags `File`/`Files.createTempFile` with no directory argument — it lands in the world-writable system temp dir, a time-of-check/time-of-use race on shared hosts. Calls that pass an explicit private directory are correctly treated as safe.
- **Regular-expression denial of service (ReDoS, CWE-1333).** Flags backtracking-prone regexes that run on attacker-controlled input (request bodies, log lines, external URLs). It fires only when the dangerous pattern actually runs *on* untrusted data — patterns on configuration values, linear character-class patterns, and non-backtracking (possessive/atomic) regexes are deliberately never flagged.
- **Broader Java coverage:** raw request bodies, query-predicate-builder injection, and script-engine evaluation are now tracked; credential defaults committed to secret-marked fields are flagged.

### Fixed

- **ReDoS detection is more precise:** no more false positives on division operators in minified code, comment banners, regex metacharacters inside character classes, or non-backtracking patterns. Genuine catastrophic-backtracking patterns are still caught.
- **Dependency-CVE findings carry structured version data** (package, matched version, fixed versions), so a finding that changes between scans is explainable from the report. A caret-ranged dependency CVE that could previously disappear silently is now pinned.
- **Workflow findings** that group several different unpinned actions now list each action with its own line.

---

## [1.2.4] — 2026-06-09

A precision release for Java codebases: substantially fewer false positives on Spring / JAX-RS / CMS stacks, broader genuine coverage, and clearer findings — so the results on your PRs hold up to scrutiny. Every change ships with regression tests and was validated with no loss of genuine findings. Pin it with `image-pin: v1.2.4`, or stay on `image-pin: 1` (re-pointed to v1.2.4) to pick it up on your next run.

### Fixed — fewer false positives

- **Java is much quieter and more accurate.** Bean setters are no longer mistaken for JavaScript timers; parameterised SLF4J logging is no longer flagged as Log4Shell when Log4j2 isn't on the classpath; constant-time comparison checks no longer fire on ordinary collection/value `.equals()`; generated-secret blocks in CloudFormation and already-patched dependency versions are recognised correctly.
- **Cleaner output.** Readable taint labels (no raw regex), consistent occurrence counts, descriptions truncated at word boundaries, and an explicit cell for repository-wide findings.

### Added

- **Detects hardcoded developer/home paths** (e.g. `/Users/<name>/…`) that leak a username and break portability — CI and system paths are excluded.
- **Broader genuine coverage** of request inputs and query/output sinks on Java (Spring + JAX-RS) stacks.
- **Workflow scanning** now covers every job in a multi-job workflow, plus `secrets: inherit` and clone-then-execute supply-chain patterns.

### Docs

- Documented `.driftrc.yml` configuration and the `.pullguardignore` per-finding suppression file.

---

## [1.2.3] — 2026-06-05

A signal-quality release: substantially fewer false positives, several closed false negatives, and more accurate CVE matching — so the findings on your PRs are actionable as-is. Every change ships with regression tests and was validated against real vulnerable applications with **no loss of genuine findings**. Pin it with `image-pin: v1.2.3`, or stay on `image-pin: 1` (re-pointed to v1.2.3) to pick it up on your next run.

### Fixed — fewer false positives

- **Findings now point at the exact file and line.** A large fraction of findings previously had no usable location (or pointed at the wrong file), which made them hard to action. The scanner now backfills an accurate location for every finding, and SARIF uploads to GitHub Code Scanning always carry a valid location.
- **Infrastructure config no longer false-flagged as hardcoded secrets.** Environment references, placeholders, and CloudFormation generated/resolved secrets (`GenerateSecretString`, `{{resolve:secretsmanager:…}}`, and CamelCase resource names ending `…Secret:`) are no longer reported as committed secrets. Real committed secrets — including in comments — still flag.
- **Idiomatic template / redirect / reflection code is no longer flagged as critical.** Server-side template rendering, redirects, dynamic reflection, and `jwt.decode` are now only reported when attacker-controlled input actually reaches them — not on routine framework code.
- **Unsafe-deserializer mentions inside comments / documentation are no longer flagged** (only real code is).

### Fixed — closed security false negatives

- **Spring & JAX-RS controllers are now fully covered.** Request parameters via `@RequestParam` / `@PathVariable` / `@RequestBody` / `@RequestHeader` (and JAX-RS `@QueryParam` / `@PathParam` / `@FormParam` / …) flowing into SQL, XPath, LDAP, email-header, open-redirect, or template sinks are now detected — previously many of these were silently missed.
- **More raw sinks detected:** Spring `JdbcTemplate` raw SQL, Python `subprocess`/`os.exec*`, Prisma `$queryRawUnsafe`/`$executeRawUnsafe`, Python `urlopen`/`aiohttp`, PHP `unserialize`, and .NET binary/SOAP deserializers — each only when untrusted input reaches them.
- **Path-traversal, log-injection, and arrow-function-handler flows** that were previously missed are now detected.

### Fixed — more accurate CVE matching

- **Version comparison rewritten** to correctly handle Maven release qualifiers (`.Final`/`.GA`/`.RELEASE`), Red Hat rebuild suffixes, semantic-version pre-releases, and PyPI/PEP 440 — fixing both false CVE matches against already-patched versions and missed pre-release vulnerabilities.
- **Maven BOM-managed dependencies resolve their real version** (via `<properties>` / `${…}` / platform BOM) instead of being flagged for every CVE for the package — e.g. a Quarkus platform dependency is no longer flagged for CVEs that were fixed in a lower patch.

### Changed — clearer reports

- **Multiple findings of the same type in one file now show a count and the affected lines** on every report surface (PR comment, step summary, text), instead of collapsing to a single under-reported row.
- **If an analyzer times out, the report now says so explicitly** (and names it) rather than silently omitting its findings — so an incomplete check is never mistaken for a clean result.
- SARIF reports the real scanner version; `except Exception:` is described accurately and graded appropriately.

### Changed — honest claims

- Enterprise SSO / audit-log / RBAC are labelled as roadmap / contracted deliverables rather than shipping features; analyzer counts, compliance scope wording, and an AI-signal description were corrected to match what the scanner does.

### Privacy & transparency

- The privacy policy now discloses our processing sub-processors (Cloudflare, Resend, GitHub) and gives accurate per-endpoint data-retention details.

---

## [1.2.2] — 2026-06-04

Two efforts in one image: **Tier-2 zero-day rule precision** and a **signal-quality false-positive / false-negative batch** from an enterprise pre-review audit. The net effect on your scans is less noise and a couple of genuine issues that were previously under-reported. Pin it with `image-pin: v1.2.2`, or stay on `image-pin: 1` (re-pointed to v1.2.2) for the improvements on your next run.

### Fixed — fewer false positives

- **Authentication entry points are no longer flagged "missing authentication."**
  A login / signup / oauth / SSO route is unauthenticated by design — you can't
  require a session to reach the login page. The suppression is deliberately
  narrow: infra/ops endpoints (`/health`, `/metrics`, actuator) and protected
  routes that merely contain a `password` segment (e.g. `/account/password`)
  **still** flag, because those can be genuine exposures.
- **Environment variables are no longer treated as attacker input.** Config-driven
  code like `fetch(process.env.API_URL)` or `exec(System.getenv("CMD"))` is no
  longer reported as SSRF / injection — environment variables (and JVM system
  properties) are operator-controlled configuration, matching the default
  behaviour of CodeQL and Semgrep. Removes a class of false positives on frontend
  and config-driven code. Command-line args, stdin, and all HTTP request inputs
  remain taint sources, so genuine injection flows are unaffected.
- **Behavioural security findings on test / fixture / mock / Storybook files are
  suppressed.** A SQL-injection, XSS, or prototype-pollution pattern inside a
  `.spec.ts`, a codemod `__testfixtures__` sample, a `__mocks__` file, a generated
  `mockServiceWorker.js`, or a Storybook story is test/demonstration code that
  never runs in production. Deliberately narrow — it does NOT touch `examples/`,
  `components/`, or `pages/` (real runnable code), and committed secrets are still
  flagged everywhere.
- **Frontend `router.post` / `app.delete` (React / axios client wrappers) are no
  longer mistaken for missing-CSRF Express routes.**
- **Optional-chaining (`?.`) and nullish-coalescing (`??`) no longer inflate
  cyclomatic-complexity findings**; genuine ternaries still count.
- **Clone detection skips test / fixture / migration boilerplate.**

### Fixed — closed security false negatives

- **CVE severity is no longer silently downgraded** when the vulnerability feed
  returns a CVSS *vector string* instead of a numeric score (the official CVSS
  v3.1 base score is now computed).
- **GitHub Actions script-injection is now detected in `actions/github-script`
  `with.script` bodies** (privileged `GITHUB_TOKEN` context), not just shell
  `run:` blocks.
- **Tier-2 zero-day precision (Spring):** `@RequestParam` / `@PathVariable` /
  `@RequestHeader`-decorated parameters are now bound as taint sources — closing
  SpEL / Log4Shell / Text4Shell false negatives on textbook Spring controllers.
  Text4Shell (Commons Text) detection now fires on all canonical idioms, and
  SpEL no longer mis-labels OGNL findings.

### Config surfaces that now work

- `.driftrc.yml` path-like excludes (`src/legacy`, `deployment/k8s`) take effect
  (previously had to be written as `*` globs).
- Per-analyzer `analyzers.<id>.severity` override is applied.
- `thresholds.duplication.minBlockSize` is honoured.

### Image tags

- `:v1.2.2`, `:1.2.2`, `:1.2`, `:1`, and `:latest` re-pointed to the new digest
  (`sha256:d79455c9807daff6c9856178e48626db733104edbcbd1303c45fdb15998bc183`).
- `:v1.2.1` and `:1.2.1` are retained at their existing digest
  (`sha256:485925d26447191520fd68b3e3f1df9c5fabcabd760fc235c38214ffd76a589f`)
  for anyone pinned.

---

## [1.2.1] — 2026-05-28

Signal-quality patch. A multi-repo rollout sweep across real customer-class codebases (NodeGoat, DVWA, WebGoat, Juice Shop, plus a representative React/Vite admin app and a TypeScript ops-desk UI) surfaced a cluster of false positives and one stateful-regex false negative. All closed structurally so the fixes hold across your codebase. Pin it with `image-pin: v1.2.1`, or stay on `image-pin: 1` (re-pointed to v1.2.1) for the precision wins on your next run.

### Fixed

- **Frontend HTTP-client files are no longer mistaken for unauthenticated Express
  routes.** A genuine server file is now required before missing-auth checks
  run — at least one of: the `express` module, a `Router()` factory, an `app.` /
  `router.` route or middleware receiver, an Express `res.` / `req.` object, or
  a NestJS class decorator. A real route that just omits the `express` import
  still flags — no false negatives. Closes the React/Vite-with-api-client false
  positive cluster.
- **Embedded base64 image / font / binary blobs are no longer flagged as AWS
  keys.** A 20-character key-shaped substring that sits inside a long
  contiguous base64 run, or immediately after a `data:<mime>;base64,` prefix,
  is treated as encoded data, not a credential. Real keys outside an encoded
  blob still report.
- **i18n / UI label values and obvious placeholders are no longer flagged as
  hardcoded secrets** by the loose `secret/password` pattern. Rejected shapes:
  values containing whitespace ("Password (if protected post)"), placeholder
  words ("changeme", "YOUR_API_KEY", "xxxxxxxxxx", "password", "your-pw"),
  masked values, `${SECRET}` templates, `process.env.X` references.
  Provider-specific keys (AWS, GitHub, Slack, Stripe, PEM private keys) stay
  strict and are NOT gated by this rubric.
- **HTML inline `<script>` blocks are no longer flagged as `command_injection`.**
  The previous rule was over-broad for static HTML; the dynamic-execution
  analyzers still cover real injection paths.
- **Node `child_process.exec` is no longer included in the SQL-injection
  regex.** It's a command-execution sink, not a SQL sink — the
  command-injection analyzer covers it correctly.
- **Cross-file false negative closed in the OWASP regex catalogs.** A stateful
  regex matcher could leave its internal cursor past a real hit on the
  previous file, silently missing the same vulnerability on the next file.
  Detection is now deterministic per file. Order-independent.
- **Security findings cluster sort order** is now consistent critical → major →
  moderate → minor → info across the PR comment, step summary, and JSON
  output. No emoji-vs-content drift.

### Image tags

- `:v1.2.1`, `:1.2.1`, `:1.2`, `:1`, and `:latest` re-pointed to the new digest
  (`sha256:485925d26447191520fd68b3e3f1df9c5fabcabd760fc235c38214ffd76a589f`).
- `:v1.2.0` and `:1.2.0` are retained at their existing digest
  (`sha256:06e0e41fa6a575839c2542b038906d1926588763ed7a0d8ae924268eff53c8b0`)
  for anyone pinned.

---

## [1.2.0] — 2026-05-23

AI-era security: secure and govern the code your AI writes, exploit-aware prioritization of known vulnerabilities, and a set of authorization-aware precision fixes. Pin it with `image-pin: v1.2.0`.

### Added

- **Security focus on AI-generated code.** When a security flaw appears in code
  that shows AI-generation signals, PullGuard now raises a single prioritized
  "review this AI-written code" finding instead of separate AI and security
  findings. Standalone AI-generation findings are informational only — signal,
  not noise.
- **AI-governance evidence.** Each scan reports the share of analyzed code that
  shows AI-generation signals, as advisory evidence toward EU AI Act and NIST AI
  RMF programs. Evidence *toward* governance — not a grant of compliance.
- **AI application-security coverage (OWASP LLM & Agentic Top-10).** Taint
  analysis now follows untrusted data into LLM completion, RAG / vector-retrieval,
  agent, and model-loading sinks — across hosted, private, and self-hosted model
  endpoints — and treats model output as untrusted input.
- **Exploit-aware vulnerability prioritization.** Known-CVE findings now carry an
  EPSS exploit-probability score and a CISA KEV "actively exploited in the wild"
  flag, so the vulnerabilities most likely to be attacked rise to the top. The
  actively-exploited feed refreshes continuously on PullGuard's side — your scans
  pick up newly-flagged CVEs without changing your pinned image.

### Changed

- **Authorization guards are recognized, not just authentication.** Routes
  protected by an authorization layer — Spring's centralized `SecurityFilterChain`,
  or Express / NestJS guards such as `isAuthorized`, `isAdmin`, `hasRole`, and
  `denyAll` — are no longer reported as missing authentication.
- **Vendored, generated, and minified third-party code is excluded from
  source-code security findings** (it is still scanned for committed secrets).

### Fixed

- **More committed-credential files are detected** — private keys (`.pem`,
  `id_rsa`), `.netrc`, and similar — even when they are not a scanned source type.
- **Cross-file taint findings point at a real, clickable line** (previously some
  could reference a line past the end of the file).
- **Fewer false "orphaned file" findings** for browser-served assets and for
  modules wired together with CommonJS `require()`.

---

## [1.1.2] — 2026-05-22

Patch release. Pin it with `image-pin: v1.1.2`.

### Fixed

- **False positive in unused-export detection on Python projects.** A symbol
  exported by one module and imported elsewhere via an absolute import
  (`from config import name`) could be flagged "not imported by any other file".
  Absolute imports are now resolved correctly; genuinely-unused exports are still
  reported.

---

## [1.1.1] — 2026-05-22

Patch release. Pin it with `image-pin: v1.1.1`.

### Fixed

- **SOC 2 compliance evidence now renders correctly.** A rendering issue could
  cause the SOC 2 section to show "0/0" controls (and omit the SOC 2 evidence
  table) on a scan; it now displays the full SOC 2 control status as intended.
  The other four frameworks (HIPAA / PCI DSS / NIST / ISO) were unaffected.

---

## [1.1.0] — 2026-05-22

Compliance evidence now renders for all five frameworks on every PR, the `@v1`
default moves to the stable release channel, and an authorization fix on the
optional Check Run integration. Pin it with `image-pin: v1.1.0`.

### Added

- **Compliance evidence for all five frameworks on every PR.** Alongside SOC 2,
  each PR now shows a compact PASS / CONCERN / FAIL summary for HIPAA Technical
  Safeguards, PCI DSS 4.0, NIST 800-53 Rev 5, and ISO 27001:2022. Turn on the
  full per-control evidence tables for any of the four with
  `compliance: { hipaa: true, pci-dss: true, nist: true, iso-27001: true }` in
  your `.driftrc.yml`. PullGuard provides evidence *toward* these frameworks — it
  does not grant compliance.

### Changed

- **`@v1` now defaults to the stable release line, not the bleeding edge.**
  If you use `uses: pullguard-dev/pullguard-action@v1` without setting
  `image-pin`, PullGuard now runs the newest **stable release** instead of
  the rolling image that rebuilt on every change. Your scans are reproducible
  between runs and still pick up each new release automatically — without the
  occasional churn of an in-progress build. To keep the previous
  always-newest behaviour, set `image-pin: latest`. To freeze an exact
  version, set `image-pin: v1.1.0` (or a `sha256:…` digest).

### Fixed

- Hardened authorization on the optional GitHub App Check Run integration —
  unlimited-repo (Enterprise) tokens are now scoped to their own organization.
- Documentation accuracy: the privacy policy now describes the single
  license-validation request made during a scan (your license key + repository
  name only — never your source code or scan results), and the cost-of-change
  formula and per-tier analyzer counts now match the product.

---

## [1.0.0] — 2026-05-20 (Spring 2026 release)

PullGuard's first pinnable release. 43 analyzers across security, code
quality, compliance, and supply-chain; five compliance frameworks (SOC 2,
HIPAA, PCI DSS 4.0, NIST 800-53 Rev 5, ISO 27001:2022); endpoint-risk
prioritisation; and actionable-vs-observational cost reporting. Pin it with
`image-pin: v1.0.0`.

### Added

- **11 new security analyzers** — coverage now includes CSRF protection,
  cookie security flags, JWT confusion attacks, HTTP security headers,
  server-side template injection, unsafe reflection, file-upload
  validation, cryptographic hygiene, generic injection sinks, Kubernetes
  IaC security (CIS Level 1), and missing-authentication detection on
  framework routes (Express / NestJS / Spring / Django / Flask / Rails).

- **Endpoint Risk Engine** — composes per-endpoint authentication state,
  data-flow taint, and call-graph reachability into a unified risk
  tier. An unauthenticated endpoint reachable from a tainted source
  becomes a **critical** finding rather than three disconnected
  medium-severity findings, giving security teams a clearer
  prioritisation signal.

- **Compliance framework expansion** — added HIPAA Technical Safeguards
  (45 CFR §164.312), PCI DSS 4.0, NIST 800-53 Rev 5, and ISO 27001:2022
  evidence mappings alongside the existing SOC 2 coverage. SOC 2 itself
  expanded from 4 to 8 in-scope controls (added CC4.1, CC6.2, CC6.7,
  CC8.2).

- **`.pullguardignore` suppression file** with PR-comment workflow —
  comment `/pullguard ignore <rule-id>` on a pull request and PullGuard
  opens a suggested edit to your `.pullguardignore`. Security-category
  analyzers cannot be suppressed by this workflow.

- **Cost partition (actionable vs observational)** — fix-cost is now
  reported as **actionable** (Moderate severity and above) versus
  **observations** (Info / Minor). Headlines reflect what an
  engineering leader would actually sign off on fixing rather than the
  inflated all-findings total.

- **`image-pin` action input** — pin the scanner to an immutable release
  tag (`image-pin: v1.0.0`) or content digest (`image-pin: sha256:…`)
  instead of the rolling `:latest`. Recommended for enterprise
  change-control and reproducible builds. Defaults to `latest`, so
  existing workflows keep their current behaviour with no change required.

### Changed

- **Tier analyzer counts**: Free **14** (unchanged), Pro **31 → 42**,
  Team & Enterprise **32 → 43**. License keys auto-pick up the new
  analyzers; no customer action required.

- **PR comment + Step Summary headers** — display
  `N actionable findings (+M observations) · $X actionable` instead of
  the previous all-findings aggregate. Each section in the PR comment
  is now collapsible; full finding descriptions render inline rather
  than truncating at 80 characters.

- **Per-language file-size thresholds** — Java tolerates 1000-line
  files (idiomatic for content-management codebases), Go 400,
  TypeScript 600, etc., replacing a single global threshold that
  unfairly penalised verbose-by-convention languages.

- **Honest line counts** — file-size and function-length checks no
  longer count comment-only or blank lines. Heavily-documented code
  is no longer flagged for documentation density. Aligned with the
  industry-standard NCLOC (Non-Commented Lines of Code) metric.

- **GitHub Actions Marketplace listing** — analyzer count, pricing
  tier descriptions, and capability claims updated across all
  customer-facing surfaces.

### Fixed

- **CI reliability** on macOS runners for cross-repo verification —
  improved authentication path reduces transient failures.

- **False positives on documented code** — typical web-framework
  codebases see roughly 30 spurious "monolithic file / function"
  findings removed per repository as a result of the line-count
  change above.

- **PR comment rendering** — descriptions no longer truncated mid-word;
  overflow findings are reachable via collapsible sections instead of
  dead "and N more findings" text.

### Compliance

- **Multi-framework evidence sections** appear in the Step Summary +
  PR comment when the relevant analyzers run. Each control is mapped
  to the analyzers that produce evidence for it; the report shows
  PASS / CONCERN / FAIL per control with the underlying violation
  count. Auditors get a one-screenshot answer; engineering teams get
  the line-by-line drill-in.

---

## How to read this changelog

- **`Added`** — new capabilities customers can use
- **`Changed`** — behaviour or output changes affecting existing scans
- **`Fixed`** — bug fixes that resolve incorrect or missing output
- **`Compliance`** — changes to compliance-evidence reporting

PullGuard is continuously deployed: the `pullguard-dev/pullguard:latest`
container image is rebuilt on every merge to the scanner repository's main
branch, and a numbered **release** (`:1.0.0`, `:1.1.0`, …) is cut from it
periodically. Customers using `uses: pullguard-dev/pullguard-action@v1`
default to the **stable release line** — they update to each new release
automatically but never to an in-progress build. Set `image-pin: latest` to
track every merge instead.

For change-controlled or reproducible scans, freeze an exact version with the
`image-pin` input (e.g. `image-pin: v1.0.0`, or a `sha256:…` digest). You then
get exactly the capabilities listed under that release section above, frozen
until you choose to move the pin — each numbered release here corresponds to an
immutable `ghcr.io/pullguard-dev/pullguard:vX.Y.Z` image tag.

For policy questions, security disclosures, or to discuss enterprise
deployment options: [hello@pullguard.dev](mailto:hello@pullguard.dev).
