# Security Policy

## This project is intentionally vulnerable

RamziRange9 is a deliberately insecure web application built as a target for
authorized penetration testing and security training. **Every vulnerability in
this repository is present on purpose.**

Please do **not** open security reports for the vulnerabilities it ships —
SQL injection, command injection, path traversal, XXE, SSRF, XSS, exposed
credentials and the rest are the entire point of the project.

## Credentials in this repository are fake

All API keys, tokens and private keys are non-functional:

- The AWS secret access key is Amazon's own published example string.
- Every other value carries `RR9`, `Fake`, `EXAMPLE` or `NotReal` markers.
- Webhook and registry hostnames use reserved `.test` domains.
- Only the key *prefixes* are real (`AKIA`, `ghp_`, `glpat-`, `sk_live_`, ...),
  which is what makes them detectable by scanners without authenticating
  anywhere.

Automated secret scanning will flag this repository. That is expected.

## What IS worth reporting

- A vulnerability in the *deployment* that could affect a host outside the
  container boundary in a way not already documented.
- A real, working credential that has accidentally been committed.

Open an issue for either.

## Safe deployment

- Isolated lab network only. Never internet-facing.
- Treat the container as compromised by design (`/admin/exec` runs as root).
- Tear it down when you are finished: `docker compose down -v`
