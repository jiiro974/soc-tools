# soc-tools

**Security Operations Center Toolkit** — a modular suite of CLI tools for SOC analysts, incident responders, and security engineers.

Built in Go. Cross-platform (macOS, Linux, Windows). Zero external runtime dependencies.

---

## Components

### Core SOC Tools

| Binary | Description | Integrations |
|--------|-------------|-------------|
| `soc-hunt` | Threat hunting via XQL templates | Cortex XSIAM |
| `soc-triage` | Automated incident triage | Cortex XSIAM, Jira Cloud, Teams |
| `soc-report` | SOC report generation (daily, weekly, executive) | Jira Cloud, Cortex XSIAM |

### Network & Forensics

| Binary | Description |
|--------|-------------|
| `ssl-audit` | TLS/SSL configuration audit (protocols, ciphers, certificate chain) |
| `http-headers` | HTTP security headers analysis (CSP, HSTS, X-Frame, CORS) |
| `cert-check` | Certificate validation, expiry monitoring, chain of trust |
| `portscan` | TCP port scanner with service detection |
| `dns-enum` | DNS enumeration, zone transfer, subdomain discovery |
| `whois` | WHOIS domain/IP intelligence |
| `traceroute` | Network path analysis with latency |
| `netcat` | TCP/UDP connection testing and banner grabbing |
| `subnet` | Subnet calculator and IP range analysis |

### Threat Intelligence & IOC

| Binary | Description |
|--------|-------------|
| `ioc-scan` | Scan files and systems for Indicators of Compromise |
| `yara-scan` | YARA rule-based malware/pattern detection |
| `mx-toolbox` | Email infrastructure auditing (SPF, DKIM, DMARC, MX records) |
| `secret-scanner` | Detect leaked secrets, API keys, tokens in code and configs |

### System & Compliance

| Binary | Description |
|--------|-------------|
| `file-integrity` | File integrity monitoring (FIM) with baseline comparison |
| `log-analyze` | Log analysis, pattern detection, anomaly flagging |
| `timeline` | Forensic timeline reconstruction from multiple log sources |
| `service-health` | Service availability monitoring and healthchecks |

### Libraries (pkg/)

| Package | Description |
|---------|-------------|
| `pkg/xsiam` | Client API Cortex XSIAM (incidents, alerts, XQL, endpoints) |
| `pkg/jira` | Client API Jira Cloud v3 (CRUD, JQL, ADF, transitions) |
| `pkg/keeper` | Integration Keeper Secrets Manager |
| `pkg/teams` | Microsoft Teams notifications (Incoming Webhooks + Adaptive Cards) |

---

## Demos

### soc-hunt — Threat Hunting

```bash
$ soc-hunt -template brute-force -timeframe 24h -threshold 10

[HUNT] Template: BruteForce | Timeframe: 24h | Threshold: 10
[XSIAM] Running XQL query... done (2.3s)

  RESULTS: 3 matches found

  #1  Source: 203.0.113.42    Target: DC-PROD-01     Attempts: 847    First: 2026-03-25T02:14:00Z
      Accounts targeted: svc-backup, admin, administrator, sa
      Status: ACTIVE — last attempt 4 min ago

  #2  Source: 198.51.100.17   Target: VPN-GW-02      Attempts: 312    First: 2026-03-25T08:41:00Z
      Accounts targeted: jdupont, mmartin, adurand
      Status: STOPPED — last attempt 6h ago

  #3  Source: 192.0.2.88      Target: MAIL-01        Attempts: 156    First: 2026-03-25T14:22:00Z
      Accounts targeted: postmaster, info, contact
      Status: ACTIVE — last attempt 12 min ago

  [ACTION] 2 active threats — run: soc-triage -severity high -source 203.0.113.42,192.0.2.88
```

```bash
$ soc-hunt -ioc-ip 203.0.113.42 -timeframe 30d

[HUNT] IOC Search: IP 203.0.113.42 | Timeframe: 30d
[XSIAM] Running XQL query... done (4.1s)

  TIMELINE:
  2026-02-24  First seen — DNS query to rare domain (c2-relay.example.net)
  2026-03-01  Port scan detected (22, 443, 3389, 5985) against 14 hosts
  2026-03-10  Failed RDP logon attempts on TERM-SRV-03 (EventID 4625, x42)
  2026-03-25  Brute-force campaign (847 attempts against DC-PROD-01)

  MITRE ATT&CK: T1595 (Active Scanning) → T1110 (Brute Force) → T1078 (Valid Accounts)
  VERDICT: HIGH RISK — multi-stage attack pattern detected
  RECOMMENDATION: Isolate source, block at perimeter, hunt for lateral movement
```

### soc-triage — Incident Triage

```bash
$ soc-triage -severity critical,high -status new -create-jira

[TRIAGE] Querying XSIAM... 7 incidents found (3 critical, 4 high)
[TRIAGE] Enriching IOCs...

  #1  INC-20260325-001 [CRITICAL]  Ransomware — Encrypted files detected
      Host: FS-PROD-02 | User: svc-backup | IOCs: 3 IPs, 2 hashes
      Enrichment: VT 14/72, AbuseIPDB score 98/100
      → Jira SOC-1847 created, assigned to on-call analyst
      → Teams alert sent to #security-critical

  #2  INC-20260325-002 [CRITICAL]  PowerShell — Base64 encoded execution
      Host: WS-DEV-14 | User: a.martin | IOCs: 1 hash, 1 domain
      Enrichment: VT 8/72, domain registered 2 days ago
      → Jira SOC-1848 created, assigned to on-call analyst
      → Teams alert sent to #security-critical

  #3  INC-20260325-003 [CRITICAL]  Credential Dump — LSASS memory access
      Host: DC-PROD-01 | User: SYSTEM (via svc-deploy) | IOCs: 1 hash
      Enrichment: VT 22/72 (Mimikatz variant)
      → Jira SOC-1849 created, priority P1, assigned to IR team
      → Teams alert sent to #security-critical + #incident-response

  ... +4 high severity incidents triaged

  SUMMARY: 7 triaged | 7 Jira tickets | 3 Teams alerts | MTTR est. < 5 min
```

### soc-report — SOC Reporting

```bash
$ soc-report -type daily -format markdown -output report.md

[REPORT] Generating daily SOC report for 2026-03-25...
[XSIAM] Fetching incidents... 42 total (3 critical, 12 high, 27 medium)
[JIRA] Fetching tickets... 38 open, 14 closed today

  Daily SOC Report — 2026-03-25
  ====================================

  Executive Summary
  - Total incidents: 42 (+8 vs yesterday)
  - Critical: 3 (2 resolved, 1 active)
  - MTTR: 4m 23s (target < 5m) ........... OK
  - False positive rate: 3.2% (target < 5%) OK
  - XQL queries executed: 847 (quota: 62% remaining)

  Top Threats
  1. Brute-force campaign from 203.0.113.0/24 (847 attempts)
  2. Ransomware attempt on FS-PROD-02 (contained in 3m)
  3. Credential dump via Mimikatz variant on DC-PROD-01

  MITRE ATT&CK Coverage
  - Initial Access: 94% detected
  - Credential Access: 88% detected
  - Lateral Movement: 76% detected (improvement needed)

  Written to: report.md (2.4 KB)
```

### ssl-audit — TLS/SSL Audit

```bash
$ ssl-audit app.example.com:443

[SSL] Connecting to app.example.com:443...

  Certificate
  ├── Subject:     *.example.com
  ├── Issuer:      Let's Encrypt Authority X3
  ├── Valid:       2026-01-15 → 2026-04-15 (20 days remaining)
  ├── Serial:      03:A1:B2:C3:...:F9
  └── SANs:        *.example.com, example.com

  Protocol Support
  ├── TLS 1.3    ✓ (preferred)
  ├── TLS 1.2    ✓
  ├── TLS 1.1    ✗ (disabled)
  └── TLS 1.0    ✗ (disabled)

  Cipher Suites
  ├── TLS_AES_256_GCM_SHA384        ✓ A+
  ├── TLS_CHACHA20_POLY1305_SHA256  ✓ A+
  └── TLS_AES_128_GCM_SHA256        ✓ A

  Security
  ├── HSTS:              max-age=31536000; includeSubDomains
  ├── OCSP Stapling:     ✓
  ├── Certificate Trans.: ✓ (2 SCTs)
  └── Key Pinning:       not set

  Grade: A
  ⚠ WARNING: Certificate expires in 20 days — renew now
```

### http-headers — Security Headers Analysis

```bash
$ http-headers https://app.example.com

[HEADERS] Analyzing https://app.example.com...

  Security Headers
  ├── Strict-Transport-Security   ✓  max-age=31536000; includeSubDomains
  ├── Content-Security-Policy     ✓  default-src 'self'; script-src 'self'
  ├── X-Content-Type-Options      ✓  nosniff
  ├── X-Frame-Options             ✓  DENY
  ├── Referrer-Policy             ✓  strict-origin-when-cross-origin
  ├── Permissions-Policy          ✓  camera=(), microphone=(), geolocation=()
  ├── X-XSS-Protection           ⚠  not set (deprecated but recommended)
  └── Cross-Origin-Opener-Policy  ✗  missing

  Cookie Security
  ├── session_id    Secure ✓  HttpOnly ✓  SameSite=Strict ✓  Path=/
  └── csrf_token    Secure ✓  HttpOnly ✗  SameSite=Lax    ✓  Path=/

  Grade: B+
  ✗ Missing: Cross-Origin-Opener-Policy
  ⚠ csrf_token cookie missing HttpOnly flag
```

### ioc-scan — IOC Scanner

```bash
$ ioc-scan --path /var/log/auth.log --rules default

[IOC] Scanning /var/log/auth.log with 847 IOC rules...

  FINDINGS: 4 matches

  #1  [HIGH]   IP 203.0.113.42 — known brute-force source
      Line 1847: "Failed password for root from 203.0.113.42 port 44831 ssh2"
      Intel: AbuseIPDB 98/100, VT 4/87, first seen 2026-02-24

  #2  [HIGH]   IP 198.51.100.77 — C2 infrastructure
      Line 2341: "Connection from 198.51.100.77 on port 4444"
      Intel: VT 12/87, tagged APT-29 infrastructure

  #3  [MEDIUM] Domain c2-relay.example.net — suspicious DNS query
      Line 3102: "query c2-relay.example.net from 10.0.1.45"
      Intel: Registered 48h ago, DGA score 0.87

  #4  [LOW]    Hash a1b2c3d4...ef56 — PUA detection
      Line 4501: "Executed /tmp/.cache/update.bin"
      Intel: VT 3/72, classified as PUP.CoinMiner

  SUMMARY: 4 IOCs found (2 high, 1 medium, 1 low)
```

### yara-scan — YARA Scanner

```bash
$ yara-scan --rules /etc/yara/malware.yar --target /opt/uploads/

[YARA] Scanning /opt/uploads/ with 156 rules...
[YARA] 2,847 files scanned in 12.3s

  MATCHES: 2

  #1  [CRITICAL] Rule: CryptoMiner_XMRig
      File:  /opt/uploads/2026-03/lib_update.so
      Size:  2.1 MB | SHA256: 7f8a9b...c3d2
      Strings matched: "$xmrig_pool", "$stratum_connect", "$cpu_mining_config"

  #2  [HIGH] Rule: WebShell_PHP_Generic
      File:  /opt/uploads/2026-03/cache.php
      Size:  4.7 KB | SHA256: e1f2a3...b4c5
      Strings matched: "$_POST['cmd']", "base64_decode", "system("

  SUMMARY: 2 matches in 2,847 files (0.07% hit rate)
```

---

## XQL Templates

18 pre-built XQL templates for Cortex XSIAM threat hunting:

| Category | Templates |
|----------|-----------|
| **Threat Hunting** | BruteForce, C2Beaconing, DCSync, PowerShellEncoded, Kerberoast, FileDropInTemp, LargeExfiltration, Ransomware |
| **IOC Hunting** | SearchByIP, SearchByHash, SearchByDomain |
| **Operational** | AlertsByCategory, CollectorHealth, EndpointsDisconnected, MTTR |
| **Network** | OutboundNonStandard, DNSToRareDomain |

---

## Installation

### Pre-built binaries

Download from [Releases](https://github.com/jiiro974/soc-tools/releases):

```bash
# macOS (Apple Silicon)
curl -LO https://github.com/jiiro974/soc-tools/releases/latest/download/soc-tools-darwin-arm64.tar.gz
tar xzf soc-tools-darwin-arm64.tar.gz -C ~/.local/bin/

# macOS (Intel)
curl -LO https://github.com/jiiro974/soc-tools/releases/latest/download/soc-tools-darwin-amd64.tar.gz

# Linux (amd64)
curl -LO https://github.com/jiiro974/soc-tools/releases/latest/download/soc-tools-linux-amd64.tar.gz

# Linux (arm64)
curl -LO https://github.com/jiiro974/soc-tools/releases/latest/download/soc-tools-linux-arm64.tar.gz

# Windows
curl -LO https://github.com/jiiro974/soc-tools/releases/latest/download/soc-tools-windows-amd64.zip
```

### Verify installation

```bash
$ soc-hunt version
soc-hunt v0.2.0 (darwin/arm64)
XQL templates: 18 | Integrations: XSIAM, Jira, Teams, Keeper

$ ssl-audit version
ssl-audit v0.2.0 (darwin/arm64)
```

---

## Configuration

### SOC integrations (soc-hunt, soc-triage, soc-report)

```bash
# Cortex XSIAM
export XSIAM_FQDN="api-tenant.xdr.eu.paloaltonetworks.com"
export XSIAM_API_KEY="your-api-key"
export XSIAM_API_KEY_ID="42"

# Jira Cloud
export JIRA_URL="https://your-org.atlassian.net"
export JIRA_EMAIL="analyst@your-org.com"
export JIRA_TOKEN="your-api-token"

# Microsoft Teams
export TEAMS_WEBHOOK_URL="https://outlook.webhook.office.com/webhookb2/..."

# Keeper Secrets Manager (recommended over env vars)
# ~/.soc/ksm-config.json (permissions 0600)
```

Network and forensics tools (ssl-audit, http-headers, portscan, etc.) work without any configuration.

---

## Architecture

```
soc-tools/
├── cmd/
│   ├── soc-hunt/         # Threat hunting via XQL templates
│   ├── soc-triage/       # Automated incident triage
│   ├── soc-report/       # SOC report generation
│   ├── ssl-audit/        # TLS/SSL audit
│   ├── http-headers/     # HTTP security headers
│   ├── cert-check/       # Certificate monitoring
│   ├── portscan/         # Port scanner
│   ├── dns-enum/         # DNS enumeration
│   ├── ioc-scan/         # IOC scanner
│   ├── yara-scan/        # YARA rule scanner
│   ├── mx-toolbox/       # Email infrastructure audit
│   ├── secret-scanner/   # Secret detection
│   ├── log-analyze/      # Log analysis
│   ├── timeline/         # Forensic timeline
│   └── service-health/   # Service monitoring
├── pkg/
│   ├── xsiam/            # Cortex XSIAM API client
│   ├── jira/             # Jira Cloud v3 API client
│   ├── keeper/           # Keeper Secrets Manager
│   └── teams/            # Microsoft Teams webhooks
└── docs/
```

---

## Part of the ClientShell ecosystem

soc-tools is part of the [ClientShell](https://clientshell.io) security platform:

| Project | Description |
|---------|-------------|
| **ClientShell** | Secure remote shell with Podman rootless isolation |
| **Sentinel** | Continuous vulnerability scanner (CVE, IOC, YARA) |
| **Patrol** | System hardening engine (ANSSI, ISO 27001, MITRE ATT&CK) |
| **Iris** | Threat intelligence correlation and scoring |
| **soc-tools** | SOC operations toolkit (this project) |

---

## Roadmap

| Phase | Timeline | Focus |
|-------|----------|-------|
| v0.1 — Foundations | Mar-Apr 2026 | Stabilization, security hardening, CI/CD |
| v0.2 — Automation | May-Jun 2026 | Playbooks, IOC enrichment, webhooks |
| v0.3 — Intelligence | Jul-Sep 2026 | TIP, ML anomaly detection, correlation |
| v1.0 — Enterprise | Oct-Dec 2026 | Multi-tenant, REST API, Kubernetes, SLA |

---

## License

Proprietary — pragma-tic.org
