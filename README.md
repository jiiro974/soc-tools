# soc-tools

**Security Operations Center Platform** — an integrated suite of security tools for SOC analysts, incident responders, and security engineers.

Built in Go. Cross-platform (macOS, Linux, Windows). Designed for corporate environments with VPN compatibility.

---

## The Platform

soc-tools is a unified ecosystem of specialized security projects, each addressing a critical SOC function:

| Project | Role | Status |
|---------|------|--------|
| [**ClientShell**](#clientshell) | Secure remote shell with Podman rootless isolation | v0.1.0-rc1 |
| [**Sentinel**](#sentinel) | Continuous vulnerability scanner | v0.1.0-rc1 |
| [**Patrol**](#patrol) | System hardening engine | v0.1.0-rc1 |
| [**Iris**](#iris) | Threat intelligence correlation | v0.1.0-rc1 |
| [**SOC Toolkit**](#soc-toolkit) | Threat hunting, triage, reporting | Roadmap v0.2 |

---

## ClientShell

**Secure remote access to operator workstations via containerized agents.**

Operators connect to isolated Podman containers running cs-agent. Sessions are recorded, shareable in real-time, and auditable. Zero trust — the server never touches the operator's host.

```
Operator machine                          Server (Scaleway)
┌──────────────────┐                     ┌──────────────────┐
│  clientshell CLI  │─── WebSocket ──────│   cs-server      │
│  ┌──────────────┐ │                     │  ┌────────────┐  │
│  │  Podman      │ │                     │  │ Dashboard  │  │
│  │  cs-agent    │ │                     │  │ WebTerminal│  │
│  │  (isolated)  │ │                     │  │ LiveShare  │  │
│  └──────────────┘ │                     │  └────────────┘  │
└──────────────────┘                     └──────────────────┘
```

**Key features:**
- Podman rootless — no Docker daemon, no root privileges
- VPN corporate compatible (slirp4netns, DNS injection)
- Session recording (asciicast format)
- Live share — invite colleagues to watch or co-edit sessions
- Offline mode with LAN relay
- 10 CLI commands: login, connect, disconnect, share, join, list, status, sync, version

```bash
$ clientshell connect Pragma
Pulling cs-agent:latest...
Starting interactive session on cs-Pragma...
Type :help for session commands

root@cs-Pragma:/# :share
Share URL: https://app.example.com/liveshare/5a2b...
Token: ABC-DEF-GHI (expires in 60 min, max 5 guests)

root@cs-Pragma:/# nmap -sV 10.0.1.0/24
Starting Nmap 7.95 ...
```

---

## Sentinel

**Continuous vulnerability scanner — CVE, IOC, YARA rules.**

Sentinel monitors the operator's environment in real-time, detecting known vulnerabilities, suspicious indicators, and malware signatures. Scans are non-intrusive and run without network access in demo mode.

```bash
$ sentinel --once

Sentinel v0.5.0
Detecting components... 6 found (go, docker, openssh, node, python, macos)
Sources: GitHub Advisory, Go Vuln DB, NVD

Scanning... done (24.5s)

  Findings: 0
  Status: GREEN

$ sentinel --demo

Sentinel v0.5.0 — DEMO MODE (no real system access)

  Findings: 10
  ┌──────────┬───────────────────────────────────────────┬──────────┐
  │ Severity │ Finding                                   │ Source   │
  ├──────────┼───────────────────────────────────────────┼──────────┤
  │ CRITICAL │ CVE-2024-40711 Veeam RCE                  │ NVD      │
  │ CRITICAL │ CVE-2024-3400 PAN-OS Command Injection    │ NVD      │
  │ CRITICAL │ Reverse shell listener on port 4444       │ IOC      │
  │ CRITICAL │ Known C2 domain in DNS cache              │ IOC      │
  │ HIGH     │ CVE-2024-21762 FortiOS Out-of-Bound Write │ NVD      │
  │ HIGH     │ Cryptominer XMRig detected                │ YARA     │
  │ HIGH     │ Suspicious PowerShell base64 execution    │ IOC      │
  │ MEDIUM   │ OpenSSH 9.2 < 9.6 (CVE-2024-6387)        │ GitHub   │
  │ MEDIUM   │ Node.js 18.x EOL approaching              │ GitHub   │
  │ MEDIUM   │ WebShell PHP signature in /tmp            │ YARA     │
  └──────────┴───────────────────────────────────────────┴──────────┘

$ sentinel rules list
  IOC Rules: 5 (hash, regex, domain, ip, url)
  YARA Rules: 5 (cryptominer, webshell, reverse_shell, credential_harvester, data_exfil)
```

---

## Patrol

**System hardening engine — ANSSI, ISO 27001, MITRE ATT&CK coverage.**

Patrol audits the operator's machine against security baselines, provides a maturity score, maps findings to MITRE ATT&CK techniques, and offers guided remediation with rollback support.

```bash
$ patrol audit

Patrol — Security Maturity Score

  Level 0 (Baseline)  ██████████████████░░  90%  9/10  [PASS]

  Overall: 90%

  [FAIL] 1 failed check(s):
    HIGH  BL-05  Automatic updates enabled  [ANSSI-34, ISO-A.12.6.1, T1190, T1203]

[MITRE] MITRE ATT&CK Coverage

  INITIAL ACCESS           ████████████████░░░░  83%
  PERSISTENCE              ████████████████████ 100%
  PRIVILEGE ESCALATION     ████████████████████ 100%
  CREDENTIAL ACCESS        ████████████████████ 100%
  DEFENSE EVASION          ████████████████████ 100%
  LATERAL MOVEMENT         ████████████████████ 100%
  COLLECTION               ████████████████████ 100%
  EXECUTION                ░░░░░░░░░░░░░░░░░░░░   0%  <- weakest
  IMPACT                   ████████████████████ 100%

  Overall MITRE: 91%

$ patrol fix

  What do you want to fix?
    1) [MITRE]  By MITRE tactic (fix all attacks of one type)
    2) [IMPACT] By impact (biggest score improvements first)
    3) [LEVEL]  By level (Level 0 → 1 → 2)
    4) [QUICK]  Quick wins only (low risk)
    5) [FIX]    Everything (interactive)
    q) Cancel

$ patrol rollback --list
  13 snapshots available
  BL-01  2026-03-24T10:14  Firewall enabled
  BL-04  2026-03-24T10:15  SSH root login disabled
  BL-05  2026-03-24T10:16  Automatic updates enabled
  ...
```

**94 security checks** across 3 maturity levels. Guided remediation with automatic snapshots and one-click rollback. Compliance mappings: ANSSI (34 controls), ISO 27001 (Annex A), MITRE ATT&CK (9 tactics).

---

## Iris

**Threat intelligence correlation and scoring engine.**

Iris collects IOCs from multiple sources, enriches them via VirusTotal and cs-toolbox, correlates indicators, and produces risk scores for security decision-making.

```bash
$ iris score --ioc 203.0.113.42

  IOC: 203.0.113.42 (IPv4)

  Sources:
  ├── VirusTotal    14/87 engines flagged    Score: 72/100
  ├── AbuseIPDB     Confidence: 98%          Category: Brute-Force
  ├── cs-toolbox    whois: AS-EXAMPLE        Registered: 2025-11-03
  └── dns-enum      PTR: scan.example.net    Suspicious: DGA score 0.84

  Correlation:
  ├── Linked to 3 other IOCs (same ASN)
  ├── First seen: 2026-02-24 (30 days ago)
  └── Campaign: BruteForce-EU-Q1-2026

  Risk Score: 87/100 [HIGH]
  Recommendation: Block at perimeter, add to watchlist

$ iris dashboard

  Iris v1.0.0-rc1 — Threat Intelligence Dashboard
  ┌──────────────────────────────────────────────────┐
  │ Active IOCs: 2,847    High Risk: 142    New: 38  │
  │ Collectors: 4 active  Last scan: 2 min ago       │
  │ Enrichment: VT, AbuseIPDB, cs-toolbox            │
  └──────────────────────────────────────────────────┘
```

**Features:** Multi-source collection, VirusTotal enrichment, cs-toolbox integration (whois, dns-enum, cert-check, http-headers), risk scoring engine, web dashboard (dark mode), FR/EN labels.

---

## SOC Toolkit

> **Coming in v0.2** — Automated threat hunting, incident triage, and SOC reporting.

The SOC Toolkit integrates with Cortex XSIAM, Jira Cloud, and Microsoft Teams to automate the analyst workflow:

| Tool | Description | Integrations |
|------|-------------|-------------|
| `soc-hunt` | Threat hunting via 18 XQL templates | Cortex XSIAM |
| `soc-triage` | Automated incident triage with ticket creation | XSIAM, Jira Cloud, Teams |
| `soc-report` | Daily/weekly/executive SOC reports | XSIAM, Jira Cloud |

**XQL Templates included:** BruteForce, C2Beaconing, DCSync, PowerShellEncoded, Kerberoast, FileDropInTemp, LargeExfiltration, Ransomware, SearchByIP, SearchByHash, SearchByDomain, AlertsByCategory, CollectorHealth, EndpointsDisconnected, MTTR, OutboundNonStandard, DNSToRareDomain.

**Roadmap:**

| Phase | Timeline | Focus |
|-------|----------|-------|
| v0.2 — Automation | May-Jun 2026 | Playbooks, IOC enrichment, webhooks |
| v0.3 — Intelligence | Jul-Sep 2026 | TIP, ML anomaly detection, STIX/TAXII |
| v1.0 — Enterprise | Oct-Dec 2026 | Multi-tenant, REST API, Kubernetes |

---

## Installation

### Pre-built binaries

Download from [Releases](https://github.com/jiiro974/soc-tools/releases):

```bash
# macOS (Apple Silicon)
curl -LO https://github.com/jiiro974/soc-tools/releases/latest/download/soc-tools-darwin-arm64.tar.gz
tar xzf soc-tools-darwin-arm64.tar.gz -C ~/.local/bin/

# Linux (amd64)
curl -LO https://github.com/jiiro974/soc-tools/releases/latest/download/soc-tools-linux-amd64.tar.gz

# Windows
curl -LO https://github.com/jiiro974/soc-tools/releases/latest/download/soc-tools-windows-amd64.zip
```

Each project can also be installed individually from its own repository.

---

## Architecture

```
                          ┌─────────────┐
                          │  cs-server  │
                          │  (backend)  │
                          └──────┬──────┘
                                 │ WebSocket + REST API
              ┌──────────────────┼──────────────────┐
              │                  │                   │
     ┌────────▼───────┐  ┌──────▼──────┐  ┌────────▼────────┐
     │  ClientShell   │  │  Dashboard  │  │   LiveShare     │
     │  CLI (Podman)  │  │  (Web UI)   │  │   (xterm.js)    │
     └────────┬───────┘  └─────────────┘  └─────────────────┘
              │
     ┌────────▼───────┐
     │   cs-agent     │    ┌───────────┐  ┌──────────┐  ┌──────┐
     │  (container)   │    │ Sentinel  │  │  Patrol  │  │ Iris │
     │  PTY + record  │    │  (vuln)   │  │ (harden) │  │(TIP) │
     └────────────────┘    └───────────┘  └──────────┘  └──────┘
```

All projects are written in Go, compiled as static binaries, and designed for air-gapped or restricted network environments.

---

## License

Proprietary — pragma-tic.org
