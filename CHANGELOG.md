# PullGuard changelog

All notable customer-visible changes. Earlier releases predate this
changelog; this file is the canonical record going forward.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Live release notes for the hosted scanner: [pullguard.dev](https://www.pullguard.dev).

## [Unreleased]

_Customer-visible changes already live on `:latest` but not yet bundled into a cut image tag. Pin a specific release below for reproducible scans._

---

## [1.5.18] — 2026-10-06

> **Ships with server 0.4.5** (self-hosted Enterprise server). The Action's `image-pin` default rides the `:1` image tag, so `@v1` users get the new scanner on their next scan. One Action input change: the runner's proxy variables (`HTTPS_PROXY` and similar) are now passed into the scan container.

### Request input passed from an inline route handler to a helper
- **Request input handed from an inline route handler to a helper function is now reported.** Arrow functions, `async` arrows and `function` expressions are followed the same way as named handlers, with the helper in the same file or imported from another. A handler that passes the helper a constant is not reported. Expect new findings on Express-style applications that route through inline handlers.

### A directory name no longer hides code the application imports
- **Code under a test-looking directory that the application imports is scanned as application code.** Findings were previously not reported for files under `mocks/`, `fixtures/`, `__mocks__/`, `testdata/`, `e2e/`, `cypress/` or `storybook/`. When a file that ships imports such a file, its findings are now reported and each names the importing file. Code imported only by tests, a test runner's configuration or an example is still treated as test code. Expect new findings where application code imports from such a directory.

### Request input through call and callee spellings
- **A flow through a helper in another file is graded like the same flow in one file.** Request input reaching a command-execution or deserialization sink through an imported helper was `major`; it is now `critical`, as it already was within one file. No finding is graded lower than before, and a scan that gates on critical findings will now fail on these.
- **Call spellings are followed across files.** `fn.call(…)`, `fn.apply(…)`, a `bind` alias of an imported function, a callback handed to an imported helper, a helper that reads `arguments[n]` and a helper that destructures its options parameter are now reported, with every line of the flow on the finding. JavaScript and TypeScript.
- **An HTTP-client call inside a helper is judged on its destination.** A request value that only fills the body or options of a request to a fixed destination is no longer reported as request forgery. When the helper takes the destination as a parameter, the finding keeps its grade and says the tainted argument does not reach the destination.
- **Values handed to inline callbacks are followed.** A helper that calls its callback with request data, or passes its own argument on to the callback, is reported when the callback's parameter reaches a sink. The "does not appear to reach this sink" note is now accurate when a parameter is copied into another one; the grade was already unchanged.

### Every pattern in a file is listed
- **Every line that matches the same pattern rule is listed.** Two weak hashes, or two string-built queries of the same shape, in one file used to show the first line only. Each matching line is now listed on the finding (up to 50 per rule per file), and the grade is unchanged.
- **A file with more than one kind of injection pattern lists each of them.** The SQL, command-injection, path-traversal, SSRF, prototype-pollution and deserialization checks reported only the first kind found in a file. The finding now takes the strongest grade among the listed lines; finding counts per file do not change, but the listed lines can.

### Weak ciphers and ECB mode
- **Weak ciphers and ECB mode are now reported by their API call in five languages.** This covers Node, Java (including `Cipher.getInstance("AES")`, which defaults to ECB), Python, Go and C#. Authenticated modes (GCM, ChaCha20-Poly1305) and text that only mentions an algorithm are not reported. Expect new `insecure_crypto` findings.
- **A weak hash no longer hides a weak cipher in the same file.** The weak-cryptography check reported only the first kind of weakness it found, so a file with an MD5 checksum and a DES or ECB cipher reported the checksum alone. Every kind found is now reported.

### Grades are not lowered by what a file says about itself (continued)
- **A JAX-RS route whose guard PullGuard only infers keeps its grade.** The finding names the inference so it can be checked, at the grade of an unguarded route. It was reported as minor.
- **A secret whose key names a development service keeps its grade.** A credential under a key naming a local mail or cloud emulator is reported at the rule's grade (critical) with a note saying so; it was graded major. If you relied on that lower grade, triage those findings as acknowledged.
- **A finding from an inline route handler names the line that reads the request**, not the helper's first line.

### Weak cryptography in entry-point files
- **Weak cryptography is now reported in entry-point and component files.** MD5 or SHA-1 password hashing, weak ciphers and similar were not reported in files named `server.js`, `index.js`, `app.js` or `main.py`, or under `components/` and `tools/`. The same applied to a secret that reaches an LLM prompt. Test files, fixtures, documentation, examples and scripts are still skipped. Expect new `insecure_crypto` findings.
- **Pattern rules run on a fixture-path file that the application imports**, matching the directory-name change above.

### Clearer findings from single-line rules
- **XXE findings are now reported as `xxe_vulnerability` (CWE-611) instead of command injection.** A Java XML parser hardened in the same file is named on the finding, and the grade is unchanged because another parser in the file could still be unhardened.
- **`yaml.load(..., Loader=SafeLoader)` is no longer reported** when the loader is exactly a safe loader. A bare `yaml.load`, any other loader or a conditional loader is still reported.
- **Some findings now state why they may be benign, at full grade.** This covers shell `eval "$(tool init -)"` bootstraps, Go `exec.Command` calls without a shell, and JSON-P readers. `eval` of a network fetch, and Go programs that hand an argument to a shell or interpreter, get no such note.

### Scans cannot be stalled by hostile input
- **A custom rule's `files` / `exclude` glob in `.driftrc.yml` can no longer stall a scan.** A pull request can change that file, and a glob with many wildcards could take exponential time on a long file name. Globs now match in time proportional to their length, with the same meaning.
- **Three checks that read scanned code no longer slow down sharply on crafted long lines.** Results are unchanged; a calibration proof facing a declaration line too long to read safely is reported as not run rather than guessed.
- **Large repositories and minified bundles scan faster, with the same results.** A check stopped at its time budget loses its findings, so this is a completeness fix too.

### C# output encoders in shared classes
- **A request value passed through an HTML encoder declared in another C# file is no longer reported as cross-site scripting.** The encoder's namespace must be imported by, or enclose, the calling file, and a helper that returns its input unchanged is still reported.
- **With `taint.calleePositionalProofs` on, the per-argument helper proof now covers C# and JavaScript too.** A call that passes a constant in a helper's returned position is no longer reported. Off by default, as before.
- **Java: a proven output encoder in another file is also recognised** when the call names its package or goes through a local variable of a `final` encoder class. A different package, or a class that could be subclassed, is still reported.
- **A value reassigned from a local that only ever holds constants is no longer reported as carrying request input.** Maven dependencies declared without a version under a Spring Boot parent or a Spring Boot / Quarkus BOM now take that platform version for vulnerability matching and the SBOM.

### Java SQL injection through chained appends
- **A value checked by a validator that throws on anything non-numeric is no longer reported when appended to a query.** A raw request value appended to the same query is still reported.
- **Request input added to a query with chained `StringBuilder.append(...).append(...)` calls is now detected.** Previously only the first call in the chain was read.

### Getting more from PullGuard
- **`pullguard doctor` now lists the capabilities your repository uses** (baseline, SARIF upload, reviewed suppressions, SLA budgets, ownership routing, compliance frameworks, test coverage) and links the ones it does not.
- **Documentation:** a new "Your first 30 days" guide on Getting Started, one page explaining agent access for local and server use, and every `scan` flag in the CLI reference. When your code calls an AI provider, the AI usage section mentions that AI-governance evidence tables are available as an opt-in.

### Scripts, migrations and examples are checked
- **Security pattern rules now run on code in `scripts/`, `bin/`, `migrations/`, `examples/` and `benchmark/` directories.** Test and fixture files are still excluded. An operational script that pipes a download into a shell, or a server started in debug mode on all interfaces, is now reported.
- **YAML under a nested `infra/` directory is now checked for hardcoded secrets,** and C and C++ test files (`*_test.cc`) are treated like other test files. No file gets fewer checks than in 1.5.17.

### Comments never add a finding
- **Comments no longer decide the missing-security-headers check, in either direction.** A comment quoting `const app = express()` was reported as an Express app without security headers, and a commented-out `app.use(helmet())` counted as configured, hiding a real gap.
- **A comment no longer produces a second finding** for hardcoded developer paths, insecure temp files, Java ReDoS, `@WebParameter` SQL concatenation, missing pagination, or cookie flags. Committed credentials in comments are still reported.

### Java CMS sort keys
- **Raw SQL run on a `SqlDatabase` held in a short variable (`db.selectFirst(sql)`) is now followed as a data flow,** including through a helper method that receives the SQL text. Previously only a concatenation on the same line was reported.
- **A request value used as a sort key now says why it matters and how to fix it.** A sort key exposes an ordering oracle over fields the caller may not read; the fix is an allow-list of sort fields. The grade is unchanged, so a severity gate blocks exactly what it blocked before.

### Which routes lead to a vulnerable dependency (more shapes)
- **NestJS controller routes, Express handlers declared in the same file, and routes that reach a package through a service layer are now attributed.** A route reached through an intermediate module says so. A value that is not a function is no longer read as a handler. Display only.

### Every matched advisory is listed
- **A vulnerable-dependency finding now lists every matched advisory id** in `vulnPackage.advisoryIds` (bounded at 100). Previously only the first five were listed and the rest appeared only as a count, hiding some from automation. The five-entry `advisories` detail is unchanged.

### Runners behind a proxy
- **PullGuard now uses your HTTP(S) proxy.** Licence validation, the signed rules bundle, advisory lookups and the results upload previously connected directly even when `HTTPS_PROXY` was set. The image, the Action and the command line now honour `HTTPS_PROXY`, `HTTP_PROXY` and `NO_PROXY`; set `NODE_USE_ENV_PROXY=0` to opt out.

### Engine
- **A Java validator that throws on a non-numeric value is recognised however the `throw` is laid out.** Recognition no longer depends on the file's line endings.
- **Faster callee analysis on Java.** Findings are unchanged.

### Ownership routing
- **The by-owner table now appears on the pull-request comment too** when `ownership.groupBy: owner` is set. It is collapsed, limited to the ten most urgent owners, and never uses `@`-mentions, so the comment notifies no one on each push.

### Grades
- **A category with a critical finding is never graded A.** The overall grade already could not be A with a critical present; a single category could. The category letter is now at most B in that case. No scores change.

### Scan coverage
- **A code file too large to read is now named, not silently skipped.** Code files over 500 KB are still not read, but the report records how many and which, and the PR comment and terminal name them. A scan-completeness attestation no longer certifies a scan that left a code file unread; oversize configuration and data files are recorded and do not affect it.

### New-code coverage gate (JaCoCo)
- **`coverage.newCodeMinimum` gates the lines a change adds or modifies.** Below the minimum, a `new_code_coverage_below_minimum` finding (major) is raised. A change that cannot be measured (no merge-base, an unreadable diff) is treated as not meeting the minimum, and changed source files the report does not cover are counted and shown.
- **A pull request cannot relax the gate that judges it.** Lowering or removing the minimum, or changing the report path, in its own `.driftrc.yml` is held to the base branch; delta mode, `.pullguardignore`, the scan cache and a committed baseline do not remove the finding. Produce the coverage report in a workflow the change cannot alter.
- **A pull request can no longer remove the settings that guard its own scan by deleting or renaming the configuration file.** Deleting `.driftrc.yml`, or adding a higher-precedence one beside a base branch's `.driftrc.yaml`, used to drop every guarded setting at once. The base branch's configuration is now found on its own and the change is held to it.

### Test coverage from JaCoCo (report-only)
- **PullGuard reads your JaCoCo report and states the line coverage.** Configure `coverage.reports`; the coverage appears in the report (`coverage`) and the PR comment. A report that cannot be used says why instead of showing a number, for example when it is outside the project, committed to the repository, too large, or XML with a DOCTYPE or ENTITY.
- **lcov and Cobertura reports are read too** (Jest, nyc, c8, coverage.py, coverlet, gcovr and others), so the new-code minimum works for TypeScript, Python, .NET and C/C++ projects. `coverage.format` defaults to `auto`; naming a format enforces it.

### Which routes lead to a vulnerable dependency
- **A dependency CVE now says which HTTP routes lead to the code that uses the package, and whether those routes are guarded.** It covers Express/NestJS, Spring, Django, Flask and Rails, and each route carries the missing-authorization analysis's own verdict. It is file-level, so a single route is named only when the file declares exactly one.
- **When no route connects, the finding says why** (package not imported, only tests import it, no cross-file resolution for the language yet, no route support for the framework, aliases not followed, or the search bound reached). It never says "unreachable", and the auth state is `unknown` when the route analysis did not run or finish.
- **The routes that actually reference the vulnerable package are named first,** with a count of the routes not attributed. A route that is not attributed is never presented as unaffected.
- **Java and Kotlin services, interfaces and controllers are connected, and Next.js, Go, path aliases and workspaces are covered.** Display only: it never changes a severity, a finding's identity or the gate, and Next.js and Go routes say "authentication state unknown" rather than "no guard". It appears on the PR comment, in the JSON report (`routeExposure`, `scaRouteExposure`) and in SARIF.

### Missing-authorization precision
- **An Express route whose URL names auth is no longer read as guarded.** The route's path, or a neighbouring route's path, never counts as a guard.
- **Optional auth and auth-named routers no longer protect a whole file.** Real file-level guards such as `app.use(authentication)` still count, and a commented-out `app.use(auth)` no longer does.
- **An Express route guarded by a camelCase auth middleware is no longer reported as unauthenticated,** for example `requireJwtAuth` or `jwtAuthGuard`. Optional or skipped auth, OAuth flows, token parsers and minters, and optional-mode factories stay reported.
- **A security policy note no longer counts as a data-flow sink in the endpoint risk,** and a flow that ends in test code is reported as a minor naming the test file, not at full severity. A template-literal log message that merely mentions a token is no longer reported as logging one; a credential interpolated into the message still is.

### Missing-authorization: chained and path-level Express registrations
- **Routes registered as a chain are now checked.** `router.route('/items').get(listItems).post(createItem)` produced no route at all, so an unguarded chained endpoint was never reported. Each chained verb is judged on its own arguments.
- **A path guarded with `router.all('/admin', requireAuth)` is no longer reported as unauthenticated** when a later route registers the same path. This applies only when every argument after the path is an authentication guard.

### Taint path shows where the data came from and where it went
- **A flow into a helper now points at the line the tainted argument actually reaches.** Each argument is matched to the parameter it binds, and the finding names the sink that parameter reaches. When the argument does not appear to reach the sink, or the call's arguments cannot be matched, the finding keeps its grade and says so.
- **Java, C# and Go: a helper that builds its dangerous argument in a typed local is now followed across functions.** The same code written without the local was already reported.
- **Cross-file: a value wrapped by a sanitizing helper inside the sink call is no longer reported.** A helper that escapes for a different class of vulnerability still does not clear the sink.
- **The taint path on a finding now names its source and its sink** (for example `req.query.id` and `db.query`) in the PR comment, the HTML report, SARIF and the JSON report.

### Install-time changes on every pull request (lockfiles)
- **The install-time changes hidden in a pull request's lockfile are named on the pull request.** It covers `package-lock.json`, `npm-shrinkwrap.json`, `yarn.lock` and `pnpm-lock.yaml`, compared with the base branch. A new install script raises `lockfile_install_script_added`; a non-registry source, mismatched tarball, new registry host, new alias or plaintext `http` download raises `lockfile_source_changed`; a removed, weakened or different integrity hash raises `lockfile_integrity_changed` (all major, security, always surfaced). Upgrades, dedupes, re-sorting and format migrations are silent.
- **Install configuration changes are review items.** Lifecycle scripts, `.npmrc`, `.yarnrc`, `pnpm-workspace.yaml`, pnpmfiles, yarn releases and plugins, pnpm build allowlists and package-replacing overrides raise `install_hook_changed` (major when the setting redirects downloads or runs code, minor otherwise). Setting names are shown, never values.
- **It fails closed.** A changed lockfile that cannot be compared raises `lockfile_unverified`, and `exclude` and `.gitignore` cannot hide a lockfile. Pro tier and above, and the base branch must be in the checkout (`fetch-depth: 0`); see [/docs/lockfile-diff](https://pullguard.dev/docs/lockfile-diff).
- **A new package whose name imitates a popular one is flagged, and Python, Ruby, Go and Gradle lockfiles are compared too.** This covers npm and PyPI look-alike names, and `uv.lock`, `poetry.lock`, `Gemfile.lock`, `go.sum` and `gradle/verification-metadata.xml`. Turning Gradle dependency verification off, or trusting more artifacts or keys, is a major install-configuration change.

### Agent-capability changes on every pull request
- **A pull request that widens what a coding agent may do is surfaced as a review item.** A new or redefined MCP server, a broader permission, a removed deny rule, a more permissive permission mode, `enableAllProjectMcpServers` turned on, or a new hook or folder-open task raises `agent_capability_widened` (major, security, always surfaced). An edited instruction file raises `agent_instruction_changed` (minor).
- **A comparison that cannot be made is itself the finding,** and an excluded or ignored agent file is still compared. Commands are recorded as hashes only, and the base branch must be in the checkout (`fetch-depth: 0`).
- **Agent and MCP configuration saved with a UTF-8 byte-order mark is now read.** Previously a backdoored `.mcp.json` saved that way produced no finding.

### Exploit witnesses (Java)
- **A Java taint finding now names the request that reaches it.** It gives the method, route, HTTP parameter name and an inert marker to look for at the sink. When none can be derived the finding says why, and it is still reported; the witness never changes severity or the gate.
- **Witnesses cover JAX-RS resources and `web.xml` servlets.** A URL is shown only when it is exact; when the scan cannot see exactly one readable application path, the witness says the path is unknown.
- **A JAX-RS form field, header or cookie flowing into command execution is now graded critical,** like its Spring and servlet equivalents. It was major because of the framework's spelling.
- **The missing-authorization check reads Spring controllers the same way however they are formatted.** Next-line paths, `path = "/x"`, wrapped guard arguments, comment blocks and a far-above class-level guard no longer change the verdict, and a path-less `@GetMapping` handler is now checked.

### Scan-completeness attestation
- **Every scan now writes a verifiable record of what it actually looked at.** `pullguard-scan-attestation.predicate.json`, plus an unsigned in-toto statement, sits beside `pullguard-report.json`. It records the files read, analyzers that ran or did not finish, the source of the zero-day rules, the age of the vulnerability data and the SHA-256 of the exact report.
- **The completeness verdict is strict and lists its reasons.** A scan is not complete after a truncated or failed read, an unfinished analyzer, a degraded engine, missing or stale vulnerability data, an unrecorded commit, a tier that does not run the security analyzers, or a pull-request configuration that could not be checked against the base branch. It proves the scan was complete, not that the code is free of vulnerabilities.
- **Your workflow signs it; PullGuard never does.** Add `actions/attest@v4` after the scan and verify with `gh attestation verify`, pinning the signer as the docs show. New commands: `pullguard attest build`, and `pullguard attest check`, which exits `1` on any mismatch and, with `--require-complete`, `3` when the scan was not complete; `--expect-commit <sha>` and `--expect-image-ref <ref>` fail unless the scan matches.
- **The attested commit is the one on disk, and a vulnerability database with an unreadable or future date is now treated as stale.** A tree that is not the recorded commit, or a scan of only a subdirectory, is not complete. Previously such a database counted as fresh, so the online supplement did not run.

### Java: every annotated request parameter is a source
- **A Spring handler's second `@RequestParam` (or `@PathVariable`, `@RequestHeader`, …) on the same line is now treated as user input.** Only the first annotation on a signature line was bound, so an injection through the second parameter reported nothing. Expect new findings on handlers with several annotated parameters on one line.

### Fewer false positives (opt-in) and a stricter pull-request guard
- **Declare a sanitizer for specific vulnerability classes with `taint.scopedSanitizers`.** A matching call makes a value safe for the listed classes only, so the same value written into HTML is still reported as XSS. It is applied where PullGuard follows data inside a function; cross-function and cross-file analysis uses the built-in sanitizers only. Invalid entries are refused when the configuration loads, and a pull request that adds or widens an entry is scanned without it until merged.
- **A sanitizer now clears only the values that pass through it.** In an expression such as `encodeURIComponent(a) + b`, a value already cleaned could make the whole result clean, so injection through `a` went unreported. These flows are now reported.
- **Python keyword arguments are no longer read as assignments.** This removes a false positive on constant commands and a missed injection where only a quoted copy was passed as a keyword argument.
- **A destination that is provably narrowed is now named on SSRF findings.** This covers a URL literal that fixes scheme and host, and a URL checked against a constant allowlist. The grade is unchanged by default; `taint.ssrfNarrowedDestinations: downrank` lowers them one step.
- **A pull request can no longer relax its own scan through three more settings.** `taint.ssrfNarrowedDestinations: downrank`, `taint.calleePositionalProofs: true` and `sqlInjection.patternSeverity: major` added in a diff are not applied until merged, and the scan notes which one it held back. Documentation correction: `taint.sanitizers` clears flows into the custom sinks you declare, not built-in findings, and never has.

### What upgrading a vulnerable dependency involves
- **A dependency CVE now says whether its fix is on the release line you run.** The lowest version that fixes every advisory on the finding is named and labelled as on your line, a new major, or not published for at least one advisory. It appears on the PR comment, in the JSON report (`upgrade`) and in SARIF.
- **The lines that use the package are listed, production code first** (up to 20, with an exact count). It never says an upgrade is "safe"; it tells you what to re-test. Display only: it never changes a severity, a finding's identity or the gate.

### JavaScript and Python: taint across files through renamed imports
- **A flow into a function imported under another name is now reported.** This covers renamed, default and namespace imports, renamed CommonJS destructuring and Python `from .svc import run as r`. An anonymous default export is never matched to another function.
- **JavaScript / TypeScript: a flow through a barrel file is now followed.** Re-exports are followed up to four files deep, and a name provided by two `export *` sources is not guessed. `export * as ns from`, type-only re-exports and CommonJS barrels are not followed yet.
- **Unused-export and breaking-change checks now read the exported name of a renamed import.** A Python relative import no longer counts the module's own name as an imported symbol.

### Server-side request forgery: Spring WebClient written across lines
- **A Spring `WebClient` / `RestClient` call formatted across lines is now checked for server-side request forgery,** like the same call written on one line. A fixed URL template with the request value only as a path variable is still not reported.

### C#: ASP.NET Core request data
- **Request data read through ASP.NET Core is now tracked.** `Request.Query`, `Request.Headers`, `Request.Cookies`, `Request.RouteValues` and `Request.Body`, also through `HttpContext.Request` or a minimal-API request, were not recognised as user input. Expect new findings on ASP.NET Core applications.
- **A C# request value that reaches a command is graded like every other language.** It was graded major instead of critical, so a scan gating on critical findings will now fail on these.

### Java: taint across files
- **A Java flow that crosses into another file is now reported.** This covers an injected service, an instance built with `new`, a static utility call, a helper that returns request input to a sink in the caller, and a fully qualified class. Only a flow that reaches an actual sink is reported, and a call through an interface reaches its implementation only when exactly one class implements it. Expect new findings on Spring-style applications.
- **A flow already reported inside one file is not reported again across files,** and two dependent calls on the same input are one finding. The second call stays as a minor pointing at the first finding; a second call that uses the input itself is still its own full-grade finding.
- **Java and C#: a call to an overloaded method is matched to the overload it calls,** by argument count. Where that cannot decide, the previous behaviour applies, and a changed match keeps the previously reported row with a note rather than dropping it.
- **LDAP and XPath injection found across functions or files is labelled as such** instead of command injection, and a value returned from another file and then escaped by a helper in the calling file is no longer reported when the helper covers the sink's kind. The latter applies to Java, JavaScript and C#.

### Server-side request forgery through Spring and Java HTTP clients
- **A request value used as the destination of a Spring `RestTemplate`, `WebClient` or `RestClient` call, or of a `java.net.http` request, is now reported as SSRF.** Only the destination counts: a fixed host with the value as a template variable, body or header is not reported.
- **A fully qualified request annotation is a taint source like the imported one,** so a flow from a parameter annotated that way is now reported at the same severity. A Java string literal no longer counts as a use of a variable it merely names.

### Missing authorization on JAX-RS endpoints
- **An unguarded JAX-RS resource method is now reported as `missing_auth_check`.** The check credits `@RolesAllowed` and `@DenyAll`, a `web.xml` security constraint that requires a role, a global request filter that rejects requests without a valid credential, name-bound filters, and `@Authenticated` or `@Auth`. `@PermitAll` is treated as public by design and is not reported.
- **A first-statement role check, a registered filter and a filter-free application are judged accurately.** A role check that rejects as the method's first statement guards it; a filter registered by the application class counts; a `@Provider` filter an application does not list no longer counts. Test-only applications or filters never guard production code.
- **JAX-RS sub-resources are now checked,** when exactly one locator returns them. When the locator rejects unauthorised callers first, the endpoint is reported as a minor naming the locator.
- **Protection PullGuard has to infer is shown, not trusted silently.** A `web.xml` in a web root, a module-wide Spring Boot authentication requirement, or a `web.xml` in a packaging WAR lowers the finding to a minor naming where the protection was inferred from; it is never removed. A role check whose role is chosen by the caller is reported as a minor that says why.

### Agent-capability review on pull requests
- **Four more ways a pull request could change what a coding agent may do are now reported:** renaming an agent settings file inside a linked configuration directory, a Windows short-name path that writes into `.claude` on checkout, a file that is agent configuration in another letter case, and a change to a script a hook runs or a file an instruction file imports. The review needs `fetch-depth: 0`; without it the report says no comparison was made.
- **A file whose name imitates agent configuration with look-alike letters is now reviewed as the agent file it imitates.**

### Cross-site scripting in Flask and Django responses
- **Request input returned inside an HTML body through Flask `make_response(...)` or Django `HttpResponse(...)` is now reported.** An escaped value, a template render, a JSON response, a header-only value and a non-HTML `content_type` or `mimetype` are not. Expect new `xss` findings.
- **The same applies when a helper function builds the response,** and the finding points at the response line in the helper.

### Fewer false positives when a helper sanitizes at the sink
- **A value sanitized on the line where a helper function uses it is no longer reported as reaching that sink through the call.** For example, a helper that writes an HTML-escaped value into `innerHTML`, or runs a command built with `shlex.quote`, was reported up to critical. The sanitizer must fit the vulnerability class: HTML escaping inside a shell command is still reported.

### Missing authorization: Spring mappings written after `method =`, and implemented interfaces
- **A Spring handler whose path is written after another argument is now checked for a missing authorization guard.** `@RequestMapping(method = RequestMethod.GET, value = "/owners")`, also what OpenAPI generators emit, was not read as a route.
- **A controller that implements an interface holding its mappings now has each handler checked on its own,** with the interface's path and the handler's own guard. The interface itself is no longer reported as an unguarded route once an implementing controller is found.

### Scanner dependency security update
- **The YAML parser the scanner uses is updated (`js-yaml` 4.3.2).** The previous version had two published high-severity advisories for quadratic CPU use on crafted YAML, which a pull request could supply in a configuration, workflow or manifest file to slow a scan.

### Vulnerable dependencies in Kotlin
- **Kotlin controllers now get a route answer for a vulnerable dependency.** A controller that calls a service using the package is connected to its routes, and each Kotlin handler, with a block or expression body, says whether it references the package.
- **A Kotlin file that imports a vulnerable package under an alias now counts as using it,** including a call made through the alias.

### VS Code MCP configuration in the composite checks
- **A VS Code `.vscode/mcp.json` now counts toward the lethal-trifecta check and the MCP rug-pull baseline.** Previously a configuration that granted private data, untrusted content and a send channel never raised `agent_lethal_trifecta`, and a changed tool was never compared with its approved definition. Expect new findings where such a file exists.

### Every inline route handler's flow is reported
- **A second injectable route handler in the same file is no longer lost.** Two anonymous handlers with the same source and sink were treated as one flow, so the second never appeared as an occurrence, in SARIF or in flow-level views. Each is now reported.
- **Two functions with the same name in one file each report their own flow.** Java overloads, or a function beside a same-named object or class method, previously reported only the last one's injection.

### Prove a calibration before accepting it
- **`pullguard calibrate prove` tests a server calibration proposal against your own code.** Give it the JSON from `GET /api/v1/repos/<repo>/calibrations` and a proposal id. It writes `pullguard-calibration-proof.json` and changes nothing else.
- **A proof checks that the call behaves like a sanitizer for the dismissed flows.** It must silence enough of them to meet the proposal thresholds (5 sinks, 2 files, 2 reviewers), must not be a known pass-through, and must show its own vulnerability class going silent while other classes still fire. Every other finding it would clear or lower is listed as collateral.
- **The result is "consistent", never "proven".** It does not verify that the call sanitizes; that is still the reviewers' judgement. Exit codes: `0` consistent, `1` rejected with reasons, `2` could not be proven.

### Server (Enterprise)
- **Action required if you set `PULLGUARD_SERVER_TRUST_PROXY` to a number.** A hop count (`1`) no longer starts the server, because a client reaching it directly could set `X-Forwarded-For` and choose the address that rate limits and the audit log see. Set it to your proxy's IP address or CIDR range instead (several allowed, comma-separated); the startup error names the replacement, and the AWS ECS (Pulumi) template now sets the VPC's range. An unrecognised value now prints a warning at startup and still means "do not trust the proxy header".
- **Dependency security updates.** The web framework is updated to Fastify 5.12.5, and the SAML sign-in XML parser (`@xmldom/xmldom` 0.8.15), the URI parser (`fast-uri` 3.1.8) and the router (`find-my-way` 9.9.0) are updated, fixing published high-severity advisories in the versions shipped before. Every release now checks the runtime dependencies of everything it ships and stops on any known advisory.
- **The Helm chart deploys the server version it ships with, not `latest`.** The default image tag is now the chart's own version (0.4.5); set `image.tag` to run another.
- **New: rule precision from your own triage.** The triage page and `GET /api/v1/repos/<repo>/rule-precision` show, per rule, the findings tracked, how many carry a current decision and how many were marked false positive. It is an observed ratio, not a measured precision. Above 50,000 tracked findings the API returns `truncated: true`, and a decision voided by a change to the finding is no longer counted.
- **Proposed calibrations, read-only and unproven.** When your team repeatedly dismisses the same kind of taint finding as a false positive, the server groups those decisions into a proposed calibration; it needs at least 5 dismissed sinks in 2 files from 2 signed-in reviewers on the protected branch. A proposal changes nothing and is not a gate input. It is shown on the triage page and at `GET /api/v1/repos/<repo>/calibrations`, and carries keys, locations and counts, never comments or reviewer identities.
- **Choose the branch calibration reads.** A repository whose protected branch is not `main` or `master` can name it with `POST /api/v1/settings/repos/<repo>/calibration-branch`. Administrators only, signed in, and audited; `null` resets.
- **Accept a calibration proof.** `POST /api/v1/repos/<repo>/calibrations/<proposalId>/accept` with the proof from `pullguard calibrate prove` records an administrator's recommendation and returns the `.driftrc.yml` entry to propose in a pull request. It changes no scan until that pull request merges. An edited proof, another repository's scan, a pull-request scan, a basis older than 30 days or a withdrawn dismissal is refused; accepting the same proof twice stores it once, a proposal ranked past the 50 shown can be accepted, and the acceptance is audited.
- **A scoped read token can now read triage decisions.** `GET /api/v1/repos/<repo>/triage` accepts a managed `read` token for repositories inside its scope; the unscoped environment read token is still refused. A request outside the scope is refused with `403` and audited, and refused reads on the rule-precision and calibration routes are now audited too.
- **Triage writes are more forgiving and stricter in the right places.** A write that sends its CSRF token in a `csrf` body field is accepted again, and `reason` is accepted as the same field as `comment`. An unknown field in the body is now refused with the list of allowed fields instead of being ignored.
- **Triage fixes.** A decision on a finding without a rule id no longer fails and is treated as always surfaced, so it needs a reason. An out-of-team-scope write is refused with `403` without also logging a server error, and a database that already holds duplicate calibration acceptances starts normally, keeps every row, and logs how many groups need resolving.

---

## [1.5.17] — 2026-09-25

> **Ships with server 0.4.4** (self-hosted Enterprise server). No Action input changes: `@v1` users get the new
> scanner image on their next scan.

### Dependency vulnerabilities and SBOM
<!-- covers: Dependency vulnerability coverage -->
<!-- covers: Dependency upgrade advice -->
<!-- covers: CVE data age is always stated -->
- **pnpm 11 and later lockfiles are read again.** pnpm now writes `pnpm-lock.yaml` as two YAML documents, and
  previously only single-document files were read. As a result, SBOMs for pnpm repositories silently lost every
  transitive dependency, and CVE matching used the lowest version a `package.json` range allows instead of the
  installed version. That could report a vulnerability you had already upgraded past, or miss one above the floor.
  Expect larger SBOMs for pnpm repositories, and CVE findings against the versions actually installed.
- **Python, Maven, Go and Ruby manifests below the repository root are now scanned** for CVEs. Each finding names
  the manifest it came from.
- **"Fix available" names the fix on your own release line** instead of the newest line of the package.
- **CVE data age is always stated.** An offline scan against old data says so.
- A manifest inside a test or fixture folder is still reported, one severity step lower, with the reason stated.
- Expect new CVE findings on repositories with manifests below the root.

### Hardcoded secrets
- **Secrets are now detected in entry points, config files, components and agent code.** This covers `app.py`,
  `server.js`, `index.ts`, `*.config.js`, React `components/`, Next.js `pages/`, agent `tools/`, `mcp/` servers and
  `infra/` code. The secret rules had previously shared a file list with the dead-code check, so none of them ran
  on these files. Expect new `hardcoded_secret` findings where such files carry credentials.
- **A real provider key in test, example or documentation code is now reported** at full severity. This covers
  AWS, GitHub, Stripe, Slack, OpenAI, Anthropic and other value-verified formats.
- **Low-confidence matches are shown at minor instead of dropped or raised to critical.** This applies to:
  - label-only placeholders in test and documentation directories;
  - values named as public keys;
  - Solana program ids, mints and transaction signatures;
  - the public members of a JSON Web Key.

  Each one is reported once per file, with the reason and a count, and does not fail a gate set at major. A value's
  name alone never hides it: a secret stored as `publicKey` or `mint` stays visible. No path reports less than it
  did in 1.5.16.
- **Entropy-based detection now runs on production paths whose names contain "test"**, such as `src/attestation/`
  and `latest/`.

### Python taint analysis
- **Python data flows between files are now analysed.** Absolute and relative imports resolve against the
  repository root and `src/`-style layouts.
- More request reads are recognised as user input across aiohttp, Starlette/FastAPI, Flask and Django.

### Zero-day threat rules in air-gapped environments
- **Air-gapped scans can now run the Tier-2 zero-day threat rules.** Previously an offline scan used the built-in rule
  catalog, which carries no zero-day rules, and there was no way to supply them. To enable them:
  1. On a connected machine, save the signed bundle: `curl -o rules-bundle.json https://pullguard.dev/api/rules`.
  2. Carry the file into the air-gapped environment.
  3. Set `PULLGUARD_RULES_BUNDLE_PATH` to its path.

  The file is verified with the signing key the scanner already embeds, and no network call is made. A modified,
  unsigned, oversized or unreadable file is rejected; the scan then uses the built-in catalog and says so.

### Finding locations
- **Findings for data that flows between functions in the same file now point at real lines.** Every step in the path
  carries the line of the call it makes, and the sink end carries the line of the dangerous operation. Previously both
  ends showed the function declaration and the intermediate steps showed no line. The source end is the line that reads
  the value passed down the chain, not a condition that only tests it. Only the displayed lines change: findings,
  severities, counts and baseline fingerprints are unaffected.

### Fewer false positives
- **Solana public keys, program ids, mints and transaction signatures** are reported as one minor finding per file
  instead of major hardcoded secrets. A 64-byte secret key, a 32-byte value named as a seed, values in an unnamed list
  and any real secret later in the file are still reported at full severity.
- Python DB-API `execute`: a value passed as a bound parameter is no longer reported as SQL injection, inline or through
  a call, and a triple-quoted bound query is no longer reported.
- A docstring that quotes old vulnerable code no longer produces a finding.
- `breaking_change` no longer reports removed parameters on unchanged functions that have several `@overload`
  definitions or generic type annotations. A parameter removed after an annotation is now reported. A signature
  baseline written by an earlier version is compared as written, then upgraded in place.
- A variable name that appears only inside a string literal is no longer treated as a use of that variable, and a
  comparison (`if (req.query.name == null)`) is no longer read as an assignment.
- A request value that only chooses between fixed strings (`res.send(req.query.n === 'a' ? 'x' : 'y')`) is no longer
  reported as XSS. A request value used as one of the results still is.
- Django open redirects: a same-origin target rebuilt by a helper, as in Django admin, is no longer reported. A target
  checked with `url_has_allowed_host_and_scheme` (or `is_safe_url` before Django 3.0) is reported **one grade lower**,
  with the check named. It is never hidden.
- Rails' own definition of `skip_forgery_protection` is no longer reported. An application that calls it still is.
- A Go module that `go.mod` replaces with a local directory is no longer matched against the published module's
  advisories.
- A Python shell command whose request values all pass through `shlex.quote` is reported **one grade lower**, with the
  reason stated. It is never hidden.
- A Python import declared by a manifest in one of the file's parent directories (`ml/requirements.txt` for
  `ml/train.py`) is no longer reported as an undeclared or hallucinated package.

### More findings
Expect new findings on code with these shapes; they can change a gate result.
- A request read inside a triple-quoted Python f-string is analysed again. It was missed in 1.5.15 and 1.5.16.
- A value passed to a method of an imported class instance (`const r = new Runner(); r.run(req.query.dir)`) is followed
  into that method.
- Functions and helpers whose names start with `$` are followed within a file, as they already were across files.
- A parameter that reaches a sink through a tuple unpack inside the called function is followed.
- **An Express catch-all handler that redirects to the raw request path is reported as an open redirect.** A
  leading-slash guard is honoured.
- On Windows (CRLF) checkouts, Python `import` lines are read correctly, so an undeclared or hallucinated package
  imported that way is reported.

### Consistency
- Findings appear in the same order on every scan of the same code.
- A step in a flow that returns a value to its caller shows the line of the `return`.
- In a long-running process such as the MCP server, a registry lookup that failed for one scan no longer affects
  another scan running at the same time.
- On Windows (CRLF) checkouts, a call written across several lines is read the same as on a Linux checkout.
- Text inside Python f-strings no longer counts toward complexity metrics, so complexity findings match 1.5.16 on
  unchanged code. Security analysis still reads f-strings as code.

### Evidence quality
- An `endpoint_risk` finding names every unauthenticated route in the file, not only the first.
- A cross-file flow into a list argument of a shell-less subprocess call is graded `moderate` with the reason stated,
  matching the same-file analysis, and its sink carries a line.

### Type coverage
- A function passed directly as a call argument (the callback in `events.map((e) => e.id)`) is typed by the receiving
  function and is no longer counted as untyped. This is a measurement correction: the type-coverage percentage and the
  `low_type_coverage` severity band can move, usually upward, on files whose source has not changed.

### Server (Enterprise) — 0.4.4
- **`pullguard-server` is now a command inside the image**:
  `docker compose exec pullguard-server pullguard-server token mint …` and `… audit verify` work as documented.
  On 0.4.3 and earlier, use `docker exec <container> node dist/index.js token mint …`.
- An unrecognised command prints usage and exits with code 2 instead of starting a second server.
- The token-mint example in the docs is now a single line, so it can be pasted as-is. Compose, Docker and Kubernetes
  forms are documented.

---

## [1.5.16] — 2026-09-24

> **No server release.** Server stays at 0.4.3; pulling the new scanner image is the whole upgrade.
> **Recommended over 1.5.15** for every user.

A hotfix for dependency vulnerability matching, plus three detection regressions from 1.5.15.

### Security — dependency vulnerabilities
- **Advisory version ranges are evaluated exactly as the advisory schema defines them** whenever a local
  vulnerability database is used, including the database that now ships in the image. Previously an advisory whose
  range ended in a "last affected" version could be matched against every later version, producing a false
  finding. This affected every earlier release that used a local database; 1.5.15 made the local database the
  default. A range that cannot be evaluated is reported as unverified at its full severity, never dropped.
- **One vulnerable package no longer hides another.** Dependency findings are now reported once per vulnerable
  package instead of once per manifest, on both the local and the online path. Expect more `known_cve` findings on
  some repositories: they were already found, and previously only the most severe package per manifest was shown.
- **Withdrawn advisories are no longer reported**, advisory ids are counted once, `Gemfile.lock` versions are used,
  Python package names are normalised, and npm aliases are checked as the real package.
- A dependency whose version cannot be resolved keeps its real severity and is marked as unresolved.
- **One-time re-key:** because dependency findings are now per package, baselines and code-scanning alerts for
  dependency findings re-key once on the first scan after upgrading.

### Detection — regressions from 1.5.15, fixed
- A shell-less list-form `subprocess` call with user input as a discrete argument is graded the same (moderate, with
  the mechanism named) whether the value is written inline, through a variable, or across functions. A value handed
  to a program that executes its arguments (`sh -c`, `python -c`, `sudo`, `ssh`) is critical again.
- Cross-file flows through a function whose name starts with `$` are reported again.
- Cross-file flows whose return value is destructured are reported again.

### Corrections to the 1.5.15 notes
- "Still reports every CVE" offline was not true in every configuration until this release (see above).
- "Comments never change a verdict" applies to code-flow rules; hardcoded-secret detection deliberately still reads
  comments, because a commented-out credential is a real leak.

## [1.5.15] — 2026-09-23

> **No server release.** Server stays at 0.4.3; pulling the new scanner image is the whole upgrade.

Every published scanner image now carries its own vulnerability database, images are built for both
`linux/amd64` and `linux/arm64` under one digest, and a taint finding points at the line that is
actually dangerous — the true source and the true sink, including when a flow crosses files.

### Security
- **A taint finding now points at the line that is actually dangerous.** A message-handler flow
  could be reported with its dangerous write attributed to the wrong line in the same function,
  while the real write went unreported; both the true source line and the true sink line are now
  reported, and the same fix applies when the flow crosses files.
- **A dependency scan that could not finish is never silent.** When the dependency analyzer can't
  complete in time, the console now prints an explicit `INCOMPLETE:` banner naming it, in addition
  to the note already carried on the report and PR comment. An opt-in `--fail-on-incomplete` (or
  `PULLGUARD_FAIL_ON_INCOMPLETE=true`) exits non-zero on an incomplete scan instead of a clean one.
- **Argument-list command execution is graded as argument injection**, not full command injection,
  unless the value could become a flag or the call runs through a shell — nothing is dropped, only
  ranked correctly.
- **`postMessage` wildcard-target detection catches a few remaining spellings** that a nested
  function call or object literal in the first argument could previously hide from it.

### Supply chain
- **Every published scanner image now ships its own vulnerability database**, refreshed when the
  image was built. CVE lookups run locally and deterministically with no configuration required —
  in the Marketplace Action and via `docker run` alike — so a runner with no outbound network access
  still reports every CVE. When the image's database is older than the configured freshness window
  and the network is reachable, the advisory service is consulted as a supplement and merged in;
  being unreachable never drops a locally-known finding. `db.path` and the new `PULLGUARD_DB_PATH`
  variable still point a scan at a different database, and `pullguard db status` now reports plainly
  when none is present.
- **Images are published for `linux/amd64` and `linux/arm64`** under a single tag and digest, so a
  self-hosted arm64 runner pulls a native image instead of emulating one.

### Precision — false positives removed
- **Comments never create, silence, or change the severity of a finding**, in either direction,
  across the reflection, injection and JWT checks.
- **A route written inside a comment or a string literal is not an endpoint**, and CORS preflight
  requests and the standard browser-convention public files (favicon, robots.txt, security.txt, web
  manifests) are no longer reported as unauthenticated.
- **A Content-Security-Policy directive governed only by `style-src`** is down-ranked rather than
  dropped; the same value under `script-src` or `default-src` is unchanged.
- **A parameterised query in Go is recognised as parameterised**, not reported as string-built SQL.
- A handful of related false positives in route detection, template rendering and cross-file
  analysis are fixed alongside these.

### Recall — shapes that are now reported
- **Go:** a request body, path or query value read through a multi-value assignment, or decoded
  directly into a struct, is now traced as a source.
- **All languages:** a request value used directly as an argument to a sink call, with no
  intermediate variable, is now reported at the same severity as the same value through a variable.
- **Open redirect:** the fallback-target idiom (`next || currentPath`) and Django's `redirect()` are
  now covered.
- **A render callback that forwards a whole request object into a template is traced through to its
  output.**
- **Go:** a handler written as a closure and passed directly into a registration call, rather than
  declared separately, is reported again — a gap introduced in the previous release is now closed.

### Performance
- **Cross-file analysis no longer recompiles a pattern for every function it inspects** on a large
  repository — one pass per file, identical findings, substantially faster on large codebases.
- **Long-running uses of the scanner (the MCP server, IDE extensions) load language grammars once
  per process instead of once per scan.**

### Quality gates
- Release verification now compares a taint finding by both the line it came from and the line it
  reached, not only its anchor point, so a finding that names the wrong line is caught before it
  ships.

## [1.5.14] — 2026-09-22

> **No server release.** Server stays at 0.4.3; pulling the new scanner image is the whole upgrade.
> **Please upgrade if you scan large Gradle monorepos** — see the first item.

A dependency scan that runs out of time no longer reports a clean result. On a large Gradle
monorepo, the subproject reading shipped in 1.5.13 grew the dependency inventory from 10 to 390;
the extra work crossed the dependency analyzer's time limit, the analyzer was stopped, and the 24
CVE findings it had already computed were discarded — the report said zero. Every part of that is
fixed, and the analyzer now does a fraction of the network requests it did.

### Security
- **Computed CVE findings survive a time limit.** Findings stream to the report as they are found;
  a run that hits its limit keeps everything found so far and marks the analyzer incomplete rather
  than reporting a clean result.
- **`PULLGUARD_ANALYZER_TIMEOUT_MS` now governs every execution mode.** The default mode had a
  separate, undocumented 30-second limit that the setting did not reach.
- **A hit limit is announced where you read.** The job log and the Checks tab now carry an explicit
  annotation naming the analyzer; the PR comment and step summary already did.

### Performance
- **One vulnerability query per scan instead of one per dependency.** A 320-dependency repository
  makes 13 advisory requests where it made 31, a lockfile-heavy one 23 where it made 74, with
  identical findings.
- **Licence resolution twice as fast**, same requests, same answers.
- **Maven freshness now measures Maven.** Three quarters of its registry requests were timing out;
  it reads the registry's static metadata instead — every dependency checked, no aborts.
- **Per-analyzer timings on every report** (`analyzerDurations`), and the slowest analyzers printed
  at the end of a scan.

### Detection
- **A call whose arguments wrap across lines is read like one that does not.** The
  `permissive_postmessage_target` rule shipped in 1.5.13 reported nothing on wrapped calls — the
  way most real sites are written. The same class affected 48 known-answer cases across the rule
  matchers, the dataflow engine and the cross-function analysis; all now agree in both spellings.
  On the OWASP Benchmark this is +31 true detections with nothing lost.
- **Fixed before shipping:** the wrapped-call change had silenced one open-redirect shape — a
  request value read inside a wrapped call a few lines above the sink. Caught by comparing every
  finding on real repositories between the previous release and this one, and fixed before the tag.
- **A guard that pins a value to one exact string is proof.** `if (url.matches("http://…"))`
  before `new URL(url)` no longer reports SSRF.
- **A `String` parameter passed as a literal at every call site is proven.** A helper concatenating
  such a parameter into SQL is kept at `major` with the proof attached (`all 6 call sites across
  2 files pass a literal`) instead of `critical`; one caller passing anything else, or a scan that
  did not read every file, keeps `critical`. Java and C#.

### Configuration
- **`sqlInjection.patternSeverity: major`** (opt-in, default `critical`): rank the taint-unaware
  SQL string-concat rule one step below dataflow-proven findings when it has no proof of its own.
  Findings are kept and record the option that moved them; the option cannot go below `major`.

### Quality gates
- The real-repository regression bed fails on a per-analyzer cost cliff, not only on a per-scan
  total, and refuses to score a scan whose analyzer did not complete.

## [1.5.13] — 2026-09-17

> **No server release.** Server stays at 0.4.3; pulling the new scanner image is the whole upgrade.

A value that crosses a function boundary is now traced as reliably as one written inside a single
function — a listener handing its payload to a helper, an annotated request parameter, code at the
top level of a module. Large Java and TypeScript projects also scan substantially faster: on the
1,420-file performance bed a whole scan goes from 88 s to 53 s, with findings verified identical
between the two published images.

### Detection
- **A `postMessage` payload handed to another function is traced.** A `message` listener that
  destructures its event and passes the value on produced nothing before, while the same two lines
  inside one function were reported.
- **An annotated request parameter is a source across a call boundary** (Java / Spring).
  `handler(@RequestParam String q) { helper(q); }` was silent while the `request.getParameter(...)`
  twin fired.
- **Code at the top level of a module is a caller.** A value read at module scope and passed to an
  imported function is now followed into it.
- **Vulnerability types are reported accurately.** `new URL`, `new File` and most `Files.*` sinks
  were reported as command injection; an SSRF is now reported as an SSRF and a Zip Slip as a path
  traversal, including across files.

### Precision
- **A route MOUNT is not a dataflow.** Registering a handler no longer reports the handler's sinks
  against the file that mounted it, and the endpoint-risk scores built on those reports are gone.
- **A sanitized value stays sanitized across a module boundary.** `escapeHtml(raw)` passed to an
  imported function was re-reported as attacker input.
- **A finding is no longer decided by a line that runs after it.** An assignment below a sink could
  both create a finding that was not real and hide one that was.
- **A finding's location is the code, not the comment.** A rule could anchor on text inside a
  docstring rather than the statement it describes.

### Performance
- **Large projects scan substantially faster.** Work that grew with the number of tracked values in
  a file is now flat; findings are unchanged.

### Reporting
- **An offline scan says the curated zero-day rules did not run.** With `PULLGUARD_OFFLINE=true`, or
  when the signed rules bundle cannot be fetched, those rules are unavailable — previously silent
  while tier and coverage both reported healthy. The cause is now named, and a machine-readable
  `threatRuleCoverage` lets a dashboard or gate tell "no zero-day findings" from "the zero-day rules
  never ran".
- **Scan duration is retained with each scan**, so the over-time dashboard and a self-hosted server
  can trend scan cost alongside findings.
- **New: `permissive_postmessage_target`.** `postMessage(payload, '*')` broadcasts to any origin.
- **The gate banner states the tense that is true.** A run that enforces nothing reads
  `would FAIL … this run is ADVISORY` instead of claiming a FAIL beside a green check.
- **A complete scan says it is complete**, rather than stating coverage only when a scan is partial.

### Licensing
- **The expiry warning starts 90 days out and escalates.** It fired only inside 14 days, which is
  not enough notice to raise a purchase request, get it approved and rotate a secret in CI.

### Dependencies
- **Every Gradle subproject is read**, not the root build file alone, and a version declared in a
  constraints or platform block resolves.

---

## [1.5.12] — 2026-09-15

> **Ships with server 0.4.3** (one attribution fix, no upgrade step beyond pulling the image).

The dataflow engine's pattern matching is 4–8x faster on large Java and JavaScript codebases, with
every finding identical before and after. Measured on the OWASP Benchmark (2,740 Java test cases) a
whole scan goes from 84.7 s to 18.7 s and on WebGoat from 24.6 s to 9.7 s; peak memory is unchanged.

### Performance
- **Scans are much faster on large codebases.** The dataflow engine's pattern matcher now uses
  literal prefiltering and DFA acceleration. Same patterns, same matches, faster; findings are
  byte-identical on every benchmark bed. The container image grows by about 280 KB.

### Fixed — detection
- **A destructured parameter with a default keeps its taint** (JavaScript / TypeScript).
  `({ data: payload = {} }) => …` and `const { q = {} } = req.body` now track `payload` / `q` as
  attacker-controlled; the default was previously misread as a reset and the flow was dropped.
- **`dangerouslySetInnerHTML` on a `<style>` element is no longer reported as XSS** (React). A style
  element's content is CSS text, never parsed as markup. Any other element is reported as before.
- **`escapeHTML(...)` and `encodeHtml(...)` are recognised as sanitizers** at an HTML sink.
- **A documentation tree's `docs/requirements.txt` counts as a declared-dependency manifest**
  (Python), so a Sphinx build's imports are no longer reported as hallucinated dependencies.

### Fixed — scan behaviour
- **`maxFiles` is an exact cap.** It used to overshoot by one directory batch; the coverage banner
  and the report now agree exactly, and a repository with exactly `maxFiles` files is reported
  complete.
- **The Action job log shows the zero-day threat-rule summary line** at the default log level, so a
  run proves the curated pass ran and what it cost.

### Server (Enterprise) — 0.4.3
- **A triage decision written with a managed token is attributed to that token** on the finding and
  in the audit log (`agent:<client>#<token id>`); a caller that sent no client name previously read
  back as `agent:unknown`. The id is the token's row id, never the secret.

---

## [1.5.11] — 2026-09-14

> **Ships with server 0.4.2** (two opt-in additions, no upgrade step beyond pulling the image).

The Java performance follow-up to 1.5.10, plus coverage you can gate on. Java-heavy repositories
scan faster again with every finding identical before and after; a default scan now analyses the
whole repository instead of a fifth of it; and both the Action and the self-hosted server can
refuse a partial scan.

### Changed — the default read cap now equals the default collection cap
- `maxFilesRead` defaults to `maxFiles` (5,000) and follows it when you raise `maxFiles`. A default
  scan used to collect up to 5,000 files but analyse only the first 1,000 by path. A default scan is
  now either complete or truncated at collection, and says so on every surface. Repositories above
  1,000 files will see longer scans and more findings from this release — those findings were
  always there. `maxFilesRead` remains an advanced override for a deliberate partial scan.

### Added — gate on scan coverage
- **Action output `partial-scan`** — `true` when a read, collection or depth cap cut the scan short,
  `false` when complete, `unknown` when the report carried no coverage data. Fail your job on it, or
  run a full scan on a schedule when the pull-request scan was capped.
- **Server quality-gate condition `partial_scan_max`** — set to `0` to refuse any capped scan. A
  report with no coverage data counts as partial.

### Fixed — scan performance on Java
- Java-heavy repositories scan faster again with findings identical, and online scans no longer
  pay a hidden per-rule pass for the zero-day threat rules (that pass was most of a Java CI scan's
  wall clock; it is now well under a second). Two numbers, stated separately: on our 2,740-file
  Java benchmark the offline engine scan went from 107.0 s to 87.2 s (-18%), and the same scan run
  online — the way CI runs — from 234.7 s to 83.9 s (-64%). The remaining Java gap against 1.5.8
  is separately attributed and work continues.
- A scan that runs out of its time budget now reports what it did, lists the analyzer it stopped
  in, and marks the findings as a lower bound, instead of being killed with no report.

### Fixed — precision and recall (Java)
- A value coerced to a non-text type, a method that commits its response as JSON, and a GraphQL
  argument declared as an enum are no longer treated as attacker text.
- A request value read through a formatter-wrapped method chain is followed again; a sort key
  spelled as a literal plus a request value is now reported.
- Findings inside a debug-filter-gated class name the gate; dataflow findings name the real source
  and sink lines.

### Changed — one grade for cross-site scripting
- `xss_vulnerability` reports at `major` from the pattern rule as well as from the dataflow engine.
  If your gate keys on critical XSS from the pattern rule, it will see those findings at major.

### Added — switches
- `threatRules.enabled: false` skips the curated zero-day pass (on by default; the skip is logged).
- `taint.calleePositionalProofs` (default `false`) enables per-position helper proofs on Java.

### Server (Enterprise) — 0.4.2
- **Quality gate `partial_scan_max`** (above).
- **`PULLGUARD_SAML_UNSOLICITED_REDIRECT=true`** (default off) bridges the identity provider's
  launcher tile: an unsolicited SAML assertion is answered with one redirect into a normal login
  instead of the `Login failed` page. The assertion is never parsed or used, the redirect target is
  fixed, a login that fails after that redirect shows the error, and the refused attempt stays in
  the audit log.

---

## [1.5.10] — 2026-09-10

> **Server unchanged at 0.4.1** — no upgrade step for self-hosted deployments.

A fast-follow patch. **Scanning a JavaScript file could take dramatically longer on 1.5.9 than
on 1.5.8** — in the worst cases long enough that a scan had to be abandoned rather than waited
out. This release returns that work to 1.5.8 timing.

### Fixed — JavaScript scan performance
- Work that depends only on a function and its name was being repeated at every step of an
  internal analysis walk instead of once. On a reduced file supplied by the customer who
  reported it, a 2KB source file went from 0.5s on 1.5.8 to 15.8s on 1.5.9; it is back to
  1.5.8 timing.
- **No detection change.** Findings are identical before and after: the OWASP Benchmark
  confusion matrix is byte-identical per category, and a per-finding differential across nine
  real vulnerable applications shows zero findings gained or lost. Nothing was disabled,
  narrowed or traded away to make the scan faster.
- The reporter's file is now a permanent regression fixture in our test suite, and scan
  duration is checked on a large-repository bed as part of every release.

### Known issue
- **Java-heavy repositories may still scan more slowly on 1.5.9 / 1.5.10 than on 1.5.8.** That
  is a separate regression, it is not addressed by this release, and work on it is ongoing.

---

## [1.5.9] — 2026-09-08

> **Server unchanged at 0.4.1** — no upgrade step for self-hosted deployments. The server notes
> below shipped with 0.4.1 and are recorded here for the first time.

This release is about **Java coverage and precision**: PullGuard now follows request data
through the shapes real Java applications actually use, and reports fewer findings it cannot
stand behind. Every finding a v1.5.8 scan reported on our benchmark applications is still
reported, except one that was matching a commented-out line.

### Added — request data is followed through the Java shapes real code uses
- **The servlet request surface.** Header enumerations, parameter names and the request line
  (URI, URL, servlet path, path info) are treated as attacker input, and a for-each loop over
  request cookies, headers or parameters taints the loop variable.
- **Wrapper classes.** When a class is constructed with the request and one of its methods
  returns a value read from it, that value is followed into sinks in the calling code — whether
  the wrapper is held in a variable, chained directly, stored in a field by the constructor and
  read in another method, or declared in another file. Only methods that demonstrably return
  request data count; when two files declare the same class name, the caller's own qualifier,
  `import` or package decides which is meant, and where there is no evidence, neither is used.
- **Assignments wrapped onto the next line** — the way most formatters break a long call — now
  reach the variable they assign to, in every language.

### Changed — fewer findings we cannot stand behind
- **A helper that encodes or returns a constant no longer taints its result.** When a value
  passes through a method whose every return is a recognised output encoder or a constant, the
  calling code treats it as that encoder: an HTML-encoded value no longer reports cross-site
  scripting at an HTML output, and still reports SQL, LDAP, XPath, command and path injection,
  because an HTML encoder is not a sanitizer for those. This now also applies to helpers in
  another file, to helpers judged per parameter, and to JavaScript, TypeScript and C# helpers
  declared in the calling file. A helper the engine cannot fully see through is unchanged.
- **SQL built by string concatenation listens to the taint engine.** When every concatenated
  value is provably derived inside that function, the finding is reported one step lower with
  the proof attached. It stays visible, keeps its type and location, and still counts. Anything
  unproven, and every file where the engine cannot run, is reported exactly as before.
- **A value overwritten with a constant before use** is no longer reported, when the overwrite
  is in the same block with no branch or loop between the two statements.
- **A sanitized value concatenated with a raw one** is no longer cleared along with it. Only
  the value that passed through the sanitizer is cleared; a raw value beside it is followed to
  the sink.

### Added — an SBOM with every scan
- Every Action run now produces **`pullguard-sbom.cdx.json`** (CycloneDX) and
  **`pullguard-sbom.spdx.json`** (SPDX) alongside the existing report artifacts, built from the
  dependency inventory the scan already collected. Licence obligations map to ISO/IEC
  27001:2022 **A.5.32** in the compliance evidence.

### Fixed
- **Self-hosted server: results are accepted again when a licence needs review.** A scan whose
  licence inventory contained an entry for review carried a field the server refuses on
  principle, so posting that report returned an error and the dashboard stopped updating for
  that repository. Reports from earlier versions were unaffected, and the server's protection
  is unchanged.
- **The licence-key email no longer says the key never expires.** Keys carry an expiry date,
  and an expired key falls back to the free tier — which your scan report already warns about
  up to 14 days ahead. The email now says so.
- **The Data Processing Addendum is linked from the site footer**, not only from inside the
  privacy policy.

### Server (Enterprise) — 0.4.1

- **Fixed — false tampering alert on upgrade.** A server database created by 0.3.5 that had
  never recorded a triage decision reported "TAMPERING" on its first 0.4.0 boot and advised a
  restore from a backup that could not exist. The 0.4.0 fix covered new deployments only; a
  database left in that state by 0.3.5 is now repaired at boot when its audit log has provably
  never held a row, and says so. A log that has held rows and lost its counter is still
  reported as tampering.
- **Added — `pullguard-server token mint`.** Mint the first admin-capable managed token from
  the host that owns the data directory, so Settings (hide repositories, tokens, gate policy,
  SLA, alerts) is reachable on deployments without single sign-on. Printed once, never
  stored, bounded expiry, audit-logged.

---
## [1.5.8] — 2026-09-04

> Ships with **server 0.4.0** — see the *Server (Enterprise)* section at the end of this entry; it carries a breaking upgrade note.

### Added — coding agents can drive PullGuard (MCP)
- **`pullguard mcp`** serves a scan to any MCP-capable coding agent (Claude Code,
  Cursor, and others) over stdio, with zero extra dependencies: scan summary, filtered
  and paged finding lists, full finding detail with the taint path and remediation, and
  rule explanations. **`pullguard mcp --server <url> --token <read-token>`** offers the
  same surface backed by your self-hosted server, plus repo listing, gate status, triage
  decisions and over-SLA findings — a scoped token bounds what an engineer's agent can
  see. A **Coding Agents** documentation page carries the `claude mcp add` snippet.
- **Three tiers of agent capability, safe by default.** Read tools are always on. Safe
  actions (run a local scan, verify a fix, export) are on by default. Actions that could
  change a merge verdict or a deployment — triage decisions, accepting risk, hiding a
  repo, gate policy, SLA policy, tokens — are **off by default and not listed until
  enabled** (`mcp.dangerousTools: true` locally; a per-token *agent write* flag on the
  server). When enabled, every such call must be explicitly confirmed by the agent and is
  audit-logged as an agent actor. A security finding still needs a reason to be
  dismissed, stays visible, and is never removed from gates, whoever asks.

### Added — cross-file taint follows arguments into imported functions
- Cross-file analysis previously followed tainted data only when an imported function
  **returned** it. It now also follows request data **passed as an argument** into an
  imported function whose own parameter reaches a dangerous sink — the more common shape
  in real services — including when the request expression is written inline in the
  call. Module-scope code is analysed too. Every such finding carries a structured
  cross-file taint path.

### Added — agent and AI supply-chain threat detection
- **Unattended command execution** (`unattended_command_execution`): a committed
  devcontainer lifecycle hook, git-hook manager (Husky, lefthook, pre-commit) or npm
  install script that downloads-and-runs or otherwise executes without the developer
  asking. A plain build script in a hook is silent.
- **Lethal trifecta** (`agent_lethal_trifecta`): one agent configuration that grants
  private-data access, untrusted-content exposure **and** an external send channel. Any
  two of the three is silent.
- **Agent workspace symlink escape** (`agent_workspace_symlink_escape`): a committed
  symlink pointing outside the project in a repo that also grants its agent broad write
  or terminal scope. The link is never followed.
- **Unsafe model deserialization** (`unsafe_model_deserialization`): `torch.load`,
  `joblib.load`, Keras custom-object loads and `torch.hub.load` fire when the same
  file also fetches or unpacks the artifact it loads.
- MCP configuration drift now also notices a swapped tool **parameter** description, and
  each AI provider row in the AI-BOM lists the CVEs the dependency scan already found
  for that provider's packages (display only).
- **Two new Tier-2 zero-day rules** delivered through the signed rules channel, each
  tied to a citable advisory: a LangGraph checkpoint SQL-injection idiom
  (CVE-2025-67644) and a CrewAI code-interpreter idiom (CVE-2026-2275).

### Added — detection coverage
- **Four OWASP Benchmark categories that previously scored zero now detect:** weak
  random numbers used for security purposes (`weak_rng_crypto`), cookies set without
  `Secure`/`HttpOnly` (`insecure_cookie`), LDAP injection (`ldap_injection`) and
  XPath injection (`xpath_injection`). Overall benchmark Youden index 0.21 → 0.39 with
  the other seven categories unchanged; every changed case is verdicted in the
  committed baseline.
- **Performance findings now report on every parsed file.** Memory-leak patterns,
  synchronous I/O inside async code and unbounded pagination previously fired only on
  files the parser could not read; expect these to appear on projects where they were
  silent before. Ephemeral event emitters and framework-managed pagination are excluded.

### Added — reports and dashboards
- A **grade and score legend** (bands and category weights) on the HTML report, the CI
  step summary and the PR-comment footer, with a *How grading works* docs anchor.
- The HTML report's **SOC 2 evidence grid filters the findings table in place**; the
  markdown grids name the top `file:line` per control.
- Every finding on the report carries its **fingerprint, a copyable `.pullguardignore`
  entry and, where the scanner would honour it, the `/pullguard ignore` command**.
- Dashboard findings tables **sort on every column** (keyboard-operable, persisted), and
  both dashboards and the report offer **CSV/JSON export of the current filtered view**
  and a prefilled new-issue link per finding — all client-side, nothing uploaded.
- Each occurrence of a folded security finding gets **its own SARIF result and Check Run
  annotation** with its own fingerprint, so each has its own alert lifecycle; taint
  paths render in the HTML report as linked source→sink hops; the Check Run summary
  discloses when GitHub's annotation display ceiling hides annotations.
- **26 security finding types gained a CWE** (every agent-era rule among them) or an
  explicit, reasoned omission.

### Added — optional analyzer isolation
- `analyzerIsolation: worker` in `.driftrc.yml` runs analyzers in a separate thread
  with a real wall-clock limit, so a pathological input can be stopped without taking
  the scan down. **Off by default** (`inline`) and, when on, produces byte-identical
  findings to the inline path.

### Added — every dependency CVE now says whether your code actually uses it
- **Reachability on every CVE finding.** Each `known_cve` finding now carries a
  verdict — `reached` (a function the advisory names as affected is called),
  `imported` (production, or test code only), `declared` (in a manifest, never
  imported), `transitive` (arrived indirectly, never imported), or `unknown` — with
  the file and line that proves it. It appears on the PR-comment row, in the JSON
  report (`scaReachability`) and in SARIF, across all five ecosystems (npm, PyPI,
  Go, Maven/Gradle, RubyGems). **Nothing is hidden**: an unreached vulnerability is
  still reported in full.
- **Symbol-level accuracy where the advisory supports it.** For Go, the affected
  function names published with the advisory are used directly. For a curated set of
  other advisories whose published text names the vulnerable function, the same
  applies. Every curated entry cites the advisory it was read from.
- **Optional severity tiering (off by default).** Set `dependencies.reachabilityTiering: true`
  in `.driftrc.yml` and unreached vulnerabilities are shown one severity band lower,
  annotated with their original severity and the reason — the one place reachability
  affects the merge gate. An `unknown` verdict never lowers a severity, a
  vulnerability on the CISA KEV catalog is never lowered, and a pull request cannot
  switch the setting on for itself.

### Changed — React and front-end code reads like front-end code
- **A React component is now measured as a component, not as a function.** JSX markup
  no longer counts toward a function's length, and JSX conditional rendering
  (`{cond && <X/>}`, `{a ? <X/> : <Y/>}`) no longer counts as branching — it is
  markup, not control flow. Logic inside a JSX callback still counts, and a guard
  clause that happens to render is still a branch.
- **New finding: `large_component`.** A component that is large by markup volume is
  reported with the numbers a front-end lead can act on — JSX elements, tree depth,
  hooks and event handlers — at minor severity, rising to moderate when the component
  holds too many concerns. A component whose branches live outside the markup is still
  reported as a monolithic function, unchanged.
- **Duplication and naming follow the same framing.** Repeated markup scaffolding folds
  into one observation (a copied render tree of 25+ lines, and any duplicated logic,
  still report normally), and PascalCase components + camelCase hooks + SCREAMING_SNAKE
  constants count as one convention rather than three competing ones.
- **A large screen file** is reported at minor with its markup share stated instead of
  being treated like a long service class.
- **Class-level triage suggestion.** When one non-security finding type dominates one
  path family, the report suggests the shape of a single class-level accepted-risk
  record instead of asking for 100+ individual dismissals. Text only — nothing is
  applied.
- Nothing is suppressed by any of the above: every finding stays in the JSON and SARIF
  output. Non-React code is unchanged.

### Added — triage, config and agent workflow
- **Time-boxed suppressions.** `/pullguard ignore <fingerprint> for:30d <reason>` on a
  PR, or `pullguard ignore <fingerprint> --for 30` locally, adds a suppression that
  reverts on its own. Every report now carries a **Suppression state** section naming
  each entry that has lapsed (*"ignore expired on &lt;date&gt;"*), each entry that no
  longer matches anything and is safe to prune, and how many are still open-ended.
  `pullguard ignore --list` prints the same picture from the CLI.
- **Acknowledgements re-alert when the code changes.** An `acknowledged` finding stays
  visible and marked reviewed; if the code it was reviewed against changes, the reviewed
  badge is dropped for that scan and the finding says *"code changed since
  acknowledgement — re-review"*. Existing acknowledgements are unaffected until their
  code changes.
- **`pullguard init --full`** writes a fully-commented `.driftrc.yml` containing every
  supported option, all commented out — a complete, copy-pasteable reference generated
  from the scanner's own schema. Scans also print a one-line summary of which
  **non-default** options were applied.
- **SLA visibility.** The `sla` age-budget policy now reports itself: *"N findings
  exceed your SLA policy"* on the PR comment, the CI Step Summary and the CLI report,
  and the policy is documented at **Configuration → SLA &amp; aging**.
- **Ownership routing** (opt-in). `ownership.groupBy: owner` adds a *Findings by owner*
  table to the CI Step Summary, resolved from git blame with a `CODEOWNERS` fallback.
  @-mentions are a separate opt-in and default off.
- **`pullguard explain <fingerprint>`** prints a machine-readable **fix contract** for
  one finding — what the rule looks for, where the flow starts and ends, the invariant a
  fix must satisfy, and the reference test fixtures the rule is held to. Built for
  coding agents, and it never discloses the detection pattern itself.
- **`pullguard verify-fix <fingerprint>`** re-scans and reports whether a finding is
  fixed or still firing (exit 0 / 1 / 2). It always re-scans the whole project and
  states that scope, and it refuses to return a verdict — rather than reporting
  "fixed" — when the re-scan ran at a lower licence tier than the report it is checking,
  or produced no findings at all on a project that had them.
- **Sample output before install.** The Getting Started page now links a live scanned
  pull request on the public playground repository and shows a real PR comment.
- **The JSON report now carries each finding's `fingerprint`.** Every triage command
  takes one (`pullguard ignore`, `explain`, `verify-fix`, and the `/pullguard ignore`
  PR comment) and the documentation has always pointed at this field — it was missing,
  so the documented workflow could not be followed from a JSON report. Additive: no
  existing field changed.


### Changed
- **`pullguard scan` now honours a reviewed false-positive override in its exit code.**
  A security finding you have marked `false_positive` in `.pullguardignore` (with a
  reason, under CODEOWNERS review) already stopped blocking `pullguard gate`; bare
  `pullguard scan` still failed the build on it, so the documented escape hatch was
  unreachable unless you also ran the gate. The two commands now agree. Only the exit
  code changes — the finding is still counted, still graded, and still rendered in the
  report, PR comment, HTML report, JSON and SARIF, tagged with your reason. A critical
  **without** an override still fails the build, and an override stops applying the
  moment the reviewed code changes.

### Added — documentation
- **Language support matrix.** A new section in the docs says plainly, per language,
  which kind of analysis you get: full taint-flow analysis (attacker-controlled input
  traced to a dangerous sink, across functions and files), calibrated pattern rules, or
  dependency CVE scanning — and where a language gets one but not the others. Ruby and
  PHP are named explicitly as pattern-only. The table is checked against the shipped
  analysis catalogs by a test, so it cannot drift from the product.
- **Evidence tiers explained.** Every security finding carries an evidence tier saying
  how it was established — dataflow-proven flows are labelled `taint-flow` or
  `interprocedural`, calibrated pattern rules are labelled `pattern` — and the reports
  documentation now covers what each means and how to triage with it. The tier is
  display metadata: it never changes a severity, a grade, or whether a finding blocks.
### Fixed — detection precision
- **Django REST Framework**: a view guarded with `permission_classes`
  (`IsAuthenticated`, `IsAuthenticatedOrReadOnly`, `IsAdminUser`, or your own
  permission class) is no longer reported as missing authentication. The guard lives
  on the view while the route lives in `urls.py`, and the two are now resolved
  together — including the project-wide `DEFAULT_PERMISSION_CLASSES` setting. A view
  declaring `AllowAny`, or declaring no permission classes at all, still fires.
- **URLconf mounts**: `url(r'^api/', include('app.urls'))` is no longer reported as an
  unauthenticated route — it is a mount, and the routes it mounts are still judged on
  their own.
- **Self-registration endpoints**: `POST /users` whose handler creates the account is
  now treated like `/signup` and `/register` — a caller with no account cannot
  authenticate. `GET /users`, `DELETE /users/:id`, and a `POST /users` that does
  something other than create an account all still fire.
- **Liveness and readiness probes**: a shallow `/healthz`, `/livez`, `/readyz` or
  `/ping` that answers with a constant is no longer reported as missing
  authentication — orchestrators cannot present credentials. A health endpoint that
  reads a database, returns configuration or exposes build metadata still fires, and
  `/metrics` and Spring actuator endpoints are unchanged.
- **SSRF**: browser-side `fetch`/XHR wrappers are no longer reported as server-side
  request forgery — a browser calling its own API is not the vulnerability. Server
  code, and server-side rendering (Next.js `getServerSideProps` and route handlers,
  Remix loaders, Nuxt event handlers), still fire. The rule also no longer mistakes an
  ordinary local named `params`, `body` or `queryString` for a request value.

- **Literal-valued conditionals**: a value chosen by a comparison against request data
  (`const dir = req.query.x === 'a' ? 'uploads' : 'tmp'`) is no longer treated as
  attacker-controlled — the comparison operand does not flow into the result. A
  conditional whose branch **is** request data still fires.
- **SQL in template literals**: a query whose only interpolations are literals, a
  literal-only conditional, a constant clause list or a provably safe identifier is no
  longer reported as SQL injection, and an HTML `<select>` element is no longer
  mistaken for a SQL `SELECT`. A template interpolating a request value still fires.
- **LDAP and XPath bound arguments**: the bound-arguments and variable-binding idioms —
  the exact remediation the guidance recommends — are no longer reported as injection.
- **Dev containers**: a `.devcontainer/Dockerfile` is not a deployed image and no
  longer produces IaC findings.

### Fixed — detection coverage
- **Java query builders**: a raw SQL clause built by string concatenation is now
  reported even when it shares a predicate with a safely bound comparator — previously
  one bound comparator could silence the whole predicate, so the same injection was
  reported or not depending on what sat beside it.

### Fixed — scan performance
- A large route file importing many rendering functions no longer makes cross-file
  analysis take minutes — one real-world shape tripled scan time; it now scans in
  seconds with identical findings.
- **Dependency freshness on a slow registry.** On a Maven-heavy project, a slow
  registry day could push the freshness check past the scanner's per-analyzer time
  limit, dropping that analyzer's findings and marking the scan incomplete. The check
  now bounds its own lookups: what it could measure is reported, the rest is counted
  as unmeasured, and the scan completes normally.

### Fixed — compliance mappings
- Three EU AI Act control titles (Articles 10, 15 and 50) corrected against the enacted
  Regulation text, in the signed rules bundle and the air-gapped fallback.

### Changed
- **Shell quoting lint**: an unquoted `$VAR` expansion (the shellcheck SC2086 class) is
  now reported as `shell_unquoted_expansion` under the quality category instead of
  `command_injection` under security. It is a real robustness defect, but not an
  injection — reporting it as one inflated grades and compliance evidence and made
  genuine command injection harder to find. The finding, its severity and its
  remediation are unchanged and it still appears in every report, SARIF file and
  dashboard. **If you filter or gate on `command_injection`, add
  `shell_unquoted_expansion` to keep seeing these.**

### Server (Enterprise) — 0.4.0

> **Upgrade note (breaking):** the legacy `PULLGUARD_SERVER_INGEST_TOKENS` variable now
> refuses to boot. Move to scoped tokens — `PULLGUARD_SERVER_INGEST_TOKEN_SCOPES="org/*=<token>"`
> — or bridge one release with `PULLGUARD_SERVER_ALLOW_UNSCOPED_INGEST=true` (warns at
> boot). SAML deployments must be on TLS. An SSO login in flight across the upgrade is
> refused once. We recommend `PULLGUARD_SERVER_AUDIT_ANCHOR_FILE` on a volume separate
> from the data directory.

- **Finding lifecycle.** A finding absent from the next scan becomes **resolved**; one
  that returns becomes **reopened** — nothing is deleted, the stored scans remain the
  evidence. Surfaced as KPI cards, a per-scan *Fixed* column, a resolved/reopened strip
  and read-only fields on the query API. `PULLGUARD_SERVER_RESOLVE_AFTER_SCANS`
  (default 1) requires N consecutive absences so a truncated scan does not look like a fix.
- **Accept risk** — a fourth triage status with a **required expiry (1–365 days) and
  reason**; on expiry the finding is active again and stops counting as triaged. Also
  available in `.pullguardignore` and via `/pullguard ignore … status:accepted_risk for:<N>d`.
- **Dashboard decisions effective in CI** (opt-in `server.pullTriage: true`): a scan
  pulls the repo's live decisions and applies them with `.pullguardignore` semantics; a
  committed `.pullguardignore` entry always wins; any error fails open to "no decisions"
  with a visible notice; the server never writes into your repository.
- **Inline, labelled triage actions** (Acknowledge / False positive / Won't fix / Accept
  risk until …) on the triage page and the repo dashboard; viewers see the same controls
  disabled with a reason, never hidden. **Bulk and class-level triage**: select many rows
  for one decision, or record one decision for a finding type over a path pattern with an
  expiry (never for always-surfaced security types).
- **Settings area**: per-repo **Hide/Unhide** with a note (display only — never a gate
  input); **scoped token management** (create, rotate, revoke, expire — admin-only,
  shown once, audit-logged); per-repo **gate policy** (versioned, echoed on the PR);
  **retention** and **backup** (consistent snapshot with checksum sidecar into
  `PULLGUARD_SERVER_BACKUP_DIR`, documented restore).
- **SLA on the server**: over-SLA strip and ageing buckets; **scan-to-scan diff view**
  (new, resolved, reopened, severity moves).
- **250+ repositories**: the header's repo pills are replaced by a searchable,
  keyboard-navigable repo switcher on every page, plus portfolio search and pagination.
- **Query API for integrations**: `GET /api/v1/openapi.json` (OpenAPI 3.1, generated from
  the live routes), a v1 additive-only stability statement, an *Integrate with the Query
  API* guide, and findings now carry `type` and `findingKey` so they are actionable.
- **Agent-to-agent** via `pullguard mcp --server` (see the scanner entry above): reads
  bounded by token scope; writes only behind a per-token *agent write* flag, confirmed
  per call and audit-logged.
- **Alerts** (off by default): HMAC-signed webhooks to Slack/Teams-style receivers on new
  critical, SLA breach, reopened and gate flips — exact-host allowlist, HTTPS only, bounded
  retries with a dead-letter list; agent credentials can never administer them.
- **Audit viewer** with in-UI hash-chain and anchor verification, an **auditor** role
  (read-only, cannot triage), and a quarterly **compliance evidence pack** export
  (worst-in-period and end state; "evidence supporting", never "compliant").
- **Assignment and "my findings"**: owner suggestions from blame and the CODEOWNERS map
  the scan carried (the server never reads your repo), `/me` and
  `GET /api/v1/me/findings`, optional mentions.
- **Team scope** from IdP group claims; **signed, expiring read-only share links** (off by
  default, revocable, audited).
- **Probes**: `/healthz` and `/readyz` documented, plus a documentation-completeness
  check so every route, environment variable and CLI subcommand is documented before it
  ships. A first-boot *expected errors* section in the quick start.
- **Fixed**: a fresh deployment restarted before any triage no longer raises a false
  tampering alert; blame-based ownership was silently absent from the query API; reads
  returned 503 when only scoped tokens were configured; the shared read guard named the
  wrong resource in refusals.

## [1.5.7] — 2026-08-29

Follow-up to the v1.5.6 customer validation plus a proactive precision/recall sweep of the
scanner against its own codebase. No breaking changes. Verified with **no loss of
true-positive coverage** (labeled benchmark byte-identical per category; vulnerable-app
sweep byte-stable per finding type; clean-OSS false-positive beds identical per finding).

### Fixed — detection
- DOM XSS: appending to an element's **own** markup (`el.innerHTML = el.innerHTML +
  '<span>…</span>'`) is no longer reported — a v1.5.6 regression in the same rule. All
  spellings of the operation agree; and a self-append that **also** concatenates a request
  or record value (`el.innerHTML = el.innerHTML + '<td>' + user['first_name']`) is reported
  again in the expanded form (the first fix had gone silent there). Copying one element's
  HTML into another stays silent; a text read re-rendered as markup now fires.
- Timing-attack detection: a "confirm your password" compare is suppressed when the
  request handle is **inherited** from a base class rather than declared in the file — the
  production shape behind the original report. A stored secret compared against a request
  DTO field of the same name (`this.apiKey.equals(request.apiKey)`) still fires.
- Findings from the language-specific rule set are anchored at the **actual match**
  rather than the first occurrence of the matched text in the file (reports, SARIF
  click-to-source, PR annotations).
- Python `builtins.exec(…)` / `__builtins__.eval(…)` and JavaScript indirect eval
  (`window.eval`, `globalThis.eval`, `self.eval`) are reported as the builtin they are;
  other receivers (`session.exec`, `parser.eval`) stay silent.
- Clean-code false positives removed: hand-rolled CSRF token checks (`x-csrf-token` +
  compare) are recognised as CSRF protection; a handler that verifies its own bearer / API
  token (`checkIngestToken(…)`, `requireAdminWrite(req, …)`) is recognised as guarded, and
  a comment block between a route and its guard no longer hides the guard (while a
  section-header comment between two routes still keeps them separate); a slash that opens a
  string literal is no longer read as a regex in JavaScript (PHP/Python string regexes are
  unaffected); a regex `.exec(…)` in a callee is no longer a cross-file command-injection
  sink; a removed export is reported only when an importer actually references it;
  identifiers inside comments never count toward naming-convention drift; SQLModel
  `session.exec(…)`, Perl POD/backtick-in-comment, `JSON.parse(body)` on a response body,
  Perl `t/` test trees (Perl files only), and value-object `equals(Object)` overrides no
  longer fire.

### Changed — release gates
- Every corpus fixture is re-verdicted under semantics-preserving rewrites (line endings,
  compound assignment, a request handle declared ⇄ inherited, an inline source hoisted into
  a local, comment insertion, every code line echoed inside a comment) and must produce the
  identical finding types and severities.
- The clean-code false-positive gate and the vulnerable-app sweep refuse a scan that ran
  below the Enterprise tier, crashed, or in which every baseline finding vanished at once.

### Fixed — self-hosted server image
- Base image pinned by digest; a `HEALTHCHECK` against `/healthz` that honours `PORT` and
  `PULLGUARD_SERVER_HOST`.

## [1.5.6] — 2026-08-28

Patch wave driven by an external customer validation of v1.5.5 (every finding
reproduced against the released image before being fixed).

### Fixed — detection
- Command injection: a request value passed **directly** as the argument of a
  Node shell call (`execSync(req.query.cmd)`, `exec(req.body.command, …)`,
  `spawn(req.params.bin, …)` and the `child_process.`-prefixed forms) is now
  reported at CRITICAL, matching the already-detected "bind to a local first"
  and string-concatenation spellings of the same vulnerability. Previously the
  shortest spelling was silent. Scoped to genuine request-input accessors and
  the shell verbs only — regex `.exec(...)`, logger/database `.exec…` methods,
  and server-configuration reads through the request object do not fire.
- Timing-attack detection now follows where a value CAME FROM rather than what
  the variable is called: a value read from a secret store or a password/token
  getter compared against request input is reported even when the variable
  has a neutral name (`stored`, `expected`), and a call receiver such as
  `user.getPassword().equals(request.getParameter(…))` is reported too — a
  class of genuine timing oracles that a rename could previously hide. Two
  false-positive shapes are closed at the same time: a "confirm your password"
  compare read through a framework request object (a `WebRequest` field, with
  the parameter names in constants) and a confirm pair of method parameters
  (`newPassword1` / `newPassword2`) no longer fire. A non-secret value compared
  against request input, or a secret compared against a public literal, stays
  silent.
- Timing-attack detection now behaves identically on files with Windows
  (CRLF) line endings. Previously the "both values are the request's own
  input" suppression never applied to CRLF files, so confirm-your-password
  compares in Windows-checked-out Java code were reported as timing oracles.
- Cross-method DOM XSS is now detected when the fetched content reaches the
  DOM-writing method through a promise `.then()` callback
  (`this.get(url).then((body) => { el.innerHTML = body; })`, and the
  three-method fetch → link → DOMParser/innerHTML module shape). Previously
  only the `await`/assignment spelling of the same chain was reported — the
  `.then()` spelling was silent. The existing precision rules carry over: a
  fetched body handed to `res.send` is still treated as proxy plumbing, and a
  parameterless callback binds nothing.

## [1.5.5] — 2026-08-27

Hardening release from two independent adversarial review passes over v1.5.4 plus a cross-release regression hunt. No breaking changes. Verified with **no loss of true-positive coverage** (labeled benchmark byte-identical per category; the vulnerable-app sweep's single delta is one hand-verified true positive gained; the clean-OSS false-positive bed was re-baselined with every finding individually verdicted).

### Security — scan-integrity hardening
- **A pull request can no longer weaken its own scan through configuration changes in the diff**: in-diff reductions to scan depth or file caps, dev-tooling severity demotions, custom sanitizer declarations, and offline-database redirects are all neutralized (extending the protection that already covered exclude patterns), and every neutralization is stated in the scan-integrity banner. Depth-limited scans are marked partial instead of presenting as complete.
- Generated dashboards and PR comments can no longer be attacked through committed history files or echoed configuration values — all such inputs are strictly validated and escaped at render.
- Repo-bound license keys are enforced outside GitHub Actions as well; repository comparisons are case-insensitive everywhere; Team-plan repository registrations are validated to a plausible `owner/repo` shape before being stored.
- `/pullguard ignore` refuses to act on fork pull requests.

### Detection — recall restored and widened
- Timing-attack detection no longer goes quiet where a stored secret is compared against request input in modern arrow-function handlers, after a nearby re-assignment, or through a field write — while the password-confirmation shapes that are provably not an oracle stay suppressed, now including destructured request reads.
- The `innerHTML +=` append idiom is detected as an XSS sink everywhere; HTML string literals containing quoted attributes are no longer missed in concatenation; `outerHTML` and `insertAdjacentHTML` gained the same coverage, including across function boundaries.
- Raw-SQL predicate injection is detected through wrapper calls (`where(not(condition(...)))`-style composition), and a literal-carrying wrapper can no longer resurrect a previously fixed query-builder false positive.
- A sanitized value concatenated or overwritten with fresh user input is treated as unsafe again; multi-line shell invocations carrying request input are detected again; Java SQL injection through inline-chained JDBC calls is detected again; `.mts`/`.cts` TypeScript modules are fully analyzed.
- Cursor Project Rules files (`.cursor/rules/*.mdc`) and current Copilot / Claude Code instruction formats are now covered by the agent-instruction rules; MCP configuration files are recognized by content, so renaming one no longer evades change tracking.

### Added
- **`PULLGUARD_OFFLINE=true`** — a single switch for air-gapped operation: signed embedded rule catalogs, local-only CVE lookups (marked unverified when absent), no registry sweep, and no report leaves the runner. The privacy and sovereignty documentation now enumerates every external call.
- **CVSS v4.0 support** in CVE severity classification using the official FIRST scoring algorithm.

### Fixed — accuracy
- SOC 2 evidence cites the correct AICPA criteria (change management under CC8.1; fraud-risk under CC3.3) and PCI DSS 4.0 known-vulnerability evidence cites Requirement 6.3 — with an automated gate verifying every shipped control id against the published standards so numbering drift cannot recur.
- Routes protected by `router.use(passport.authenticate(...))` are no longer flagged as missing authentication.
- The documented `thresholds:` configuration block now steers the analyzers; dependency-freshness no longer recommends date-versioned artifacts or pre-releases as upgrade targets; large-repository scans no longer degrade in throughput past 1,000 files.

## [1.5.4] — 2026-08-14

Precision, anti-evasion, and provenance release. No breaking changes; new configuration is additive and opt-in. Verified with **no loss of true-positive coverage** (labeled benchmark byte-identical per category, vulnerable-app sweeps per-finding verdicted, clean-OSS false-positive bed unchanged).

### Added
- **AI-authorship provenance now records three evidence layers** (opt-in via `aiProvenance.enabled`): marker strings in source comments; the same markers in git commit trailers — where AI coding tools actually write their disclosure by default; and C2PA / Content Credentials provenance manifests on committed image assets, reported as present (unverified — signature validation is on the roadmap). Evidence for EU AI Act Article 50 transparency, never authorship proof; absence of a mark proves nothing.
- **Vendored findings can stay visible** (opt-in `vendored.visibility: info`): findings in vendored/bundled third-party code are kept at informational severity — annotated with their original severity, excluded from the grade — instead of the default drop. Committed credentials keep full severity either way.
- **Deterministic dependency-freshness**: the aggregate is no longer skewed by transient registry failures, and an opt-in offline mode makes identical-input scans byte-identical with no network calls.
- Dev-tooling tiering accepts any-depth path patterns (`**/provision`).

### Fixed — scanner integrity (anti-evasion)
- **A pull request can no longer remove its own files from security scanning** by editing scan-exclusion configuration in the same diff.
- **Renaming an agent/MCP configuration file no longer evades detection** — agent-config rules key on the file's content shape, not its name.
- **A padded line can no longer suppress security findings** — a single over-long line used to classify the whole file as a minified bundle and hide findings on ordinary lines elsewhere in the file. Genuine minified bundles remain excluded.
- The Check Run integration's authorization was hardened.

### Fixed — detection accuracy
- Framework query-builder and prose false positives removed across chained verbs, structured predicate builders, and validation-message text, with the high-recall pattern retained (its unique true-positive contribution was measured before any change).
- `timing_attack` no longer fires on password-confirmation compares regardless of how the fields are named — a compare whose operands are both the current request's own input is not a timing oracle. Genuine oracles (stored-secret, HMAC, basic-auth compares) are pinned as permanent must-fire tests.
- `insecure_crypto` down-ranks MD5 used for content digests even when the evidence lives in the method signature; password and signing-key uses keep full severity.
- Vendored recognition covers `_resource` asset trees; `.env` findings no longer report CDN/content-hash paths as credentials; duplication findings fold framework boilerplate; findings name the method actually matched instead of a sibling method absent from the line.

### Fixed — detection coverage
- **Fetched remote content written into the DOM is now detected as XSS**, including the real-world shape: fetch in one method, a DOMParser round-trip and `innerHTML` write in another. Server-side proxying of upstream responses is deliberately not flagged.
- `innerHTML` assignments of HTML built by string concatenation are flagged exactly like their template-literal equivalents.
- RAG prompt-isolation detection follows retrieved content through helper functions that assemble the prompt, across ingestion sources.
- **Sanitizers now clear only the vulnerability classes they actually mitigate**, everywhere in the analysis — an HTML encoder no longer hides a SQL-injection or request-forgery flow, whether the sanitizer appears inline, in an earlier assignment, or across function boundaries.
- Multi-line `return` statements are now analyzed across function boundaries in every supported language — previously a silent cross-function blind spot.

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
