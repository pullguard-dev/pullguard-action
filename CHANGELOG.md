# PullGuard changelog

All notable customer-visible changes. Earlier releases predate this
changelog; this file is the canonical record going forward.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Live release notes for the hosted scanner: [pullguard.dev](https://www.pullguard.dev).

## [Unreleased]

_Customer-visible changes already live on `:latest` but not yet bundled into a cut image tag. Pin a specific release below for change-controlled, reproducible scans._

---

## [1.5.3] — 2026-08-09

AI-threat catch-up release. No breaking changes; new detections are additive and reuse existing finding types. Verified with **no loss of true-positive coverage**.

### Added
- **Comment-injection detection in source code.** PullGuard now inspects source-code comments — the context AI coding assistants read — for two attack classes seen in the wild in 2026: invisible Unicode control characters hidden in comments (the Trojan Source class, and the vector used to smuggle instructions past human review), and comment blocks addressed to AI assistants carrying override, concealment, or exfiltration directives (the vector used to steer coding assistants into leaking repository tokens). Detection is comment-aware — it distinguishes code from comments and is tuned to reduce false positives on ordinary comments that merely mention AI tools — so findings reflect override, concealment, or exfiltration directives rather than incidental mentions. Calibrated against two dozen real open-source repositories, including AI frameworks, agent applications, and MCP servers, before release.
- **OWASP Top 10 for Agentic Applications (ASI) categorization.** Agentic-risk findings — prompt injection, agent instruction and tool poisoning, MCP configuration risks, over-privileged agent tools, unbounded agent loops, and related detections — now carry their OWASP ASI category in SARIF output, both as a per-finding property and as rule tags. They can be filtered and reported against the published agentic-AI risk taxonomy in GitHub code scanning and other SARIF consumers. Every category identifier and title is verified against the published OWASP list; findings outside the taxonomy carry no tag.

### Fixed
- **Report source links no longer degrade on ownership-mismatched mounts.** When a repository is mounted with different file ownership than the container user, the entrypoint's own git operations — which resolve the scan reference used for report source links — could still be refused, producing a spurious "dubious ownership" warning after an otherwise successful scan. Those operations now run with the same safe-directory configuration as the scan itself, scoped to the exact path and never a wildcard. Any `GIT_CONFIG_*` environment entries you provide are always preserved.

## [1.5.2] — 2026-08-08

Hardening and AI-threat response release. No breaking changes; new detections are additive and reuse existing finding types. Verified with **no loss of true-positive coverage**.

### Added
- **Detection of malicious agent hooks and auto-run editor tasks.** PullGuard now inspects hook commands committed in agent settings files, and editor tasks configured to run automatically on folder open — the file surfaces weaponized by the August 2026 self-propagating npm-worm campaign to execute credential-stealing payloads the moment a repository is opened, with no install step and no script file to review. Flags download-and-execute, credential-read, and encoded or inline script payloads; legitimate hooks and ordinary build tasks are unaffected.
- **Detection of invisible Unicode Tag-block characters** in agent instruction files — the "invisible instruction" vector recently found in malicious published agent skills.
- **Dependency CVEs are now resolved against `yarn.lock` and `pnpm-lock.yaml`** (all current lockfile versions), extending the lockfile-accuracy fix beyond npm: no more false criticals on already-patched dependencies, and vulnerable resolutions under a clean version range are caught — across the whole JS package-manager ecosystem.
- **GitHub Security tab severity ranking.** SARIF output now carries the `security-severity` property on security rules, so findings rank Critical/High/Medium/Low in GitHub code scanning instead of landing in generic level buckets.

### Fixed
- **Git-backed analysis no longer degrades silently in containers.** On a mounted checkout owned by a different user, git refused to run and history-based analyzers (bus factor, secret history, blame, source links) quietly produced nothing. The container now marks the scanned checkout as a git safe.directory — scoped to that exact path, never a wildcard — and any remaining git refusal is reported loudly with the fix instead of vanishing.
- **Self-hosted server SSO docs corrected to the actual (stricter) behaviour:** an authenticated user with no mapped role is denied by default in every configuration; granting a fallback role always requires an explicit opt-in.

## [1.5.1] — 2026-08-07

Precision and triage-usability release. No breaking changes; new finding types and configuration are additive. Verified with **no loss of true-positive coverage**.

### Added
- **Dev-tooling tiering (opt-in).** Declare build/preview/test-container paths in `.driftrc.yml` (`devTooling.paths`) and findings there are down-ranked exactly one severity step — annotated with their original severity, never dropped from any surface. Committed credentials and AI-agent attack findings are exempt and keep full severity. Off by default; nothing is inferred from path names.
- **Self-hosted server: triage without an identity provider (opt-in)** via `PULLGUARD_SERVER_OPEN_TRIAGE=true`. Decisions remain audit-logged (recorded as unauthenticated); the server refuses to start if the flag is combined with SSO, where role-based access governs instead.
- **Self-hosted server: dashboards link the triage page**, so triage is reachable from every dashboard.
- **Detection of backdoored MCP tool commands** in committed agent configuration.

### Changed
- **Light is the default theme** for reports and dashboards (dark remains one click away and persisted).
- **Polynomial ReDoS is now reported at `minor`** with an explicit O(n²) description; catastrophic-backtracking shapes remain `major`. Nothing is dropped.
- **Dashboards:** the findings table is sortable, truncated lists are expandable, and the first sort click orders worst-first.

### Fixed
- **Dependency CVEs are evaluated against the lockfile-resolved version**, not the declared range floor — removing false criticals on already-patched dependencies and catching a vulnerable resolution under a clean floor.
- **`dangerous_file_tracked` no longer fires on files git ignores** when scanning a repository subdirectory.
- **Security-check precision**, from a customer evaluation of v1.5.0: framework query-builder injection recall across all chain verbs with per-argument safe-idiom analysis; structured-comparator builders (JPA Criteria, jOOQ, QueryDSL) no longer flagged on bound parameters; bound-parameter `createQuery` calls no longer flagged; browser-side fetches no longer misreported as SSRF; password-confirmation compares no longer flagged as timing attacks; locale/i18n files no longer flagged for secrets; context-gated `insecure_crypto`; bundler-output scoping; and scan determinism pinned at the licensed tier.
- **Self-hosted server: the triage page shows the assigned quality-gate verdict** (named, with the same condition details CI receives); without an assigned gate the banner is labelled "Critical gate".
- **Quality gates: clearer verdicts on uploads without a baseline** — the fail-closed behaviour is unchanged, but a failing `new_*` condition now says every finding counts as new and recommends `total_*` metrics for full-inventory uploads.
- **Self-hosted server: pages show the real server version in the footer and serve a favicon.**

## [1.5.0] — 2026-07-24

AI-agent security release. Adds a dedicated analyzer for AI-agent and MCP configuration, a wave of new AI-agent security checks, and substantially wider prompt-injection coverage — alongside coverage, precision, and compliance improvements. No breaking changes; no change to the scanner output format. New finding types are additive (existing types unchanged). Verified with **no loss of true-positive coverage**.

### Added
- **New analyzer for AI-agent and MCP configuration** (Pro and above). PullGuard now inspects the files that drive coding agents and Model Context Protocol servers — agent instruction files, MCP server definitions, editor/agent rule files, and agent hook scripts — for hidden-instruction attacks, backdoored tool definitions, and instruction injection a human reviewer would not see.
- **Detection of over-privileged agent tools** — a tool exposed to a model that passes its model-chosen argument straight into a shell, database, filesystem, or network call with no validation.
- **Detection of disabled agent human-approval gates**, unbounded agent run budgets, and agent configuration "rug-pulls" (an agent config that changes after a trusted baseline).
- **Detection of unauthenticated MCP servers** exposed on the network without an authentication layer, and of MCP tool descriptions crafted to inject instructions.
- **Detection of retrieved-context injection (RAG)** — when a project ingests attacker-influenceable documents and feeds retrieved text into a model prompt without isolating it.
- **Much wider prompt-injection coverage** — tracing now follows user input across function boundaries and recognises many more AI entry points (agent frameworks, retrieval/agent engines, and hand-rolled calls to model gateways) in more languages.
- **AI supply-chain checks** — known-CVE fixtures for the AI stack and detection of typosquatted / hallucinated AI package names.
- **AI usage visibility** now recognises hand-rolled model clients that call provider REST endpoints directly, so a backend that talks to a model without an official SDK no longer reports as having no AI usage.
- **Per-finding security false-positive override.** A security finding a team marks as a false positive stays visible, reason-required, and reviewable by code owners — but no longer permanently blocks the pull request. It is recorded and audited, never silently dropped.

### Fixed
- **Reliable large-repository coverage.** Large monorepo scans now report exactly how much of the repository was analyzed and state clearly when a scan was truncated, instead of presenting a partial scan as complete. Committed configuration and credential files (such as `.env` and agent rule files) are now read and analyzed. Use `maxFilesRead` in `.driftrc.yml` to raise the analysis cap.
- **Zero-day threat rules restored.** The bundled zero-day threat rules (Log4Shell, Text4Shell, SpEL, OGNL, and others) fire correctly again after a regression.
- **SQL-injection precision.** The pattern-based SQL-injection check no longer fires on provably safe query construction (constant table names, validated integers, escaped or parameter-bound values), while still flagging genuine user-input concatenation.
- **A broad set of false-positive fixes** across taint precision, injection sinks, redirect and email-header checks, timing-attack detection, ReDoS shape analysis, and vendored-code handling.
- **Compliance mapping accuracy.** The new AI-agent security findings now contribute evidence to the EU AI Act, ISO/IEC 42001, and NIST AI RMF mappings.

### Changed
- Analyzer count is now 46 (Free 14 / Pro 44 / Enterprise 46).

---

## [1.4.6] — 2026-07-22

Coverage and accuracy release for AI-era security, plus a second pass on analyzer signal quality. No breaking changes; no change to the scanner output or finding types. Verified with **no loss of true-positive coverage** (OWASP Benchmark results unchanged).

### Added
- **Broader AI/LLM coverage.** Prompt-injection and sensitive-data-to-model detection now recognises substantially more of the AI SDK surface teams actually use today — current client and streaming call styles, structured-response APIs, composed chains, agent frameworks, managed cloud model services, and the mainstream Java AI stack. Code that was previously invisible to these checks is now covered.
- **More AI provider credentials recognised as secrets**, including service-account and administrative key formats and several additional model providers and gateways. These now also apply on the commit-history, container, and workflow scanning surfaces.
- **Wider AI usage visibility.** The AI inventory and AI bill-of-materials recognise additional providers, routers, and OpenAI-compatible gateways — a project built on these no longer reports as having no AI usage.
- **Detection of unsafe model loading** extended to more of the Python serialization ecosystem commonly used to distribute models.

### Fixed
- **Correct destination reporting for AI gateways.** Model calls routed through an OpenAI-compatible gateway are attributed to the actual provider rather than to OpenAI, so the AI bill-of-materials names the real data-egress destination.
- **Compliance control accuracy.** AI-governance control titles now match the published standards exactly, the AI security mapping points at the correct framework subcategory, and additional finding types — including hallucinated/typosquatted dependencies and disabled transport security — now contribute compliance evidence. The compliance summary also reports the correct framework count when AI frameworks are enabled.
- **Safety-filter opt-outs detected in more forms**, and agent iteration caps set to effectively-unlimited values are now flagged.
- **Composite AI risk no longer over-escalates.** A composed AI-plus-security finding always carries the severity of its most severe underlying finding, so a finding your configuration has downgraded is never re-raised.
- **Data-egress reporting is explicit when unavailable.** If the accelerated analysis path is unavailable, the AI usage panel states that data-egress risk was not assessed instead of showing zero.
- **Less structural noise.** Architecture and complexity checks no longer judge end-to-end test scaffolding as shipped code, framework model boilerplate folds more completely, and the consolidated endpoint-risk finding no longer restates a single already-reported issue.

### Changed
- The AI bill-of-materials carries standard component identifiers, a document serial number, and a creation timestamp for better interoperability with BOM tooling.

## [1.4.5] — 2026-07-17

Precision and signal-quality release, plus the self-hosted control plane. No breaking changes; no change to the scanner output or finding types. Verified against the OWASP Benchmark and a suite of deliberately-vulnerable applications with **no loss of true-positive coverage**.

### Fixed
- **Much less structural noise on large real-world codebases.** Parallel data-class / model boilerplate now folds into a single observation instead of one finding per file; the structural analyzers (architecture, complexity, large-file, naming) no longer judge test scaffolding as if it were shipped code; and component-library / styleguide files are no longer reported as unused. Security analysis is unchanged — security findings in test and styleguide code still surface.
- **Sharper framework query-builder analysis.** Injection detection for Java CMS / ORM query builders is more precise on the safe parameter-bound idioms those frameworks use throughout, so correctly-written queries no longer produce findings.
- The insecure-temp-file explanation is corrected (a shared-temp file is a readability/confidentiality issue, not a create-time race), and repository "bus factor" is no longer reported for squash-imported histories that have no real timeline.

### Added
- **Self-hosted control plane (Enterprise).** The customer-hosted server adds server-administered quality-gate policy — defined centrally and enforced in CI as a required status check — GitHub / GitHub Enterprise Server sign-in with team-based access control, and an AWS ECS (Fargate) deployment option alongside the existing Helm chart.

---

## [1.4.4] — 2026-07-15

Reliability, precision, and hardening release. No breaking changes; no change to the scanner output or finding types. Verified against the OWASP Benchmark and a suite of deliberately-vulnerable applications with **no loss of true-positive coverage**.

### Changed
- **Baseline mode keeps pre-existing security-critical findings visible.** `scan --baseline` now surfaces pre-existing security criticals (marked as pre-existing) and still enforces them at the gate, instead of letting a baseline hide them from view. Everyday non-security noise stays collapsed.
- **More reliable incremental scanning.** A `--cache` run now consistently reflects the current state of your repository, so a cached scan matches a fresh scan of the same tree.

### Fixed
- **Fewer false positives** across command-injection and secret detection, tuned to stay quiet on safe code patterns, with no reduction in true-positive coverage.

### Security
- **Self-hosted server hardening (Enterprise).** The customer-hosted server image adds security response headers, a non-root renderers-only runtime, request-size and repository operation caps, and store-aware readiness probes.

## [1.4.3] — 2026-07-10

### Added
- **Broader AI-agent & Go security coverage.** Data-flow analysis now follows untrusted input into MCP (Model Context Protocol) tool calls and Go's context-aware APIs (`exec.CommandContext`, the `db.*Context` query methods) and `ioutil` file APIs — closing detection gaps on modern agent and Go codebases (taint-gated; no new false positives).
- **Self-hosted server — full-inventory visibility.** Repos behind a baseline now report their complete finding inventory to the server dashboard (previously such a repo could appear "clean"); each finding is tagged pre-existing vs. actionable. Scan history now records branch and commit.

### Fixed
- **AWS ARN false positive.** AWS Secrets Manager ARNs — resource references, not credential values — are no longer mis-flagged as hardcoded secrets in YAML/IaC config.

## [1.4.2] — 2026-07-09

Signal-quality, honesty, and integration release. No breaking changes.

### Added
- **Embed PullGuard in your own dashboards (Enterprise).** The self-hosted server now exposes a read-only JSON API so you can surface repo grades, security posture, compliance status, and finding trends inside your own control-plane UI — read-only token + CORS allowlist; results only, never source.
- **Cloudflare Wrangler hardening check.** Flags a Worker still exposed on its default `*.workers.dev` URL (bypassing your zone WAF / rate-limiting).
- **Free-tier coverage note.** Free scans now state how many analyzers ran, so a free-tier "Security: A" is never mistaken for a full security pass.

### Changed
- **Security analyzers can no longer be switched off in configuration** — the integrity guarantee is now enforced.

### Fixed
- Parameterized SQL queries and plain shell `exec()` calls are no longer mis-flagged as SQL injection.
- Native Check Runs are restored and now attach inline annotations to the Files-changed tab.
- PR-comment / Check-Run polish (pluralization, trend-arrow direction, actionable-cost headline) and a corrected `.pullguardignore` + air-gapped setup doc.

## [1.4.1] — 2026-07-08

**A verifiable supply chain.** Primarily a supply-chain release — no change to how your code is scanned.

### Supply chain
- **Published images are now cryptographically signed.** Both the scanner and the self-hosted server images carry a keyless **cosign / Sigstore** signature recorded in the public transparency log — so teams running signed-images-only admission policies can admit them, and anyone can verify authenticity independently of the registry with `cosign verify`. This builds on the SLSA build provenance and SBOM already shipped in 1.4.0.

### Compliance
- **AI-governance evidence now works on standard online scans**, not just air-gapped runs. The opt-in EU AI Act, ISO/IEC 42001, and NIST AI RMF mappings are delivered to every scan. Still opt-in per framework, and still evidence toward an obligation rather than a certification.

### Enterprise (self-hosted server)
- **License-gated direct download for the server image.** Enterprise customers can fetch the self-hosted server image over an air-gapped-friendly HTTPS endpoint authenticated with their existing license key, with an integrity checksum to verify the download.

## [1.4.0] — 2026-07-07

**AI-era security & governance.** The biggest PullGuard release yet — see and govern everything AI touches in your codebase, with broader infrastructure coverage, sharper findings, and a verifiable supply chain. (This release also includes the security-hardening and Enterprise improvements previously previewed as 1.3.4, which was never published as its own image.)

### Added
- **AI usage inventory ("Shadow-AI map") — Enterprise.** A new view lists every external AI provider your code calls — which providers, which models, and where — and flags any call site that sends secrets or personal data to a model. Answers "how does AI touch our codebase?" Visibility only; it never affects your grade.
- **AI Bill of Materials — Enterprise.** Export a standard CycloneDX ML-BOM of the AI in your codebase (every model and provider, with call-site evidence and a data-egress flag) for procurement and AI-governance requests. Works fully offline.
- **AI-governance compliance evidence (opt-in).** Map your findings to **EU AI Act, ISO/IEC 42001, and NIST AI RMF** to produce audit evidence — off by default, enable only the frameworks your scope needs. Evidence toward each obligation, not a compliance certification.
- **AI provider key detection.** Leaked AI-provider API keys (OpenAI, Anthropic, Hugging Face, Google, and more) are now caught as secrets — direct billing and data-egress exposure.
- **Broader infrastructure & insecure-default coverage.** New checks for world-writable file permissions, over-permissive cloud IAM grants, world-open network ingress, and additional permissive-CORS patterns — each tuned to stay quiet on the safe, idiomatic forms.

### Changed
- **git-blame tells you when it can't work.** With finding age/owner enabled on a shallow checkout, PullGuard now warns you to deepen the checkout instead of silently leaving findings undated.

### Fixed
- **Sharper, better-labelled findings.** A class of non-credential issues (infrastructure, smart-contract, and query risks) that used to be reported as "hardcoded secret" now carry their correct category, so filtering and triage by type are accurate. Infrastructure files in nested `infra/`/`deployment/` folders are no longer skipped, and several false-positive sources were removed.

### Supply chain
- **The scanner image is now verifiable.** Every published image ships **SLSA build provenance and an SBOM** — your security team can confirm how it was built and what's inside before running it (`docker buildx imagetools inspect`).

### Enterprise (self-hosted server)
- **Get the server image your way.** Private-registry install is fully documented, with an air-gapped mirror flow and a **license-gated direct download** for environments that can't use a container registry — each image carries provenance and an SBOM.
- **Repository-band licensing** and **dashboard auth hardening** — a sign-in audit trail, brute-force protection, stronger SAML validation, and file-mounted secrets.

### Security
- A broad internal security-hardening pass across the scanner, licensing, and self-hosted server. No change to normal scans.

### Notes
- **Anonymous free-tier usage signal (opt-out).** Free scans in GitHub Actions send an anonymous adoption count — no code, no findings, no personal data. Opt out with `telemetry: false` (or `PULLGUARD_TELEMETRY=off` / `DO_NOT_TRACK=1`). Paid, offline, and air-gapped scans never send anything.

---

## [1.3.3] — 2026-06-24

**Self-maintaining baselines, finding ownership, and editor integration.**

### Added
- **Self-maintaining baseline** — a new Action input lets the scan that already runs on your base branch write and refresh your baseline itself, so your "new findings only" view stays accurate without a separate baseline step to keep in sync.
- **Finding age & owner** — opt in to show each finding's "introduced" date and author (from your git history) in the HTML report and dashboard.
- **Single sign-on for the self-hosted dashboard (Enterprise)** — log in to your self-hosted PullGuard dashboard with your identity provider (OIDC or SAML) and gate access by role. Your code and findings stay inside your own boundary.
- **PullGuard for VS Code** — see your findings inline in the editor (Problems panel, quick-fixes, grade status bar), rendered from your scan report — no scanner runs in the editor.

---

## [1.3.2] — 2026-06-24

**Baseline accuracy.** Your PR "new findings only" view now reflects your plan's full analyzer set, so pre-existing work is correctly suppressed on licensed scans.

### Added
- **Tidy pre-existing security in PR comments** — an opt-in setting collapses security findings that already exist in your baseline into a separate, counted section, so a PR's inline list focuses on what that PR introduced. New security findings still appear inline, and every security finding stays in the full report and the build gate — nothing is hidden.
- **Send results to a self-hosted dashboard (Enterprise)** — new Action inputs let your pipeline post scan results to a PullGuard server you run inside your own boundary. Results only — your source never leaves your runners.

### Fixed
- **Baselines now reflect your plan's analyzers.** Generate your baseline with the same Action/image you scan with so "new findings only" works correctly on licensed scans.

### Notes
- Security findings always appear on a PR by design — the delta hides resolved-elsewhere work, never security.

---

## [1.3.1] — 2026-06-23

**Reporting, dashboards & visibility.** Turn scan results into something every role can use — all **air-gapped**: single self-contained HTML files, no network.

### Added
- **Self-contained HTML report** — findings with a severity breakdown, fix-cost, finding dates, and **clickable links straight to the offending file and line** on your git host.
- **Over-time dashboard** — score / findings / fix-cost / AI-era-risk trends, with **drill-down to a scan's findings and their exact file:line**, finding owner & age, exploit priority, a **branch / PR view selector**, and **SOC 2 posture over time**.
- **Portfolio view** across every repository PullGuard runs on.
- **Light / dark theme** toggle.
- **Finding date-stamps** ("first discovered" + introduced date/author), **triage** (acknowledge a finding without suppressing it), **baselines beyond PRs** (only-new-findings on pushes/schedules), and an **SLA / aging build gate**.
- Report + dashboard are **rendered and uploaded as CI artifacts** on every run.

### Fixed
- Java false positive where large files could report "0 functions".
- Duplication percentage could exceed 100%; boilerplate data-class duplication is now treated as an observation rather than actionable cost.
- `--version` now reports the real release version.

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
