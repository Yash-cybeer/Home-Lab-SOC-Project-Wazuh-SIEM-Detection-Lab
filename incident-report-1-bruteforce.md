# Incident Report #1 — Brute-Force Failed Login Attempts

**Date:** November 3, 2026
**Reported by:** SOC Analyst (Home Lab)
**Asset Affected:** win-victim (192.168.56.102, Windows 10 Pro)
**Detection Source:** Wazuh SIEM (Threat Hunting module)
**Severity:** Medium (escalated to High via lockout)

## Summary
Multiple failed authentication attempts were detected against the win-victim host, consistent with a brute-force credential attack. The attack was simulated using `net use` to generate repeated failed logon attempts against the host.

## Detection Details
| Rule ID | Description | Level |
|---|---|---|
| 60122 | Logon failure | 5 |
| 60204 | Multiple logon failures | 10 |
| 60115 | Account locked out | 9 |

- **MITRE ATT&CK:** T1110 — Brute Force (Credential Access)
- **Windows Event ID:** 4625 (An account failed to log on)
- **Logon Type:** 3 (Network)
- **Source IP:** 127.0.0.1 (local test)
- **Total hits:** 11

## Analysis
The repeated 4625 events combined with the eventual lockout event indicate the target account's lockout policy was triggered, which is the expected defensive behavior. Wazuh correctly escalated severity across the sequence (failure → repeated failure → lockout), demonstrating proper rule tuning for brute-force detection.

## Recommendation
- Confirm account lockout policies are enforced lab-wide
- Consider configuring alert integrations (email/Slack) for rule 60204+ in a production setting
- MITRE mapping confirms this maps cleanly to T1110 for future detection tuning and dashboard correlation
