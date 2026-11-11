# Home Lab SOC Project — Wazuh SIEM Detection Lab

A hands-on security operations lab built from scratch to practice attack simulation, log-based detection, and incident response using **Wazuh** as the SIEM/XDR platform.

## 🎯 Objective

Build a working detection lab that demonstrates the full SOC analyst workflow: deploy a SIEM, onboard an endpoint, simulate realistic attacker behavior, verify detections, map findings to MITRE ATT&CK, and document each incident as a formal report — the same loop performed daily in a real SOC.

## 🏗️ Lab Architecture

| Component | Role | Details |
|---|---|---|
| **Ubuntu VM** | Wazuh Manager (SIEM server) | Wazuh 4.9.2, host-only IP `192.168.56.101` |
| **Windows 10 VM** | Monitored endpoint ("victim") | Wazuh agent `win-victim`, IP `192.168.56.102` |
| **Kali Linux VM** | Attacker machine | Used for network-based attack simulation |
| **Hypervisor** | VirtualBox | NAT + host-only adapter networking between VMs |

All three VMs run isolated on a host-only network, so all traffic and attacks stay fully contained within the lab.

## 🔧 Setup Highlights

- Deployed Wazuh manager on a resource-constrained host (8 GB RAM laptop, ~4.5 GB allocated to the Ubuntu VM) — required careful disk sizing (resized to 50 GB) and troubleshooting a VM freeze during install
- Installed and verified the Wazuh agent on the Windows endpoint, confirmed `Active` status in the dashboard
- Configured custom `syscheck` (File Integrity Monitoring) rules via shared agent group configuration (`/var/ossec/etc/shared/default/agent.conf`)

## 🧪 Attacks Simulated & Detected

### Attack 1 — Brute-Force / Failed Logon Attempts
**Technique simulated:** Repeated failed authentication via `net use` against the Windows endpoint.

| Detail | Value |
|---|---|
| MITRE ATT&CK | T1110 — Brute Force (Credential Access) |
| Rule 60122 | Logon failure (level 5) |
| Rule 60204 | Multiple logon failures (level 10) — Event 4625, Logon Type 3 |
| Rule 60115 | Account locked out (level 9) |
| Total hits | 11 |

**Outcome:** Wazuh correctly escalated severity across the attack sequence (failure → repeated failure → lockout), confirming the account lockout policy functioned as designed.

---

### Attack 2 — Unauthorized Local User Creation (Persistence)
**Technique simulated:** Local account `testuser` created on the Windows endpoint, simulating attacker persistence following a successful compromise.

| Detail | Value |
|---|---|
| MITRE ATT&CK | T1136 — Create Account (Persistence) |
| Rule 60109 | Local user account created (level 8) — Event 4720 |
| Total hits | 2 |

**Outcome:** Account creation detected immediately. In a live environment, correlating this with the preceding brute-force alerts (Attack 1) would indicate a full compromise → persistence attack chain.

---

### Attack 3 — File Integrity Monitoring (FIM) / File Tampering
**Technique simulated:** Monitored directory (`C:\Users\Public\fim_test`) configured via Wazuh `syscheck`; a file was created, modified, and deleted to trigger full FIM lifecycle detection.

| Detail | Value |
|---|---|
| MITRE ATT&CK | T1565 — Data Manipulation / T1070.004 — Indicator Removal (File Deletion) |
| Rule 554 | File added to the system (level 5) — 21:29:37 |
| Rule 550 | Integrity checksum changed (level 7) — 21:31:44 |
| Rule 553 | File deleted (level 7) — 21:33:15 |
| Bonus | Rule 533 — Listened ports status changed (netstat), observed as ambient system activity |

**Outcome:** Full file lifecycle (create → modify → delete) captured in real time via `realtime="yes"` FIM configuration, with content-level diffs enabled via `report_changes="yes"`.

## 📄 Incident Reports

Formal incident reports were written for all three attacks, including summary, detection details, analysis, and recommendations — modeled on real SOC incident documentation standards. See [`/incident-reports`](./incident-reports) for the full write-ups.

## 🗺️ MITRE ATT&CK Coverage Summary

| Attack | Tactic | Technique |
|---|---|---|
| 1 — Brute Force | Credential Access | T1110 |
| 2 — User Creation | Persistence | T1136 |
| 3 — File Tampering | Impact / Defense Evasion | T1565 / T1070.004 |

## 🔑 Key Skills Demonstrated

- SIEM deployment and configuration (Wazuh manager + agent architecture)
- Endpoint log source onboarding and agent management
- Attack simulation across multiple MITRE ATT&CK tactics
- Detection rule analysis and tuning (`ossec_rules.xml`)
- File Integrity Monitoring (FIM) configuration with real-time syscheck
- Incident documentation and reporting
- VirtualBox networking (NAT + host-only) for isolated lab environments

---

*A self-built home-lab project demonstrating practical, hands-on SOC analyst skills.*
