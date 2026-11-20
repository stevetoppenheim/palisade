# Changelog

All notable changes to this project are documented here. Format based on
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow
[SemVer](https://semver.org/).

## [1.0.0] - 2026-11-20

First public release.

### Added
- Authorization-gated orchestrator: scope enforcer (default-deny, deny-wins, date
  window), append-only JSONL audit log, dry-run mode.
- Input parsers for `.txt`, `.csv`, `.xlsx`, `.xml` (including reusing an nmap `-oX` file),
  using defusedxml when available.
- Tool wrappers: nmap (`-sV`, NSE vuln), masscan/naabu port discovery, httpx, nuclei,
  nikto, testssl.sh, optional Greenbone/OpenVAS hook.
- Per-IP reports (Markdown, HTML, JSON) with a Scan coverage section, a verification
  caveat, and a roll-up summary.
- `selfcheck` self-audit command; 33 unit tests across scope, inputs, tool-output
  parsers, reporting, and audit log.
- Defensive honeypot companion (Cowrie, OpenCanary compose).
- Published internal audit (`docs/audit/`) and `KNOWN_ISSUES.md`.

### Fixed (from the internal audit)
- F-01: masscan trailing-comma JSON output now parses; parse errors are surfaced.
- F-02: NSE CVE extraction matches the `CVE:CVE-...` format.
- F-03: `scope.max_rate_pps` is enforced as a hard cap on masscan, naabu, and nuclei.
- F-04: the discovery step parses and reports host status instead of discarding it.
- F-05: testssl WARN maps to low severity.
- F-06: parser errors are logged, not swallowed.
- F-07: XML input uses defusedxml when installed.
- R-01: reports disclose scan coverage so an empty result is not read as clean.
- R-02: reports carry a verification caveat; findings have an optional remediation field.
- R-03: findings are de-duplicated across tools.

### Notes
- Documentation was corrected to remove capabilities that were not implemented
  (katana, ffuf, whatweb); these are listed under Roadmap.
- The internal audit is a self-review, not an independent audit.
