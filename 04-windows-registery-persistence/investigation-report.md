# Investigation 04 — Registry Persistence Hunting with Autoruns

| Field | Details |
|---|---|
| **Date** | 17 June 2026 |
| **Analyst** | Musfira Zafar |
| **Technique** | Registry Run Key Persistence + Orphaned Service |
| **Severity** | 🔴 Critical |
| **Verdict** | CONFIRMED — Two persistence artifacts detected and remediated |

---

## 🎯 Objective

Simulate a common malware persistence technique using the Windows Registry Run key, then detect and remediate it using Sysinternals Autoruns — the tool used by real SOC analysts and threat hunters for persistence hunting.

---

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| Registry Editor (regedit.exe) | Visual inspection of Run key entries |
| reg.exe (cmd) | Create and delete registry values |
| Autoruns (Sysinternals) | Automated persistence hunting across all locations |
| VirusTotal | IOC verification (integrated in Autoruns) |

---

## 🖥️ Environment

| Machine | Role |
|---|---|
| Windows 11 (host laptop) | Victim / detection machine — WIN-L1J6DSGIRAC |

---

## ⚔️ Attack Simulation

### Technique — Registry Run Key Persistence (T1547.001)

Attackers write a registry value under the Run key so their malware executes automatically every time a user logs in. The key is:

```
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
```

**Command used to simulate:**
```cmd
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v "WindowsUpdater" /t REG_SZ /d "C:\Windows\Temp\malware.exe" /f
```

**Why this works as an attacker:**
- Survives every reboot — runs on every login automatically
- HKCU requires no admin privileges — any user can write here
- Named `WindowsUpdater` to blend in with legitimate Windows processes (masquerading — T1036)
- Points to Temp folder — common malware staging location

**Verified via reg query:**
```
WindowsUpdater    REG_SZ    C:\Windows\Temp\malware.exe
```

---

## 🔍 Detection — Autoruns Analysis

### What Autoruns Found

Opened Autoruns as Administrator → Logon tab → immediate visual detection.

**Finding 1 — WindowsUpdater (🟡 Yellow — File not found)**

| Field | Value |
|---|---|
| Entry name | WindowsUpdater |
| Registry key | HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run |
| Publisher | **blank — no publisher** |
| Image path | **File not found: C:\Windows\Temp\malware.exe** |
| Color code | 🟡 Yellow — binary missing, entry orphaned |

**Why it stood out:**
- No publisher — every legitimate Windows startup entry has a verified publisher
- Path in `C:\Windows\Temp\` — legitimate software never installs to Temp permanently
- Name mimics Windows Update (`WindowsUpdater`) — classic masquerading

**Finding 2 — FakeService2 (🟡 Yellow — Services tab)**

Autoruns also caught the orphaned service from Investigation 03 in the Services tab:

| Field | Value |
|---|---|
| Entry name | FakeService2 |
| Registry key | HKLM\System\CurrentControlSet\Services |
| Publisher | **blank** |
| Image path | **File not found: C:\Windows\Temp\malware.exe** |
| Color code | 🟡 Yellow — binary missing |

**Key insight:** Autoruns automatically scanned ALL persistence locations and surfaced both artifacts without any manual searching. This is why threat hunters use Autoruns as their first tool.

### Legitimate vs Malicious — Side by Side

| Entry | Publisher | Path | Verdict |
|---|---|---|---|
| OneDrive | (Verified) Microsoft Corporation | C:\Program Files\... | ✅ Legitimate |
| Canva | (Verified) Canva | C:\Users\...\AppData\Local\Programs\... | ✅ Legitimate |
| electron.app.Loom | (Verified) Loom Inc. | C:\Users\...\AppData\Local\Programs\... | ✅ Legitimate |
| **WindowsUpdater** | **blank** | **C:\Windows\Temp\malware.exe** | 🔴 **Malicious** |
| **FakeService2** | **blank** | **C:\Windows\Temp\malware.exe** | 🔴 **Malicious** |

The pattern is clear — legitimate entries always have verified publishers and live in Program Files or AppData\Local\Programs. Malicious entries have no publisher and live in Temp.

---

## 📸 Screenshot Evidence

| # | File | What It Shows |
|---|---|---|
| 1 | `01-regedit-run-key.png` | Registry Editor showing WindowsUpdater alongside legitimate entries |
| 2 | `02-autoruns-logon-malicious.png` | Autoruns Logon tab — WindowsUpdater highlighted yellow |
| 3 | `03-autoruns-services-fakeservice.png` | Autoruns Services tab — FakeService2 highlighted yellow |
| 4 | `04-autoruns-logon-clean.png` | Autoruns Logon tab after remediation — WindowsUpdater removed |

---

## 🚨 Indicators of Persistence

```
========================================
REGISTRY PERSISTENCE
========================================
Key      : HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
Value    : WindowsUpdater
Data     : C:\Windows\Temp\malware.exe
Type     : REG_SZ
Technique: T1547.001 — Boot or Logon Autostart: Registry Run Keys

========================================
SERVICE PERSISTENCE (from Investigation 03)
========================================
Key      : HKLM\System\CurrentControlSet\Services\FakeService2
Binary   : C:\Windows\Temp\malware.exe
Start    : Automatic
Account  : LocalSystem
Technique: T1543.003 — Create or Modify System Process: Windows Service

========================================
MASQUERADING
========================================
Name used: WindowsUpdater
Mimics   : Windows Update processes
Technique: T1036 — Masquerading
```

---

## ✅ Analyst Verdict

**CONFIRMED — Two persistence artifacts detected.**

A registry Run key entry named `WindowsUpdater` was created under `HKCU\...\Run` pointing to `C:\Windows\Temp\malware.exe`. The name was chosen to masquerade as a legitimate Windows process. Autoruns detected it instantly due to missing publisher verification and Temp folder path. An orphaned service entry (`FakeService2`) from a previous simulation was also detected automatically by Autoruns in the Services tab — demonstrating how Autoruns surfaces artifacts across all persistence locations simultaneously.

---

## 🛠️ Remediation

**Step 1 — Remove registry persistence:**
```cmd
reg delete "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v "WindowsUpdater" /f
```

**Step 2 — Remove orphaned service:**
```cmd
sc delete FakeService2
```

**Step 3 — Verify in Autoruns:**
Refresh Autoruns (F5) → confirm no yellow entries remain in Logon or Services tabs.

**Result:** Both artifacts removed. Autoruns clean.

---

## 📋 Recommended Response Actions (Real Incident)

| Priority | Action |
|---|---|
| 🔴 Immediate | Delete malicious Run key entry |
| 🔴 Immediate | Locate and quarantine the actual malware binary |
| 🟠 High | Run Autoruns across ALL endpoints — check for same entry |
| 🟠 High | Search SIEM for any process execution from C:\Windows\Temp\ |
| 🟡 Medium | Check when the entry was created — establish infection timeline |
| 🟡 Medium | Review other persistence locations (Scheduled Tasks, Services, Winlogon) |
| 🟢 Low | Submit malware hash to threat intel platforms |

---

## 🗺️ MITRE ATT&CK Mapping

| Technique ID | Name | What We Simulated |
|---|---|---|
| T1547.001 | Boot or Logon Autostart: Registry Run Keys | WindowsUpdater in HKCU Run key |
| T1543.003 | Create or Modify System Process: Windows Service | FakeService2 in Services |
| T1036 | Masquerading | Named entry "WindowsUpdater" to blend in |

---

## 🔎 Detection Rules

**Splunk SPL — Hunt for Temp folder in Run keys:**
```
index=wineventlog EventCode=4657
Object_Name="*\\CurrentVersion\\Run*"
| search Object_Value_Data="*Temp*" OR Object_Value_Data="*AppData\\Roaming*"
| table _time, Object_Name, Object_Value_Name, Object_Value_Data, SubjectUserName
```

**Persistence hunting checklist (Autoruns):**
```
1. Run Autoruns as Administrator
2. Options → Scan Options → Check VirusTotal.com → Rescan
3. Filter by: Hide Microsoft entries + Hide verified entries
4. Everything remaining = investigate
5. Yellow = file missing (orphaned persistence)
6. Pink = unverified publisher (unknown software)
7. Red = VirusTotal detections (confirmed malicious)
```

---

## 📚 What I Learned

- How attackers use Registry Run keys for boot persistence without admin privileges
- Why `C:\Windows\Temp\` is always a red flag for persistence entries
- How masquerading works — naming malware after legitimate Windows processes
- How Autoruns surfaces ALL persistence locations simultaneously in one view
- The difference between yellow (file missing), pink (unverified), and red (malicious) in Autoruns
- How to verify publishers and use VirusTotal integration inside Autoruns
- How to remediate registry and service persistence artifacts
- How orphaned artifacts from previous incidents remain detectable long after the fact

---

*All simulations performed on analyst's own machine in a controlled lab environment for educational purposes only.*
