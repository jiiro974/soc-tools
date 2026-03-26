# soc-tools

**Security Operations Center Toolkit** — a modular suite of CLI tools for SOC analysts, incident responders, and security engineers.

Built in Go. Cross-platform (macOS, Linux, Windows). Zero external dependencies at runtime.

## Tools

### soc-hunt — Threat Hunting

Automated threat hunting via XQL templates against Cortex XSIAM.

```bash
# Hunt brute force attacks over 24h
soc-hunt -template brute-force -timeframe 24h -threshold 10

# Search IOC by IP across 30 days
soc-hunt -ioc-ip 91.80.49.85 -timeframe 30d

# Hunt C2 beaconing patterns
soc-hunt -template c2-beaconing -timeframe 7d
```

**18 XQL templates included:** BruteForce, C2Beaconing, DCSync, PowerShellEncoded, Kerberoast, FileDropInTemp, LargeExfiltration, Ransomware, SearchByIP, SearchByHash, SearchByDomain, AlertsByCategory, CollectorHealth, EndpointsDisconnected, MTTR, OutboundNonStandard, DNSToRareDomain.

### soc-triage — Incident Triage

Automated incident triage with Jira integration for ticket creation and tracking.

```bash
# Triage critical/high incidents, create Jira tickets
soc-triage -severity critical,high -status new -create-jira

# Dry-run triage with Teams notification
soc-triage -severity critical -dry-run -notify teams
```

### soc-report — SOC Reporting

Generate SOC reports from incident data, metrics, and threat intelligence.

```bash
# Daily SOC report in Markdown
soc-report -type daily -format markdown -output report.md

# Weekly executive summary
soc-report -type weekly -format pdf -output weekly.pdf
```

### Network & Forensics Tools

| Tool | Description |
|------|-------------|
| `ssl-audit` | TLS/SSL configuration audit |
| `http-headers` | HTTP security headers analysis |
| `cert-check` | Certificate chain validation and expiry monitoring |
| `portscan` | TCP port scanner with service detection |
| `dns-enum` | DNS enumeration and zone analysis |
| `whois` | WHOIS domain intelligence |
| `traceroute` | Network path analysis |
| `netcat` | TCP/UDP connection testing |

### IOC & Threat Intelligence

| Tool | Description |
|------|-------------|
| `ioc-scan` | Scan files and systems for Indicators of Compromise |
| `yara-scan` | YARA rule-based malware detection |
| `ip-lookup` | IP reputation and geolocation |
| `mx-toolbox` | Email infrastructure auditing (SPF, DKIM, DMARC) |

### System & Compliance

| Tool | Description |
|------|-------------|
| `secret-scanner` | Detect leaked secrets in code and configs |
| `file-integrity` | File integrity monitoring (FIM) |
| `log-analyze` | Log analysis and pattern detection |
| `timeline` | Forensic timeline reconstruction |
| `service-health` | Service availability monitoring |

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

# Windows
curl -LO https://github.com/jiiro974/soc-tools/releases/latest/download/soc-tools-windows-amd64.zip
```

### Verify

```bash
soc-tools version
```

## Configuration

SOC integration tools (soc-hunt, soc-triage, soc-report) require API credentials:

```bash
# Cortex XSIAM
export XSIAM_FQDN="api-tenant.xdr.eu.paloaltonetworks.com"
export XSIAM_API_KEY="your-api-key"
export XSIAM_API_KEY_ID="42"

# Jira Cloud
export JIRA_URL="https://your-org.atlassian.net"
export JIRA_EMAIL="user@example.com"
export JIRA_TOKEN="your-api-token"

# Microsoft Teams
export TEAMS_WEBHOOK_URL="https://outlook.webhook.office.com/webhookb2/..."
```

Network and forensics tools work without any configuration.

## Part of the ClientShell ecosystem

soc-tools is part of the [ClientShell](https://clientshell.io) security platform:

- **ClientShell** — Secure remote shell with Podman isolation
- **Sentinel** — Continuous vulnerability scanner
- **Patrol** — System hardening (ANSSI/ISO 27001/MITRE ATT&CK)
- **Iris** — Threat intelligence correlation engine
- **soc-tools** — SOC operations toolkit (this project)

## License

Proprietary — pragma-tic.org
