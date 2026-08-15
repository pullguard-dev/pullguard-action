# Security Policy

## Supported Versions

We ship security fixes for the latest published Docker image tag
(`ghcr.io/pullguard-dev/pullguard:latest`) and the current major version
of the `@pullguard/cli` npm package.

## Reporting a Vulnerability

Email **security@pullguard.dev** with:

- A description of the vulnerability and potential impact.
- Steps to reproduce, ideally with a minimal reproduction repo.
- Your environment: PullGuard version, OS, Node.js version.
- Whether you've disclosed the issue elsewhere.

We will acknowledge receipt within **48 hours** and provide an initial
assessment (confirmed / triaged / out-of-scope) within **5 business days**.

Please do not open public GitHub issues for security vulnerabilities.

## Scope

**In scope:**
- The PullGuard scanner engine (`@pullguard/cli`, Docker image).
- The PullGuard GitHub Action (`pullguard-dev/pullguard-action`).
- The Cloudflare Worker API at `pullguard.dev/api/*`.
- License-key generation and validation logic.

**Out of scope:**
- Issues in third-party dependencies (report those to the upstream project).
- Denial-of-service via scan timeouts on pathological inputs
  (PullGuard is opinionated about bounded scan durations).
- Findings in code scanned BY PullGuard (those are product output, not
  security issues in PullGuard itself).

## Disclosure Timeline

- We aim to release a fix within **30 days** of confirmation for
  critical/high severity issues, **90 days** for moderate/low.
- We coordinate disclosure with the reporter — public disclosure typically
  happens alongside the patched release.
- Credit is given to reporters (unless they prefer anonymity).

## Bug Bounty

We do not currently offer a paid bug bounty program. Security reports are
welcome and appreciated — contributors are credited in release notes.
