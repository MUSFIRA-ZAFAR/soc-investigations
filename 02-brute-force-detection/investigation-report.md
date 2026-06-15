# Brute Force Attack Detection Using Windows Event Viewer

## Project Overview

This project demonstrates the detection and investigation of a brute-force authentication attack in a controlled lab environment. The objective was to simulate repeated login attempts from a Kali Linux system against a Windows 11 machine and identify the attack using native Windows security logs.

## Lab Environment

### Attacker System

* Kali Linux
* VirtualBox Virtual Machine

### Target System

* Windows 11 Host Machine

### Monitoring Tool

* Windows Event Viewer

### Attack Technique

* Password brute-force authentication attempts

---

## Objective

The goal of this investigation was to:

1. Generate multiple failed authentication attempts from Kali Linux.
2. Capture the resulting Windows Security Events.
3. Identify indicators of brute-force activity.
4. Determine the source of the attack.
5. Document findings as a SOC-style investigation.

---

## Attack Execution

### Step 1: Connectivity Verification

The Kali Linux machine successfully communicated with the Windows 11 system using ICMP (ping), confirming network connectivity between the attacker and target systems.

### Step 2: Monitoring Preparation

Before generating the attack, Event Viewer was opened on Windows 11.

Location:

Windows Logs → Security

This allowed real-time monitoring of authentication events during testing.

### Step 3: Password List Creation

A custom password list containing multiple incorrect passwords was created on the Kali Linux machine to simulate repeated authentication failures.

### Step 4: Authentication Attack Simulation

Multiple authentication attempts were generated from Kali Linux against the Windows 11 system.

The purpose was not to gain access but to trigger failed authentication events for detection and analysis.

---

## Detection Process

### Event Viewer Investigation

The Security log was filtered using:

Event ID: 4625

Event ID 4625 represents a failed logon attempt.

A noticeable cluster of failed authentication events appeared within a short time period, indicating potential brute-force activity.

### Key Indicators Observed

* Multiple failed logon attempts
* Repeated failures within a short timeframe
* Consistent targeting of the same account
* Authentication failures originating from a remote system

---

## Evidence Collected

### Security Event ID

Event ID: 4625

### Important Fields

| Field                  | Observation     |
| ---------------------- | --------------- |
| Event ID               | 4625            |
| Log Type               | Security        |
| Audit Status           | Audit Failure   |
| Authentication Package | NTLM            |
| Source Network Address | 192.168.1.8     |
| Workstation Name       | kali            |
| Target Computer        | Windows 11 Host |

### Findings

The source workstation was identified as:

Workstation Name: kali

Source IP Address:

192.168.1.8

These details directly linked the failed authentication attempts to the Kali Linux attack system.

---

## Investigation Summary

The investigation confirmed a brute-force authentication attempt against a Windows 11 system.

Evidence supporting this conclusion included:

* Repeated Event ID 4625 entries
* Multiple authentication failures in a short period
* Identification of the attacking host
* Source IP attribution
* NTLM authentication failures

The attack was successfully detected using native Windows logging without requiring a SIEM platform.

---

## Lessons Learned

* Event ID 4625 is critical for detecting failed logon attempts.
* Event Viewer can be used effectively for basic brute-force investigations.
* Source IP addresses and workstation names provide valuable attribution data.
* Repeated authentication failures over a short period are strong indicators of brute-force activity.
* Proper log analysis enables rapid identification of suspicious authentication behavior.

---

## MITRE ATT&CK Mapping

| Tactic            | Technique           |
| ----------------- | ------------------- |
| Credential Access | T1110 - Brute Force |

---

## Conclusion

This lab successfully demonstrated the complete attack-to-detection lifecycle:

Kali Linux Attack Simulation → Failed Authentication Attempts → Windows Security Logging → Event ID 4625 Analysis → Attacker Identification

The exercise provided practical experience in detecting brute-force activity and performing basic SOC-style investigations using Windows Event Viewer.
