# 🔍 SOC Analyst — Hands-On Investigations

> Real-world security investigations and blue team exercises  
> Documenting my journey from networking fundamentals to SOC Analyst

---

## 👩‍💻 About This Repository

This repo contains hands-on cybersecurity investigations completed as part of my SOC Analyst training. Each investigation follows a real analyst workflow:

```
Detect  →  Analyse  →  Extract IOCs  →  Report  →  Recommend Response
```

Every finding here was discovered by working through real malware traffic, security tools, and blue team exercises — not copied from walkthroughs.

---

## 📁 Repository Structure

```
soc-investigations/
│
├── README.md                          ← You are here
│
├── 01-pcap-analysis/
│   ├── investigation-report.md        ← Full investigation report
│   ├── iocs.txt                       ← Extracted indicators of compromise
│   └── screenshots/
│       ├── 01-wireshark-open.png
│       ├── 02-protocol-hierarchy.png
│       ├── 03-http-request-filter.png
│       ├── 04-dns-filter.png
│       ├── 05-tcp-stream-payload.png
│       └── 06-virustotal-confirmation.png
│
├── 02-splunk-detection/               ← Coming soon
│   ├── investigation-report.md
│   └── screenshots/
│
├── 03-phishing-analysis/              ← Coming soon
│   ├── investigation-report.md
│   └── screenshots/
│
└── 04-incident-response-simulation/   ← Coming soon
    ├── investigation-report.md
    └── screenshots/
```

---

## 🧪 Investigations

| # | Investigation | Malware / Topic | Tools Used | Status |
|---|---|---|---|---|
| 01 | [Emotet C2 Traffic Analysis](./01-pcap-analysis/investigation-report.md) | Emotet dropper | Wireshark, VirusTotal | ✅ Complete |
| 02 | Splunk Detection Rules | Brute force, port scan | Splunk, Wazuh | ⏳ Upcoming |
| 03 | Phishing Email Analysis | Credential harvester | Any.run, MXToolbox | ⏳ Upcoming |
| 04 | Incident Response Simulation | Ransomware scenario | CyberDefenders | ⏳ Upcoming |

---

## 🗺️ Learning Roadmap

Following the SOC Analyst complete roadmap — targeting L1 position in 3 months.

| Phase | Topic | Status |
|---|---|---|
| Phase 1 | Networking fundamentals — OSI, TCP/IP, DNS, protocols | ✅ Complete |
| Phase 1 | Wireshark & PCAP analysis | ✅ Complete |
| Phase 1 | Windows Event IDs & Active Directory | 🔄 In progress |
| Phase 2 | Linux log analysis | ⏳ Upcoming |
| Phase 2 | SIEM — Wazuh & Splunk | ⏳ Upcoming |
| Phase 3 | Incident response simulation | ⏳ Upcoming |
| Phase 3 | Phishing analysis | ⏳ Upcoming |
| Phase 4 | Resume, portfolio & job applications | ⏳ Upcoming |

---

## 🛠️ Tools & Platforms

| Tool | Category | Used For |
|---|---|---|
| Wireshark | Network analysis | PCAP investigation, protocol filtering |
| VirusTotal | Threat intel | IOC verification |
| Splunk | SIEM | Log analysis, detection rules |
| Wazuh | SIEM | Home lab monitoring |
| TryHackMe | Training | SOC Level 1 path |
| Any.run | Sandbox | Malware behaviour analysis |
| malware-traffic-analysis.net | Dataset | Real malicious PCAP files |

---

## 📬 Connect

**LinkedIn:** [your LinkedIn URL]  
**TryHackMe:** [your TryHackMe profile URL]  
**networking-notes repo:** [link to your networking-notes repo]

---

*All PCAPs and datasets used are from public security training resources. No live malware was executed.*
