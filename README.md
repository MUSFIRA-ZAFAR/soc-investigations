# 🔍 SOC Analyst — Hands-On Investigations

> Real-world security investigations and blue team exercises  
> Documenting my journey from networking fundamentals to SOC Analyst

---

## 👩‍💻 About This Repository

This repo contains hands-on cybersecurity investigations completed as part of my SOC Analyst training. Each investigation follows a real analyst workflow:

```
Detect  →  Analyse  →  Extract IOCs  →  Report  →  Recommend Response
```

Every finding here was discovered by working through real malware traffic, attack simulations, and Windows Security logs — not copied from walkthroughs. All techniques are mapped to the MITRE ATT&CK framework.

---

## 📁 Repository Structure

```
soc-investigations/
│
├── README.md
│
├── 01-pcap-analysis/
│   ├── investigation-report.md
│   └── screenshots/
│       ├── 01-wireshark-open.png
│       ├── 02-protocol-hierarchy.png
│       ├── 03-http-request-filter.png
│       ├── 04-dns-filter.png
│       ├── 05-tcp-stream-payload.png
│       └── 06-virustotal-confirmation.png
│
├── 02-brute-force-detection/
│   ├── investigation-report.md
│   └── screenshots/
│       ├── 01-hydra-attack.png
│       ├── 02-event-viewer-4625-flood.png
│       └── 03-event-properties-attacker-ip.png
│
└── 03-windows-attack-simulation/
    ├── investigation-report.md
    └── screenshots/
        ├── 01-4688-process-creation.png
        ├── 02-4720-account-created.png
        ├── 03-4697-service-installed.png
        └── 04-4776-ntlm-auth.png
```

---

## 🧪 Investigations

| # | Investigation | Technique | Tools Used | MITRE ATT&CK |
|---|---|---|---|---|
| 01 | [Emotet C2 Traffic Analysis](./01-pcap-analysis/investigation-report.md) | C2 communication, payload delivery | Wireshark, VirusTotal | T1071.001 |
| 02 | [RDP Brute Force Detection](./02-brute-force-detection/investigation-report.md) | Password brute force via RDP | Hydra, Windows Event Viewer | T1110.001 |
| 03 | [Windows Attack Simulation](./03-windows-attack-simulation/investigation-report.md) | Process creation, persistence, NTLM auth | Windows Event Viewer, auditpol | T1059, T1136, T1543, T1550 |

---

## 🗺️ Skills Demonstrated

**Network Analysis**
- PCAP analysis and protocol inspection in Wireshark
- C2 traffic identification and IOC extraction
- DNS evasion technique recognition
- TCP stream analysis

**Windows Security**
- Windows Event Log analysis (Event IDs 4624, 4625, 4688, 4697, 4720, 4776)
- Audit policy configuration via `auditpol` and Group Policy
- Attack simulation and detection — brute force, backdoor accounts, malicious services, NTLM abuse
- Process creation monitoring and command line logging

**Threat Intelligence**
- IOC verification using VirusTotal and AbuseIPDB
- MITRE ATT&CK framework mapping
- Structured incident report writing
- Detection rule writing (Splunk SPL, Wazuh)

**Attack Simulation (Offensive for Defensive purpose)**
- Kali Linux / Hydra RDP brute force
- Windows persistence techniques (services, accounts)
- LOLBins and PowerShell abuse simulation

---

## 🛠️ Tools & Platforms

| Tool | Category | Used In |
|---|---|---|
| Wireshark | Network analysis | Investigation 01 |
| Hydra | Attack simulation | Investigation 02 |
| Windows Event Viewer | Log analysis | Investigation 02, 03 |
| VirusTotal | Threat intelligence | Investigation 01 |
| auditpol.exe | Audit policy | Investigation 03 |
| VirtualBox + Kali Linux | Lab environment | Investigation 02 |
| TryHackMe | Training platform | Ongoing |
| malware-traffic-analysis.net | PCAP dataset | Investigation 01 |

---

## 📊 Event IDs Covered

| Event ID | Name | Investigation |
|---|---|---|
| 4625 | Failed Logon | 02 — Brute Force Detection |
| 4624 | Successful Logon | 02 — Brute Force Detection |
| 4688 | Process Creation | 03 — Windows Attack Simulation |
| 4697 | Service Installed | 03 — Windows Attack Simulation |
| 4720 | User Account Created | 03 — Windows Attack Simulation |
| 4776 | NTLM Credential Validation | 03 — Windows Attack Simulation |

---

## 🗺️ Learning Roadmap

| Phase | Topic | Status |
|---|---|---|
| Phase 1 | Networking fundamentals — OSI, TCP/IP, DNS, protocols | ✅ Complete |
| Phase 1 | Wireshark & PCAP analysis | ✅ Complete |
| Phase 1 | Windows Event IDs & attack simulation | ✅ Complete |
| Phase 2 | Linux log analysis | 🔄 In progress |
| Phase 2 | SIEM — Wazuh & Splunk | 🔄 In progress |
| Phase 3 | Incident response simulation | ⏳ Upcoming |
| Phase 3 | Phishing email analysis | ⏳ Upcoming |

---

## 🔗 Related Repositories

| Repo | Description |
|---|---|
| [networking-notes](https://github.com/MUSFIRA-ZAFAR/networking-notes) | OSI model, TCP/IP, DNS, protocols, devices, subnetting — theory foundations |

---
*All attack simulations and PCAPs used are from public security training resources or performed on the analyst's own machines in a controlled lab environment. No systems were harmed.*
