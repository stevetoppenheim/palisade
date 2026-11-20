<img width="1600" height="1000" alt="palisade_logo" src="https://github.com/user-attachments/assets/fd697477-e973-49e0-8f29-9168a5472cb9" />


# Palisade

**Website:** https://www.opey.org/opeyit/palisade/ · **Contact:** steveoppenheim@opey.org

> **Status: v1.0.0.** 33 unit tests across scope enforcement, input parsing, tool-output
> parsing, and reporting. All findings from the published
> [internal audit](docs/audit/audit_findings.md) are fixed or addressed
> ([`KNOWN_ISSUES.md`](KNOWN_ISSUES.md)). Authorized use only. The review is internal,
> not independent, and the parsers are validated against representative samples rather
> than every live tool version, so validate against your installed versions on first run.

A **reactive, authorization-gated vulnerability assessment orchestrator** for Kali Linux.

It ingests a list of targets (IPs / CIDRs / URLs / ports) from a text, CSV, XLSX, or
XML file, enforces an explicit engagement **scope**, runs a configurable pipeline of
security tools against *in-scope* targets only, and produces a **per-IP vulnerability
report** plus a combined summary. Every action is written to an append-only **audit
log** (append-only, not tamper-proof: see the audit for the precise claim).

**Repo map:** [`palisade/`](palisade) engine · [`config/`](config) scope & pipeline
examples · [`tests/`](tests) · [`honeypot/`](honeypot) defensive companion ·
[`docs/audit/`](docs/audit) the audit · [`docs/marketing/`](docs/marketing) brief ·
[`CONTRIBUTING.md`](CONTRIBUTING.md) · [`SECURITY.md`](SECURITY.md)

---

## ⚖️ Read this first: authorization is mandatory, not optional

Running active scanners against systems you do not own or do not have **written
permission** to test is a crime in most jurisdictions (e.g. the U.S. Computer Fraud and
Abuse Act, the UK Computer Misuse Act, and equivalents elsewhere). This tool is for:

- systems you own, or
- systems covered by a signed engagement / rules-of-engagement (RoE) document, or
- a bug-bounty program whose published scope authorizes active testing.

**Enforcement is built in.** The orchestrator will not scan any target that is not
listed in an explicit `scope.yaml` allowlist, and the scope file must contain
engagement metadata (client, authorizing party, ticket/contract ref, start/end dates).
Out-of-window or out-of-scope targets are refused and logged. This is a guardrail, not
a legal opinion: you remain responsible for having real authorization.

---

## What it does, and what it does not

**Does (detection and evidence), implemented today:**
- Host discovery (nmap ping sweep)
- Fast port discovery (masscan / naabu) + accurate service & version detection (nmap -sV)
- Web service fingerprinting (httpx: status, title, tech-detect)
- Template-based vulnerability detection (nuclei: CVEs, misconfigurations, exposures)
- NSE vulnerability scripts (nmap --script vuln)
- Web server checks (nikto)
- TLS/SSL assessment (testssl.sh)
- Optional heavyweight authenticated/network vuln scanning (Greenbone GVM / OpenVAS): hook only
- Normalizes everything into findings and writes a report per IP.

**Roadmap (not yet implemented, do not rely on these):**
- Web crawling (katana), content discovery (ffuf), richer fingerprinting (whatweb)
- Per-finding tailored remediation text and CVSS scoring

> This project ships with a published internal audit (`docs/audit/`) whose findings are
> all fixed or addressed in v1.0.0. The review is internal, not independent. Authorized
> use only.

**Does NOT (by design):**
- Automated **exploitation** / payload delivery / shellcode.
- Credential **brute-forcing** or password spraying.
- Automated lateral movement, persistence, or data exfiltration.

Those steps turn an *assessment* into an *attack*, can destroy production systems, and
in a lawful engagement are performed **manually, one target at a time, by a human** who
has confirmed the target and blast radius. For that step the tool exposes a
scope-locked, interactive `verify` helper (see below) rather than an unattended
auto-exploit loop. If your RoE requires automated exploitation, that belongs in a
separate, human-supervised Metasploit/CobaltStrike workflow outside this orchestrator.

---

## Architecture

```
            ┌─────────────┐
 input file │ inputs.py   │  txt / csv / xlsx / xml  -> Target objects
 ──────────▶│ (parsers)   │
            └──────┬──────┘
                   │ raw targets
            ┌──────▼──────┐
            │ scope.py    │  allowlist + date window + concurrency cap
            │ (ENFORCER)  │  refuses out-of-scope, logs every decision
            └──────┬──────┘
                   │ in-scope targets
            ┌──────▼──────┐        ┌────────────┐
            │ pipeline.py │◀──────▶│ audit.py   │  JSONL: every command, ts, decision
            │ (scheduler) │        └────────────┘
            └──────┬──────┘
                   │ per-target tool chain (concurrent, concurrency-capped)
            ┌──────▼──────┐
            │ tools.py    │  nmap / masscan / naabu / httpx /
            │ (wrappers)  │  nuclei / nikto / testssl / gvm(hook)
            └──────┬──────┘
                   │ normalized Findings
            ┌──────▼──────┐
            │ report.py   │  per-IP  report_<ip>.{md,html,json} + summary
            └─────────────┘
```

The **honeypot** component is intentionally separate (`honeypot/`): honeypots are
*defensive* decoys you deploy on **your own** network to detect intruders: they are not
something you aim at a target. See `honeypot/README.md`.

---

## Install (Kali Linux)

```bash
# system tools (most ship with Kali; this refreshes/adds the modern suite)
sudo apt update
sudo apt install -y nmap masscan nikto testssl.sh python3-pip
# ProjectDiscovery suite (latest, Go-based): the ones Palisade actually calls
sudo apt install -y golang-go
for t in nuclei naabu httpx; do \
  go install github.com/projectdiscovery/$t/v3/cmd/$t@latest; done
sudo cp ~/go/bin/{nuclei,naabu,httpx} /usr/local/bin/ 2>/dev/null || true
nuclei -update-templates          # run this now, and schedule it (cron): Palisade
                                  # does NOT auto-update templates per scan

# optional heavy scanner
sudo apt install -y gvm && sudo gvm-setup   # Greenbone/OpenVAS

# python side
cd whitehat-recon
python3 -m pip install -r requirements.txt
```

## Use

```bash
# 0. confirm your toolchain and run the built-in self-audit
python3 -m wrecon.cli selfcheck

# 1. edit the scope to your authorized engagement
cp config/scope.example.yaml scope.yaml && $EDITOR scope.yaml

# 2. DRY RUN first: shows exactly what would run, touches nothing
python3 -m wrecon.cli scan --targets targets.txt --scope scope.yaml \
        --pipeline config/pipeline.example.yaml --out runs/acme --dry-run

# 3. real run
sudo python3 -m wrecon.cli scan --targets targets.xlsx --scope scope.yaml \
        --pipeline config/pipeline.example.yaml --out runs/acme

# reports land in runs/acme/reports/report_<ip>.{md,html,json} + summary.html
# audit trail in runs/acme/audit.jsonl
```

`sudo` is only needed for raw-socket scans (SYN scan, masscan); otherwise it runs
unprivileged with TCP connect scans.

## Audit / vetting

- `python3 -m wrecon.cli selfcheck`: reports tool availability & versions, validates
  config schemas, and runs the scope self-test.
- `python3 -m pytest tests/`: unit tests covering the scope enforcer and input parsers
  (the safety-critical code). CI config in `.github/workflows/` runs these on every push.
- `runs/<name>/audit.jsonl`: append-only record of every command executed, its target,
  timestamp, and the scope decision that permitted it.

## License / disclaimer

Provided as-is for authorized security testing and education. The authors accept no
liability for misuse. You are responsible for holding valid authorization for every
target.
