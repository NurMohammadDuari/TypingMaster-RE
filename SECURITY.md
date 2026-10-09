# Security Policy

## About this repository

This is a **public research repository** containing a reverse-engineering
study of a third-party Windows application. It contains no binaries and no
runtime; it is documentation and analysis only.

## Reporting a vulnerability

If you believe you have found a security problem in **this repository's own
content** — for example in a script under `analysis/` or in the CI workflows —
please report it privately.

- **Preferred:** open a private security advisory via
  **Security → Advisories → Report a vulnerability** on GitHub.
- **Alternative:** contact the maintainer directly at
  [@NurMohammadDuari](https://github.com/NurMohammadDuari).

Please do **not** open a public issue for security matters.

### What to include

- A description of the problem and its impact.
- Steps to reproduce, or the exact file and line involved.
- Any suggested fix, if you have one.

### What to expect

- Acknowledgement as soon as reasonably possible.
- An assessment and, where warranted, a fix with credit if you wish.

## Out of scope

The following are **not** in scope for this repository:

- Vulnerabilities in **TypingMaster** itself — those belong with the product's
  vendor, not here.
- Vulnerabilities in **Wine** or the **Freedesktop/Flatpak runtime** — report
  those to the respective upstream projects (https://www.winehq.org/,
  https://gitlab.freedesktop.org/). They are not bundled here.
- Any request to bypass, crack, or defeat the product's licence check. Such
  requests will be declined; see `CONTRIBUTING.md`.

## Third-party runtimes

This repository does **not** bundle Wine, the Freedesktop platform runtime, or
any other third-party binaries. Vulnerabilities in those projects should be
reported to the respective upstreams (https://www.winehq.org/,
https://gitlab.freedesktop.org/).

## Secrets

Do not commit credentials of any kind. If you spot a secret in the history,
report it privately so it can be rotated and purged.
