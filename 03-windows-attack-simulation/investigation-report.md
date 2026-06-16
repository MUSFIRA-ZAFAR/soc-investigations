# Investigation 03 — Windows Attack Simulation: Process Creation, Backdoor Account, Fake Service & NTLM Auth

| Field | Details |
|---|---|
| **Date** | 16 June 2026 |
| **Analyst** | Musfira Zafar |
| **Attack Type** | Persistence & Privilege Abuse Simulation |
| **Severity** | 🔴 Critical |
| **Verdict** | CONFIRMED — All 4 attack techniques detected via Windows Event Logs |

---

## 🎯 Objective

Simulate four common attacker techniques on a Windows 11 machine and detect each one using Windows Security Event Logs — proving that every action an attacker takes leaves a trace.

---

## 🛠️ Tools Used

| Tool | Role |
|---|---|
| Windows 11 (Administrator cmd) | Victim machine — ran simulated attacker commands |
| Windows Event Viewer | Detection — found all 4 events in Security log |
| auditpol.exe | Enabled required audit policies |
| Group Policy Editor (gpedit.msc) | Enabled process creation + command line logging |

---

## 🖥️ Lab Environment

| Machine | Role | Details |
|---|---|---|
| Windows 11 laptop | Victim / detection machine | Host OS — WIN-L1J6DSGIRAC |

---

## ⚔️ Simulated Attacks & Detections

---

### Attack 1 — Suspicious Process Creation (Event ID 4688)

**What attackers do:** Run recon commands, launch PowerShell with encoded payloads, use LOLBins to evade detection.

**Simulated command:**
```cmd
net user backdoor2 Password123! /add
powershell.exe -EncodedCommand aQBwAGMAbwBuAGYAaQBnAA==
mshta.exe about:blank
```

**What Windows logged — Event 4688:**

| Field | Value |
|---|---|
| Event ID | 4688 |
| New Process Name | C:\Windows\System32\net.exe |
| Creator Process | C:\Windows\System32\cmd.exe |
| Process Command Line | **net user backdoor2 Password123! /add** |
| Token Elevation Type | TokenElevationTypeFull (2) — ran as full admin |
| Mandatory Label | High Mandatory Level |
| Logged | 6/16/2026 6:10:59 AM |

**Key finding:** Windows logged the **exact command typed** including the password. This is why command line logging in 4688 is so powerful for forensics — attackers can't hide what they ran.

**SOC detection rule:** Alert on 4688 where New Process Name contains `powershell.exe` or `mshta.exe` and Creator Process is `winword.exe`, `excel.exe`, or `outlook.exe`.

**MITRE ATT&CK:** T1059.001 — Command and Scripting Interpreter: PowerShell

---

### Attack 2 — Backdoor Account Created (Event ID 4720)

**What attackers do:** Create a hidden user account for persistent access — even if kicked out, they can log back in via the backdoor account.

**Simulated command:**
```cmd
net user backdoor2 Password123! /add
```

**What Windows logged — Event 4720:**

| Field | Value |
|---|---|
| Event ID | 4720 |
| Task Category | User Account Management |
| Keywords | Audit Success |
| New UAC Value | 0x15 |
| Account Status | Account Disabled initially |
| Computer | WIN-L1J6DSGIRAC |
| Logged | 6/16/2026 6:10:59 AM |

**Key finding:** Every account creation is logged with its UAC flags. In a real environment, any 4720 outside of an approved IT change ticket is an immediate red flag.

**SOC detection rule:** Alert on any 4720 event outside business hours or from a non-IT admin account.

**MITRE ATT&CK:** T1136.001 — Create Account: Local Account

---

### Attack 3 — Malicious Service Installed (Event ID 4697)

**What attackers do:** Install a Windows service pointing to a malware binary in a temp folder — services survive reboots, giving persistent execution.

**Simulated command:**
```cmd
sc create FakeService2 binpath= "C:\Windows\Temp\malware.exe" start= auto
```

**What Windows logged — Event 4697:**

| Field | Value |
|---|---|
| Event ID | 4697 |
| Service Name | **FakeService2** |
| Service File Name | **C:\Windows\Temp\malware.exe** |
| Service Type | 0x10 |
| Service Start Type | 2 (Automatic — starts on boot) |
| Service Account | **LocalSystem** — highest privilege |
| Task Category | Security System Extension |
| Logged | 6/16/2026 6:10:59 AM |

**Key findings:**
- Path is `C:\Windows\Temp\` — legitimate services never live in Temp
- Service account is `LocalSystem` — maximum privilege
- Start type is Automatic — survives every reboot

**SOC detection rule:** Alert on 4697 where Service File Name contains `\Temp\`, `\AppData\`, or `\Downloads\`.

**MITRE ATT&CK:** T1543.003 — Create or Modify System Process: Windows Service

---

### Attack 4 — NTLM Authentication (Event ID 4776)

**What attackers do:** Use stolen NTLM hashes (Pass-the-Hash) to authenticate as privileged accounts without knowing the plaintext password.

**Simulated command:**
```cmd
runas /user:backdoor2 cmd.exe
```

**What Windows logged — Event 4776:**

| Field | Value |
|---|---|
| Event ID | 4776 |
| Authentication Package | MICROSOFT_AUTHENTICATION_PACKAGE_V1_0 |
| Logon Account | **backdoor2** |
| Source Workstation | WIN-L1J6DSGIRAC |
| Error Code | **0x0** — authentication succeeded |
| Task Category | Credential Validation |
| Logged | 6/16/2026 6:11:10 AM |

**Key finding:** Error Code `0x0` = success. The backdoor account authenticated via NTLM. In a real Pass-the-Hash attack, this would show a privileged account authenticating from an unusual workstation.

**SOC detection rule:** Alert on 4776 success (0x0) for admin accounts from workstations that have never previously authenticated as that user.

**MITRE ATT&CK:** T1550.002 — Use Alternate Authentication Material: Pass the Hash

---

## 📸 Screenshot Evidence

| # | File | Event | What It Shows |
|---|---|---|---|
| 1 | `01-4688-process-creation.png` | 4688 | net.exe spawned by cmd.exe — full command line visible |
| 2 | `02-4720-account-created.png` | 4720 | backdoor2 account created — User Account Management |
| 3 | `03-4697-service-installed.png` | 4697 | FakeService2 pointing to C:\Windows\Temp\malware.exe |
| 4 | `04-4776-ntlm-auth.png` | 4776 | backdoor2 NTLM credential validation — Error Code 0x0 |

---

## 🚨 Indicators of Attack Summary

```
========================================
PROCESS CREATION (4688)
========================================
Process     : C:\Windows\System32\net.exe
Parent      : C:\Windows\System32\cmd.exe
Command     : net user backdoor2 Password123! /add
Elevation   : Full admin (TokenElevationTypeFull)

========================================
BACKDOOR ACCOUNT (4720)
========================================
Account     : backdoor2
Created by  : Administrator session
Machine     : WIN-L1J6DSGIRAC
Time        : 6/16/2026 6:10:59 AM

========================================
FAKE SERVICE (4697)
========================================
Name        : FakeService2
Binary path : C:\Windows\Temp\malware.exe
Start type  : Automatic (boot persistence)
Account     : LocalSystem

========================================
NTLM AUTH (4776)
========================================
Account     : backdoor2
Result      : 0x0 (SUCCESS)
Package     : MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
Machine     : WIN-L1J6DSGIRAC
```

---

## 🗺️ MITRE ATT&CK Coverage

| Technique ID | Technique Name | Event Detected |
|---|---|---|
| T1059.001 | PowerShell execution | 4688 |
| T1136.001 | Create Local Account | 4720 |
| T1543.003 | Windows Service persistence | 4697 |
| T1550.002 | Pass the Hash / NTLM auth | 4776 |

---

## 📋 Recommended Response Actions

| Priority | Action |
|---|---|
| 🔴 Immediate | Delete backdoor account — `net user backdoor /delete` |
| 🔴 Immediate | Remove malicious service — `sc delete FakeService` |
| 🟠 High | Audit all local accounts — remove any not in approved list |
| 🟠 High | Audit all installed services — flag any with Temp/AppData paths |
| 🟡 Medium | Enable account lockout policy |
| 🟡 Medium | Restrict `sc create` to administrators only via AppLocker |
| 🟢 Low | Review all 4776 events for unusual NTLM authentications |

---

## 🔎 Detection Rules

**Splunk SPL — Detect backdoor account creation:**
```
index=wineventlog EventCode=4720
| table _time, Account_Name, Subject_Account_Name, Computer
| where NOT Subject_Account_Name IN ("Administrator","IT-Admin")
```

**Splunk SPL — Detect malicious service installation:**
```
index=wineventlog EventCode=4697
| search Service_File_Name="*Temp*" OR Service_File_Name="*AppData*"
| table _time, Service_Name, Service_File_Name, Computer
```

**Splunk SPL — Detect suspicious process creation:**
```
index=wineventlog EventCode=4688
| search New_Process_Name="*powershell*" OR New_Process_Name="*mshta*"
| table _time, Creator_Process_Name, New_Process_Name, Process_Command_Line
```

---

## 📚 What I Learned

- How to enable Windows audit policies using `auditpol` and `gpedit.msc`
- Why command line logging in Event 4688 is critical for forensics
- How attackers create persistence via backdoor accounts (4720) and services (4697)
- What NTLM authentication looks like in logs and how Pass-the-Hash is detected via 4776
- How every attacker action — no matter how small — leaves a Windows Security log entry
- How to map detected techniques to MITRE ATT&CK
- How to write Splunk detection rules from observed attack behaviour

---

*All simulations performed on analyst's own machine in a controlled lab environment for educational purposes.*
