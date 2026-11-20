# Security Policy

## Scope of this tool

Palisade is an **authorized** vulnerability-assessment orchestrator. It is designed to
refuse to scan any target not on an explicit allow-list. Do not use it against systems
you do not own or lack written permission to test. See `LICENSE` for the use notice.

Project home: https://www.opey.org/opeyit/palisade/

## Reporting a vulnerability in Palisade itself

If you find a security issue **in Palisade's own code** (for example, a way to bypass the
scope enforcer, a command-injection path, or unsafe handling of input files):

1. **Do not** open a public issue for it.
2. Email the maintainer at `steveoppenheim@opey.org` with a description and reproduction
   steps.
3. Expect an acknowledgement within a few business days.

Please give us reasonable time to remediate before public disclosure.

## Known issues

This project ships with a published internal audit and a tracked list of known
limitations: see [`KNOWN_ISSUES.md`](KNOWN_ISSUES.md) and
[`docs/audit/`](docs/audit/). Several scanning-engine defects are **open**; review them
before relying on Palisade for a real engagement.
