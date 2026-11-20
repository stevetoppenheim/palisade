# Known issues and remediation status

This project published an internal audit ([`docs/audit/`](docs/audit/)) and tracks every
finding here. All 15 audit findings were fixed or addressed in v1.0.0.

Legend: FIXED, ADDRESSED (mitigated, minor follow-up possible), OPEN.

## Resolved in v1.0.0

Code defects:

- FIXED F-01 (High): masscan `-oJ` trailing-comma output now parses via a lenient JSON
  loader, and parse errors are logged instead of swallowed. Test:
  `test_masscan_trailing_comma_now_parses`.
- FIXED R-01 (High): every per-IP report has a Scan coverage section listing which tools
  ran, were skipped (not installed), disabled, or errored, so an empty result is never
  read as a clean host. Tests in `tests/test_report.py`.
- FIXED F-02 (Med): NSE CVE extraction uses a regex that matches the `CVE:CVE-...` format.
- FIXED F-03 (Med): `scope.max_rate_pps` is now a hard cap applied to masscan, naabu, and
  nuclei rate flags. Tests: `test_runner_rate_cap`, `test_masscan_uses_capped_rate`.
- FIXED F-04 (Low): the discovery step parses the ping result and reports host status.
- FIXED F-05 (Low): testssl WARN maps to low severity, not unknown.
- FIXED F-06 (Low): parser errors are recorded to the audit log and stderr.
- FIXED F-07 (Info): XML input uses defusedxml when installed.
- FIXED R-03 (Low): findings are de-duplicated across tools; the coverage section lists
  the tools that ran.
- ADDRESSED R-02 (Med): every report carries a verification/false-positive caveat and a
  Finding now has an optional remediation field. Per-finding tailored remediation text is
  still authored by the operator, not generated.

Documentation integrity:

- FIXED M-01: katana, ffuf, whatweb removed from Features and listed under Roadmap.
- FIXED M-02: nuclei template updating described as a manual, scheduled step.
- FIXED M-03: rate limiting implemented (F-03); wording corrected.
- FIXED M-04: parser tests added; "unit-tested" scope stated precisely.
- FIXED M-05: "provably inside the fence" softened to "scoped and logged."

## Open (process, not code)

- OPEN: this project has not had an independent security, legal, or compliance review.
  Have a security engineer read the code before operational use.
- OPEN: parsers are validated against representative samples, not live output from every
  tool version. Validate against your installed versions during your first authorized run.

## Roadmap (not defects)

- Web crawling (katana), content discovery (ffuf), richer fingerprinting (whatweb).
- Per-finding tailored remediation text and CVSS scoring.
