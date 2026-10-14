# Security Operations Center (SOC) Investigation: Phishing Unfolding

## Overview
- **Platform:** TryHackMe
- **Lab:** SOC Simulator
- **Category:** Security Operations / Incident Investigation
- **Difficulty:** Medium
- **Status:** Completed
- **Objective:** Practice investigating security alerts, analyzing suspicious activity, and documenting incident findings during an active multi-stage breach.

## Skills Demonstrated
- Security alert triage
- Log analysis (Sysmon, Network Logs)
- Incident investigation
- Threat identification (Reconnaissance, Lateral Movement, Data Staging, DNS Tunneling)
- Incident response recommendations

## Investigation Methodology
1. Reviewed the available security alert and supporting evidence across multiple endpoints.
2. Investigated relevant logs and indicators to track attacker lateral movement and exfiltration channels.
3. Assessed the suspicious activity and its potential impact on the network.
4. Documented findings and recommended appropriate response actions for containment and eradication.

## Findings
- **Alert investigated:** Various alerts related to inbound phishing emails, suspicious OS system processes (False Positives), malicious script execution, lateral movement via a mapped network drive, and data exfiltration via DNS (Alerts 1000–1034).
- **Evidence analyzed:**
  - Alerts for legitimate Windows operating system processes incorrectly flagged as threats (False Positives on Alerts 1001, 1002, 1008, 1009, 1012).
  - Multiple inbound phishing email attempts and untrusted external domains (e.g., Alert 1000 and 1004).
  - Evidence of compromise on host `win-3450` involving the creation of the `PowerView.ps1` reconnaissance script.
  - A network drive disconnected from a local drive (Alert 1024), executed after the attacker mapped the network share and staged data.
  - A sequence of suspicious parent-child process relationships (Alerts 1027, 1028) involving DNS queries using `nslookup.exe` to the domain `haz4rdw4re.io` with Base64-encoded subdomains, confirming active data exfiltration.
- **Assessment:** The investigation identified a multi-stage attack that compromised host `win-3450`. The attacker established access, conducted network reconnaissance, laterally accessed network shares to collect data, attempted defense evasion by disconnecting the mapped network drive, and actively exfiltrated sensitive data using DNS tunneling to adversary-controlled infrastructure. Concurrently, a significant volume of alerts triggered during this timeframe were verified as normal, benign Windows system behaviors. **Verdict: Mixed** (A combination of False Positives for standard operating system processes and True Positives for advanced compromise and data exfiltration activity).
- **Recommended action:** Immediately isolate host `win-3450` from the corporate network and implement a block for the domain `haz4rdw4re.io` (including all subdomains) at the perimeter firewall and DNS resolvers. Terminate the active malicious processes (PowerShell and nslookup) on the compromised host, and reset the credentials for the affected user account. Furthermore, tune the Sysmon detection rules to reduce False Positives for legitimate Windows internal tasks.

## Key Takeaways
Gained hands-on experience tracking a live attacker within the network, recognizing post-exploitation techniques such as AD reconnaissance (PowerView), data staging, and DNS tunneling for exfiltration. Improved ability to filter out noisy false positives during high-stress alert triage.

## Evidence
See the screenshots included in this repository, with sensitive information removed.

## Disclaimer
This project documents my personal learning experience in an authorized TryHackMe lab environment. All findings are based on the work I actually performed.
