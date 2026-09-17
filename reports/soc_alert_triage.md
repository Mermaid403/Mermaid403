# Blue Team Lab Report: SOC Alert Triage & Event Monitoring

## 🛡️ Executive Summary
During a routine monitoring cycle within a simulated enterprise Security Operations Center (SOC) environment, a critical security alert was triggered indicating a potential brute-force attack followed by unauthorized privilege escalation on an internal Windows endpoint. This project documents the complete lifecycle of the incident—from initial alert detection and Security Information and Event Management (SIEM) log correlation to incident triage, host isolation, and defensive remediation.

---

## 📈 Topology & Tools Overview
* **SIEM / Log Aggregator:** Elastic Stack (Elasticsearch & Kibana) / LimaCharlie EDR
* **Endpoint Protection:** Universal EDR Agent deployed on target machine
* **Victim Host:** Windows Server 2022 Data Center (`Target-Win-01`)
* **Attacker Host:** External Malicious Node IP (`192.168.10.45`)

---

## 🔍 Incident Timeline & Triage Steps

### Phase 1: Alert Generation (Detection)
A high-severity telemetry alert fired within the dashboard: 
* **Alert Name:** `Suspicious Process Execution - Local Admin Group Modification`
* **Trigger Event:** Event ID 4732 (A member was added to a security-enabled local group).
* **Timestamp:** 2026-09-16T14:22:10Z

### Phase 2: Log Analysis & Correlation (Investigation)
Using Kibana Lucene queries, the authentication logs for `Target-Win-01` were aggregated. The query revealed a cluster of malicious activities preceding the alert:

1. **Brute Force Inbound:** Over 450 failed RDP authentication attempts (`Event ID 4625`) detected from external IP `192.168.10.45` within a 3-minute window, culminating in a single successful logon (`Event ID 4624`) under the compromised user account `j_doe`.
2. **Persistence Mechanics:** Following the breach, the attacker spawned an administrative PowerShell session to force-add a backdoor local account (`badactor`) to the local Administrators group.

```powershell
# Captured Command Line Execution Event
net localgroup administrators badactor /add
```

3. **Telemetry Extraction:** Network sockets audited in Wireshark showed active encrypted C2 (Command & Control) traffic beacons beaconing out to port `443` on the attacker's server infrastructure.

---

## 🛑 Containment & Remediation Actions

1. **Host Isolation:** Using the EDR management console, network isolation commands were instantly pushed to `Target-Win-01` to sever active C2 persistence links and mitigate the threat of lateral movement across production subnets.
2. **Account Revocation:** Discovered compromise vectors were terminated; the `badactor` account was deleted, and the credential set for `j_doe` was forced into expiration and locked.
3. **Rule Tuning & Hardening:** Drafted and deployed an explicit Windows Firewall configuration block against the malicious external IP. Recommended tightening Account Lockout Policies to lock endpoints automatically after 5 sequential failed logon attempts.

---

## 💡 Key Takeaways
This exercise reinforced the vital importance of real-time log ingestion and unified indexing. Recognizing the correlation between clustered failed login sequences and structural account mutations transforms basic alert response into precise, proactive threat hunting.
