# Threat Hunting Investigation: Typo Snare (Ransomware Incident)

## Overview
- **Platform:** TryHackMe
- **Lab:** Threat Hunting Simulator (Typo Snare)
- **Category:** Incident Response / Proactive Threat Hunting
- **Difficulty:** Medium
- **Status:** Completed

## Threat Hypothesis
An attacker gained access to a user's system via a trojanized file downloaded from an untrusted website. Following initial execution, the attacker established persistence, extracted credentials, moved laterally to other systems, and ultimately deployed ransomware.

## Data Sources & Telemetry
- Elastic SIEM
- Sysmon Logs (Event ID 1: Process Creation, Event ID 3: Network Connection)

## Hunting Process & Findings

### 1. Initial Access
- **Query/Hunt:** Investigated Elastic SIEM for suspicious network connections using `winlog.event_id: 3 and destination.ip: 206.189.34.218`.
- **Finding:** Host `WKSTN-03` connected to the malicious IP address `206.189.34.218` on September 26, 2023, at 14:23:46.
- **Evidence:**

![Initial Access - Network Connection](../01-Initial-Access.jpg)

### 2. Execution (Malware Installation)
- **Query/Hunt:** Searched for the execution of the downloaded payload using `winlog.event_id: 1 and "7z2301-x64.msi"`.
- **Finding:** The native Windows installer (`msiexec.exe`) was used to execute a malicious MSI file (`7z2301-x64.msi`) located in the user's Downloads folder.
- **Evidence:**

![Execution - Malicious MSI](../02-Execution-MSI.jpg)

### 3. Defense Evasion & Credential Access
- **Query/Hunt:** Investigated subsequent malicious processes using `winlog.event_id: 1 and "7zipp.dll"`.
- **Finding:** The attacker used a Living off the Land Binary (LOLBIN) technique. They executed a malicious DLL (`7zipp.dll`) using the legitimate `rundll32.exe` process (Defense Evasion). Concurrently, they used PowerShell to download `Invoke-NanoDump.ps1` from GitHub to extract credentials from LSASS memory (Credential Access).
- **Evidence:**

![Defense Evasion and Credential Access](../03-04-DefenseEvasion-NanoDump.jpg)

### 4. Lateral Movement
- **Query/Hunt:** Tracked the attacker's movement to other hosts using `winlog.event_id: 1 and agent.hostname: "WKSTN-02" and process.name: "LogonUI.exe"`.
- **Finding:** The attacker successfully moved laterally to `WKSTN-02`, as evidenced by the `LogonUI.exe` process execution, indicating an active Remote Desktop (RDP) session.
- **Evidence:**

![Lateral Movement - RDP Session](../05-LateralMovement-LogonUI.jpg)

### 5. Impact (Ransomware Deployment)
- **Query/Hunt:** Searched for the final ransomware execution using `winlog.event_id: 1 and "777bomb.exe"`.
- **Finding:** The attacker used a remote PowerShell command (`Invoke-Command`) to download the ransomware payload (`777bomb.exe`) from `7zipp.org` and execute it as `bomb.exe` on `WKSTN-02`.
- **Evidence:**

![Impact - Ransomware Execution](../06-Impact-Ransomware.jpg)

## Conclusion
The investigation confirms the initial hypothesis. The attacker successfully executed a multi-stage attack starting with a typosquatted malware download, leading to credential theft, lateral movement, and the deployment of ransomware across the network.
