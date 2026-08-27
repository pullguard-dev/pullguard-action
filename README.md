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
| `license-key` | Unlocks Pro (44 of 46), Team (all 46 incl. custom-rules + SOC 2), or Enterprise (same bundle + SSO/SLA/audit via contract) | Empty (free tier, 14 analyzers) |
| `fail-on-severity` | Fail workflow at this severity: `info`, `minor`, `moderate`, `major`, `critical` | Empty (never fail) |
| `hourly-rate` | Developer hourly rate for cost estimation | `150` |
| `config` | Path to `.driftrc.yml` config file | Auto-detected |
| `path` | Path to scan | `.` (repo root) |
| `image-pin` | Which scanner image to run. `1` (default) tracks the newest stable **v1.x release** — current with security fixes, but never the per-merge churn of `:latest`. Set `latest` for the bleeding-edge image rebuilt on every merge to main. Pin a release tag (e.g. `v1.0.0`; preserved forever) or a `sha256:<hex>` digest to freeze one exact version for change-controlled or air-gapped CI. | `1` |

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
Solo developers & single projects. Free + SQL injection, XSS, SSRF, command injection, path traversal, hardcoded secrets, insecure crypto, prototype pollution, ReDoS, dependency CVEs, type coverage, architecture violations, API contract, AI-era risk detection (hallucinated dependencies), cost-of-change estimation, CORS, CSP, NoSQL injection, **Dockerfile IaC** (runs-as-root, :latest, ADD-from-URL, embedded secrets), **git-history secret scan**, **GitHub Actions security** (pwn-request, script injection, unpinned actions, excessive permissions). 44 of 46 analyzers.

### Team — $99/mo (10 private repos, unlimited contributors)
Startups & mid-sized engineering teams. Everything in Pro plus **all 46 analyzers** (adds custom rules — YAML policies, custom taint sources & sinks) and **SOC 2 security evidence** reports (8 controls: CC3.1, CC3.3, CC4.1, CC6.1, CC6.2, CC6.7, CC6.8, CC8.1).

### Enterprise — from $15k/yr (annual contract)
Up to unlimited private repos & contributors, priced in repo bands (see [pullguard.dev](https://pullguard.dev/#enterprise)). Everything in Team plus SSO (SAML / OIDC), a hash-chained admin-exportable audit trail, dedicated Slack channel, priority support (4 business-hour response SLA), air-gapped deployment. Contact [hello@pullguard.dev](mailto:hello@pullguard.dev).

## Verifying signed releases (optional, supply-chain assurance)

PullGuard signs release tags (`v[0-9]+.[0-9]+.[0-9]+`) and the rolling `v1` ref with the operator's GPG key. Customers with compliance requirements around supply-chain attestation can verify cryptographically:

```bash
# One-time: fetch the operator public key (fingerprint below)
gpg --keyserver keys.openpgp.org --recv-keys E70D94576B869369ABF83866541382ADC561473B

# Per-verification:
git clone --depth=1 --branch=v1 https://github.com/pullguard-dev/pullguard-action /tmp/pga
cd /tmp/pga
git verify-tag v1   # exit 0 if signed by the trusted key
```

The operator's GPG fingerprint — `E70D 9457 6B86 9369 ABF8 3866 5413 82AD C561 473B` — is published in [pullguard.dev/.well-known/security.txt](https://www.pullguard.dev/.well-known/security.txt) (RFC 9116). GitHub's `Verified` badge on the action repo's Releases page confirms tags are signed by the operator's uploaded key automatically, without needing to fetch the key.

For air-gapped / high-assurance environments, combine `git verify-tag` with the `image-pin: sha256:<hex>` input above for end-to-end supply-chain assurance: signed source ref → known image digest → reproducible scan.

## Contact

hello@pullguard.dev | [pullguard.dev](https://www.pullguard.dev)
