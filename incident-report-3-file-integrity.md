# Incident Report #3 — File Integrity Monitoring (File Tampering)

**Date:** November 11, 2026
**Reported by:** SOC Analyst (Home Lab)
**Asset Affected:** win-victim (192.168.56.102, Windows 10 Pro)
**Detection Source:** Wazuh SIEM (syscheck / FIM module)
**Severity:** Medium-High

## Summary
A monitored directory (`C:\Users\Public\fim_test`) was configured via Wazuh's File Integrity Monitoring (syscheck) module. A file was created, modified, and then deleted within the monitored path to validate full-lifecycle FIM detection.

## Configuration
```xml
<agent_config>
  <syscheck>
    <directories check_all="yes" report_changes="yes" realtime="yes">C:\Users\Public\fim_test</directories>
  </syscheck>
</agent_config>
```
Applied via the Wazuh manager's shared agent group configuration (`/var/ossec/etc/shared/default/agent.conf`), with `realtime="yes"` enabling near-instant detection and `report_changes="yes"` capturing content-level diffs.

## Detection Details
| Timestamp | Rule ID | Description | Level |
|---|---|---|---|
| 21:29:37 | 554 | File added to the system | 5 |
| 21:31:44 | 550 | Integrity checksum changed | 7 |
| 21:33:15 | 553 | File deleted | 7 |
| — | 533 | Listened ports status (netstat) changed | 7 |

- **MITRE ATT&CK:** T1565 — Data Manipulation (file modification); T1070.004 — Indicator Removal: File Deletion (file deletion)

## Analysis
Wazuh successfully captured the complete file lifecycle in real time: creation, content modification (checksum change), and deletion — each generating a distinct, correctly-leveled alert. The 533 (netstat) alert appeared as ambient system activity during the test window and was not directly related to the FIM test, illustrating the value of correlating alerts by timestamp and host during investigation.

## Recommendation
- Extend FIM monitoring to additional sensitive paths (e.g., `System32`, startup folders, browser extension directories)
- Build a correlation rule flagging file deletion events (553) that follow a modification (550) within a short time window — a common indicator-removal pattern
- Consider enabling Wazuh's Active Response module to automatically quarantine or alert on changes to high-value monitored paths
