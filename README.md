# PullGuard GitHub Action

Code quality, security & compliance on every pull request.

## Quick Start

```yaml
# .github/workflows/pullguard.yml
name: PullGuard
on: [pull_request]
permissions:
  contents: read
  pull-requests: write
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: pullguard-dev/pullguard-action@v1
```

That's it. 14 analyzers run on every PR — no license key, no configuration, no account needed.

## Unlock Pro / Team / Enterprise

Add your license key as a repo secret to unlock paid analyzers:

```yaml
      - uses: pullguard-dev/pullguard-action@v1
        with:
          license-key: ${{ secrets.PULLGUARD_LICENSE_KEY }}
          fail-on-severity: critical
```

## Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `license-key` | Unlocks Pro (31 of 32), Team (all 32 incl. custom-rules + SOC 2), or Enterprise (same bundle + SSO/SLA/audit via contract) | Empty (free tier, 14 analyzers) |
| `fail-on-severity` | Fail workflow at this severity: `info`, `minor`, `moderate`, `major`, `critical` | Empty (never fail) |
| `hourly-rate` | Developer hourly rate for cost estimation | `150` |
| `config` | Path to `.driftrc.yml` config file | Auto-detected |
| `path` | Path to scan | `.` (repo root) |

## Outputs

| Output | Description |
|--------|-------------|
| `score` | Drift score (0-100, lower is better) |
| `grade` | Drift grade (A-F) |
| `findings` | Total finding count |

## Pricing

### Free — $0
14 analyzers, unlimited public repos + 1 private repo. Core code quality + supply-chain hygiene: complexity, dead code, duplication, monolithic files, naming, nesting, imports, module systems, file structure, error handling, test patterns, API consistency, **dangerous tracked files** (.env, id_rsa, etc.), **repo hygiene** (SECURITY.md, CODEOWNERS, LICENSE, CI workflow presence).

### Pro — $29/mo (1 private repo)
Solo developers & single projects. Free + SQL injection, XSS, SSRF, command injection, path traversal, hardcoded secrets, insecure crypto, prototype pollution, ReDoS, dependency CVEs, type coverage, architecture violations, API contract, AI-generated code detection, cost-of-change estimation, CORS, CSP, NoSQL injection, **Dockerfile IaC** (runs-as-root, :latest, ADD-from-URL, embedded secrets), **git-history secret scan**, **GitHub Actions security** (pwn-request, script injection, unpinned actions, excessive permissions). 31 of 32 analyzers.

### Team — $99/mo (10 private repos, up to 20 contributors)
Startups & mid-sized engineering teams. Everything in Pro plus **all 32 analyzers** (adds custom rules — YAML policies, custom taint sources & sinks) and **SOC 2 security evidence** reports (CC3.1, CC3.2, CC6.1, CC6.8).

### Enterprise — Contact us (annual contract)
Unlimited private repos & contributors. Everything in Team plus SSO (SAML / OIDC), audit-log export, dedicated Slack channel, priority support (4h SLA), air-gapped deployment. Contact [hello@pullguard.dev](mailto:hello@pullguard.dev).

## Contact

hello@pullguard.dev | [pullguard.dev](https://www.pullguard.dev)
