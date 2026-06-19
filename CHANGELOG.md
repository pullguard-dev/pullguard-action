# PullGuard changelog

All notable customer-visible changes. Earlier releases predate this
changelog; this file is the canonical record going forward.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Live release notes for the hosted scanner: [pullguard.dev](https://www.pullguard.dev).

## [Unreleased]

_Customer-visible changes already live on `:latest` but not yet bundled into a cut image tag. Pin a specific release below for change-controlled, reproducible scans._

---

## [1.3.0] — 2026-06-19

The **AI-era security release**. PullGuard adds a new category of checks for the security risks that AI-assisted development introduces — both the code your team writes with AI and the code that calls AI — alongside continued false-positive reduction. **44 analyzers** (Free 14 / Pro 42 / Team & Enterprise 44). Pin it with `image-pin: v1.3.0`, or stay on `image-pin: 1` (re-pointed to v1.3.0) to pick it up on your next run.

### Added — AI-era security

- **Protect sensitive data from AI** — surfaces where secrets or personal data could be exposed to an AI model. *(Pro and above)*
- **Untrusted input reaching AI** — flags attacker-influenced data flowing into AI and agent features.
- **Risky AI-generated code** — catches common insecure defaults and unsafe patterns that AI coding assistants frequently produce, including dependencies that don't resolve to a real package.
- **AI safety & cost guardrails** — flags disabled AI safety controls and unbounded autonomous-agent behaviour.
- **AI × security risk scoring** — highlights the most dangerous combinations so they rise to the top of the report.
- **AI governance evidence** — supports transparency obligations under EU AI Act Article 50. *(Enterprise)*

### Added — configuration

- Import your existing rules and a complete configuration reference, so PullGuard fits your team's policies.

### Changed

- Replaced an earlier AI-origin heuristic with the concrete AI security checks above.
- Compliance evidence now renders consistently in the PR comment and step summary.

### Fixed

- Further false-positive reductions across multiple languages.

---

## [1.2.6] — 2026-06-10

A precision release for ReDoS (catastrophic-backtracking regex) detection — now caught on the inputs that actually matter, without new false positives. Pin it with `image-pin: v1.2.6`, or stay on `image-pin: 1` (re-pointed to v1.2.6).

### Fixed

- **ReDoS (CWE-1333) now catches the real attacker-input patterns** it previously missed (e.g. webhook/event payloads, log and field values, method parameters), while patterns applied only to constants or configuration stay silent and hardened (non-backtracking) patterns are recognised as safe — so the zero-false-positive bar holds.
- **No false "missing permissions" finding** on a non-workflow YAML file that happens to live under `.github/workflows/`.

---

## [1.2.5] — 2026-06-10

A security-depth release for Java. PullGuard now catches two classes of real vulnerability that a leading free SAST tool flagged but earlier versions missed — at PullGuard's zero-false-positive bar. Pin it with `image-pin: v1.2.5`, or stay on `image-pin: 1` (re-pointed to v1.2.5).

### Added — security depth

- **Insecure temporary files (CWE-377)** — flags temp-file creation that lands in a world-writable shared location (a time-of-check/time-of-use race); calls that target an explicit private location are treated as safe.
- **ReDoS on attacker-controlled input (CWE-1333)** — flags backtracking-prone patterns that actually run on untrusted data; patterns on configuration values and non-backtracking patterns are deliberately never flagged.
- **Broader Java coverage** of request inputs and risky sinks; committed credential defaults in secret-marked fields are flagged.

### Fixed

- **More precise ReDoS detection** — fewer false positives on non-vulnerable, regex-like code; genuine catastrophic-backtracking patterns are still caught.
- **Dependency-CVE findings carry structured version data** (package, matched version, fixed versions), so a finding that changes between scans is explainable from the report.
- **Workflow findings** that group several different findings now list each on its own line.

---

## [1.2.4] — 2026-06-09

A precision release for Java codebases: substantially fewer false positives, broader genuine coverage, and clearer findings — so the results on your PRs hold up to scrutiny. Every change ships with regression tests and was validated with no loss of genuine findings. Pin it with `image-pin: v1.2.4`, or stay on `image-pin: 1` (re-pointed to v1.2.4) to pick it up on your next run.

### Fixed — fewer false positives

- **Java is much quieter and more accurate** — a batch of false positives across logging, comparison, and configuration patterns no longer fire on idiomatic code.
- **Cleaner output** — clearer finding labels, consistent occurrence counts, and descriptions truncated at word boundaries.

### Added

- **Detects hardcoded developer/home paths** (e.g. `/Users/<name>/…`) that leak a username and break portability — CI and system paths are excluded.
- **Broader genuine coverage** of request inputs and sinks on Java stacks.
- **Workflow scanning** now covers every job in a multi-job workflow, plus shared-secret and clone-then-execute supply-chain patterns.

### Docs

- Documented `.driftrc.yml` configuration and the `.pullguardignore` per-finding suppression file.

---

## [1.2.3] — 2026-06-05

A signal-quality release: substantially fewer false positives, several closed false negatives, and more accurate dependency-CVE matching — so the findings on your PRs are actionable as-is. Every change ships with regression tests and was validated against real vulnerable applications with **no loss of genuine findings**. Pin it with `image-pin: v1.2.3`, or stay on `image-pin: 1` (re-pointed to v1.2.3) to pick it up on your next run.

### Fixed — fewer false positives

- **Findings now point at the exact file and line.** Findings that previously had no usable location (or pointed at the wrong file) are now accurately located, and SARIF uploads to GitHub Code Scanning always carry a valid location.
- **Infrastructure config no longer false-flagged as hardcoded secrets** — environment references, placeholders, and generated/resolved cloud secrets are no longer reported. Real committed secrets — including in comments — still flag.
- **Idiomatic template / redirect / reflection code is no longer flagged as critical** — these are only reported when attacker-controlled input actually reaches them, not on routine framework code.
- **Mentions of risky APIs inside comments / documentation are no longer flagged** (only real code is).

### Fixed — closed security false negatives

- **Spring & JAX-RS controllers are now fully covered** — request parameters flowing into injection sinks are now detected, where many were previously missed.
- **Broader sink coverage** across Java, Python, JS/TS, PHP, and .NET — each only when untrusted input reaches it.
- **Path-traversal, log-injection, and additional handler flows** that were previously missed are now detected.

### Fixed — more accurate CVE matching

- **Version comparison rewritten** to handle release qualifiers and pre-releases across ecosystems — fixing both false matches against already-patched versions and missed pre-release vulnerabilities.
- **Build-managed dependencies resolve their real version** instead of being flagged for every CVE for the package.

### Changed — clearer reports

- **Multiple findings of the same type in one file now show a count and the affected lines** on every report surface, instead of collapsing to a single under-reported row.
- **If an analyzer times out, the report now says so explicitly** rather than silently omitting its findings — so an incomplete check is never mistaken for a clean result.
- SARIF reports the real scanner version, and broad exception handling is described and graded accurately.

### Changed — honest claims

- Enterprise SSO / audit-log / RBAC are labelled as roadmap / contracted deliverables rather than shipping features; analyzer counts and compliance-scope wording were corrected to match what the scanner does.

### Privacy & transparency

- The privacy policy now discloses our processing sub-processors (Cloudflare, Resend, GitHub) and gives accurate per-endpoint data-retention details.

---

## [1.2.2] — 2026-06-04

Two efforts in one image: **zero-day rule precision** and a **signal-quality false-positive / false-negative batch** from an enterprise pre-review audit. The net effect on your scans is less noise and a couple of genuine issues that were previously under-reported. Pin it with `image-pin: v1.2.2`, or stay on `image-pin: 1` (re-pointed to v1.2.2) for the improvements on your next run.

### Fixed — fewer false positives

- **Authentication entry points are no longer flagged "missing authentication."** A login / signup / SSO route is unauthenticated by design. The suppression is deliberately narrow: infra/ops endpoints and protected routes that merely contain a `password` segment still flag, because those can be genuine exposures.
- **Environment variables and system properties are no longer treated as attacker input** — they are operator-controlled configuration (matching CodeQL and Semgrep defaults), removing a class of false positives on config-driven code. Command-line args, stdin, and all HTTP request inputs remain untrusted, so genuine injection flows are unaffected.
- **Security findings on test / fixture / mock / story files are suppressed** — that code never runs in production. Deliberately narrow: it does not touch real runnable code, and committed secrets are still flagged everywhere.
- **Frontend client wrappers are no longer mistaken for unauthenticated server routes.**
- **Optional-chaining and nullish-coalescing no longer inflate complexity findings**; genuine branches still count.
- **Clone detection skips test / fixture / migration boilerplate.**

### Fixed — closed security false negatives

- **CVE severity is no longer silently downgraded** when the vulnerability feed returns a severity vector instead of a numeric score.
- **CI/CD script-injection is now detected** in privileged inline-script contexts, not just shell steps.
- **Zero-day precision (Spring):** request-bound controller parameters are now tracked as untrusted, closing several false negatives on textbook Spring controllers.

### Config surfaces that now work

- `.driftrc.yml` path-like excludes take effect (previously had to be written as globs).
- Per-analyzer severity override is applied.
- Duplication block-size threshold is honoured.

### Image tags

- `:v1.2.2`, `:1.2.2`, `:1.2`, `:1`, and `:latest` re-pointed to the new digest
  (`sha256:d79455c9807daff6c9856178e48626db733104edbcbd1303c45fdb15998bc183`).
- `:v1.2.1` and `:1.2.1` are retained at their existing digest
  (`sha256:485925d26447191520fd68b3e3f1df9c5fabcabd760fc235c38214ffd76a589f`)
  for anyone pinned.

---

## [1.2.1] — 2026-05-28

Signal-quality patch. A multi-repo rollout sweep across real customer-class codebases surfaced a cluster of false positives and one false negative, all closed structurally so the fixes hold across your codebase. Pin it with `image-pin: v1.2.1`, or stay on `image-pin: 1` (re-pointed to v1.2.1) for the precision wins on your next run.

### Fixed

- **Frontend HTTP-client files are no longer mistaken for unauthenticated server routes** — a genuine server file is now required before missing-auth checks run. A real route still flags; the frontend-with-API-client false-positive cluster is closed.
- **Embedded encoded data (base64 images / fonts / binary blobs) is no longer flagged as a credential.** Real keys outside an encoded blob still report.
- **UI labels and obvious placeholders are no longer flagged as hardcoded secrets** by the loose secret pattern. Provider-specific keys stay strict and are unaffected.
- **Static inline scripts are no longer flagged as command injection** — the dynamic-execution analyzers still cover real injection paths.
- **A command-execution API is no longer matched by the SQL-injection check** — the command-injection analyzer covers it correctly.
- **A cross-file false negative is closed** — detection is now deterministic per file, order-independent.
- **Security findings sort consistently** (critical → info) across the PR comment, step summary, and JSON output.

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

- **Security focus on AI-generated code.** When a security flaw appears in code that shows AI-generation signals, PullGuard raises a single prioritized "review this AI-written code" finding instead of separate findings. Standalone AI-generation findings are informational only.
- **AI-governance evidence.** Each scan reports the share of analyzed code that shows AI-generation signals, as advisory evidence toward EU AI Act and NIST AI RMF programs — evidence *toward* governance, not a grant of compliance.
- **AI application-security coverage (OWASP LLM & Agentic Top-10)** — analysis now follows untrusted data into AI and agent features, across hosted, private, and self-hosted models, and treats model output as untrusted input.
- **Exploit-aware vulnerability prioritization.** Known-CVE findings carry an EPSS exploit-probability score and a CISA KEV "actively exploited" flag, so the vulnerabilities most likely to be attacked rise to the top. The actively-exploited feed refreshes continuously on PullGuard's side — your scans pick up newly-flagged CVEs without changing your pinned image.

### Changed

- **Authorization guards are recognized, not just authentication** — routes protected by an authorization layer are no longer reported as missing authentication.
- **Vendored, generated, and minified third-party code is excluded from source-code security findings** (it is still scanned for committed secrets).

### Fixed

- **More committed-credential files are detected** — including key and credential files that are not a scanned source type.
- **Cross-file findings point at a real, clickable line.**
- **Fewer false "orphaned file" findings** for browser-served assets and modules wired together at runtime.

---

## [1.1.2] — 2026-05-22

Patch release. Pin it with `image-pin: v1.1.2`.

### Fixed

- **False positive in unused-export detection on Python projects** — a symbol exported by one module and imported elsewhere via an absolute import could be flagged "not imported." Absolute imports now resolve correctly; genuinely-unused exports are still reported.

---

## [1.1.1] — 2026-05-22

Patch release. Pin it with `image-pin: v1.1.1`.

### Fixed

- **SOC 2 compliance evidence now renders correctly** — a rendering issue could cause the SOC 2 section to show "0/0" controls; it now displays the full SOC 2 control status as intended. The other four frameworks were unaffected.

---

## [1.1.0] — 2026-05-22

Compliance evidence now renders for all five frameworks on every PR, the `@v1` default moves to the stable release channel, and an authorization fix on the optional Check Run integration. Pin it with `image-pin: v1.1.0`.

### Added

- **Compliance evidence for all five frameworks on every PR.** Alongside SOC 2, each PR shows a compact PASS / CONCERN / FAIL summary for HIPAA Technical Safeguards, PCI DSS 4.0, NIST 800-53 Rev 5, and ISO 27001:2022. Turn on full per-control evidence tables for any of the four in your `.driftrc.yml`. PullGuard provides evidence *toward* these frameworks — it does not grant compliance.

### Changed

- **`@v1` now defaults to the stable release line, not the bleeding edge.** Without setting `image-pin`, PullGuard runs the newest **stable release** — reproducible between runs, still picking up each new release automatically. Set `image-pin: latest` to keep always-newest, or pin an exact version / digest to freeze it.

### Fixed

- Hardened authorization on the optional GitHub App Check Run integration — unlimited-repo (Enterprise) tokens are now scoped to their own organization.
- Documentation accuracy: the privacy policy now describes the single license-validation request made during a scan (your license key + repository name only — never your source code or scan results), and the cost-of-change formula and per-tier analyzer counts now match the product.

---

## [1.0.0] — 2026-05-20 (Spring 2026 release)

PullGuard's first pinnable release. 43 analyzers across security, code quality, compliance, and supply-chain; five compliance frameworks (SOC 2, HIPAA, PCI DSS 4.0, NIST 800-53 Rev 5, ISO 27001:2022); endpoint-risk prioritisation; and actionable-vs-observational cost reporting. Pin it with `image-pin: v1.0.0`.

### Added

- **11 new security analyzers** — coverage now includes CSRF protection, cookie security flags, JWT confusion attacks, HTTP security headers, server-side template injection, unsafe reflection, file-upload validation, cryptographic hygiene, generic injection sinks, Kubernetes IaC security (CIS Level 1), and missing-authentication detection on framework routes (Express / NestJS / Spring / Django / Flask / Rails).
- **Endpoint Risk Engine** — composes per-endpoint authentication state, data-flow, and reachability into a unified risk tier. An unauthenticated endpoint reachable from untrusted input becomes a **critical** finding rather than three disconnected medium-severity findings.
- **Compliance framework expansion** — added HIPAA Technical Safeguards (45 CFR §164.312), PCI DSS 4.0, NIST 800-53 Rev 5, and ISO 27001:2022 evidence mappings alongside SOC 2. SOC 2 itself expanded from 4 to 8 in-scope controls.
- **`.pullguardignore` suppression file** with PR-comment workflow — comment `/pullguard ignore <rule-id>` on a pull request and PullGuard opens a suggested edit. Security-category analyzers cannot be suppressed by this workflow.
- **Cost partition (actionable vs observational)** — fix-cost is reported as **actionable** (Moderate severity and above) versus **observations** (Info / Minor), so headlines reflect what's worth signing off on.
- **`image-pin` action input** — pin the scanner to an immutable release tag or content digest instead of the rolling `:latest`. Recommended for enterprise change-control and reproducible builds.

### Changed

- **Tier analyzer counts**: Free **14** (unchanged), Pro **31 → 42**, Team & Enterprise **32 → 43**. License keys auto-pick up the new analyzers; no customer action required.
- **PR comment + Step Summary headers** — display `N actionable findings (+M observations) · $X actionable`. Sections are collapsible; full descriptions render inline.
- **Per-language file-size thresholds** — Java tolerates 1000-line files (idiomatic for content-management codebases), Go 400, TypeScript 600, etc., replacing a single global threshold.
- **Honest line counts** — file-size and function-length checks no longer count comment-only or blank lines (aligned with the industry-standard NCLOC metric).
- **GitHub Actions Marketplace listing** — analyzer count, pricing-tier descriptions, and capability claims updated across all customer-facing surfaces.

### Fixed

- **CI reliability** on macOS runners for cross-repo verification.
- **False positives on documented code** — typical web-framework codebases see roughly 30 spurious "monolithic file / function" findings removed per repository as a result of the line-count change above.
- **PR comment rendering** — descriptions no longer truncated mid-word; overflow findings are reachable via collapsible sections.

### Compliance

- **Multi-framework evidence sections** appear in the Step Summary + PR comment when the relevant analyzers run. Each control maps to the analyzers that produce evidence for it; the report shows PASS / CONCERN / FAIL per control with the underlying violation count.

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
