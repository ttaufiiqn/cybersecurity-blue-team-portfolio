# Threat Hunting Investigation: Health Hazard (Supply Chain Attack)

## Overview
- **Platform:** TryHackMe
- **Lab:** Threat Hunting Simulator
- **Category:** Proactive Threat Hunting
- **Difficulty:** Easy
- **Status:** Completed

## Threat Hypothesis
An attacker may have leveraged a compromised third-party software package to gain initial access to the system and silently stage a payload for later execution. They likely established persistence to maintain access without immediate detection.

## Data Sources & Telemetry
- Splunk SIEM
- Sysmon Logs (Event ID 1: Process Creation, Event ID 22: DNS Query, Event ID 13: Registry Value Set)

## Hunting Process & Findings

### 1. Initial Access & Execution
- **Query/Hunt:** Investigated Splunk for malicious process creation.
- **Finding:** A compromised NPM package `healthchk-lib` was installed. The `postinstall` script triggered a hidden, Base64-encoded PowerShell command (Event ID 1).
- **Evidence:**
![Encoded PowerShell Execution](../Screenshot%202026-10-14%20202408_2.png)

### 2. Payload Staging & C2 Communication
- **Query/Hunt:** Decoded the Base64 payload and searched for network connections.
- **Finding:** Decoding the payload revealed a script designed to download `SystemHealthUpdater.exe` from `global-update.wlndows.thm`. Splunk logs (Event ID 22) confirmed the DNS query to this malicious domain.
- **Evidence:**
![CyberChef Payload Decoding](../Screenshot%202026-10-14%20203249_2.png)

![Malicious DNS Query](../Screenshot%202026-10-14%20202250.png)

### 3. Persistence Mechanism
- **Query/Hunt:** Tracked registry modifications made by the malicious PowerShell process.
- **Finding:** The attacker established persistence by creating a Registry Run Key (Event ID 13) named `Windows Update Monitor` to ensure the payload executes automatically upon system reboot.
- **Evidence:**
![Registry Run Key Persistence](../Screenshot%202026-10-14%20202349.png)

## TryHackMe Automated Threat Report
Upon successful reconstruction of the attack chain, the system generated the following executive summary confirming the findings:
![Threat Case Report](../findings_2.png)

## Detection & Remediation Recommendations
1. **Containment:** Isolate the host `paw-tom` from the network.
2. **Eradication:** 
   - Uninstall the `healthchk-lib` package.
   - Delete `SystemHealthUpdater.exe` from the user's `APPDATA` directory.
   - Remove the `Windows Update Monitor` registry key.
3. **Prevention:** Block the domain `global-update.wlndows.thm` and implement strict dependency scanning for NPM packages.
