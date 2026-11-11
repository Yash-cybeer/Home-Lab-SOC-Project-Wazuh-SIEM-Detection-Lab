# Incident Report #2 — Unauthorized Local User Creation

**Date:** November 3, 2026
**Reported by:** SOC Analyst (Home Lab)
**Asset Affected:** win-victim (192.168.56.102, Windows 10 Pro)
**Detection Source:** Wazuh SIEM
**Severity:** Medium-High (persistence indicator)

## Summary
A new local user account (`testuser`) was created on the win-victim host, simulating an attacker establishing persistence after gaining access.

## Detection Details
| Rule ID | Description | Level |
|---|---|---|
| 60109 | Local user account created | 8 |

- **MITRE ATT&CK:** T1136 — Create Account (Persistence)
- **Windows Event ID:** 4720 (A user account was created)
- **Total hits:** 2

## Analysis
Account creation events are a classic persistence indicator. The fact that this immediately followed the brute-force activity in Incident #1 would, in a real scenario, suggest a successful compromise followed by persistence setup — worth correlating as a single incident chain in a mature SOC environment.

## Recommendation
- Alert on any 4720 event for non-admin-initiated account creation
- Correlate with preceding authentication failures to detect multi-stage attack chains
- Review the new account's group memberships immediately after creation (watch for unexpected addition to Administrators group)
