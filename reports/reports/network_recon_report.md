# Offensive Security Lab Report: Automated Network Reconnaissance & Port Auditing

## 🎯 Executive Summary
Before defensive teams can protect an environment, offensive security professionals must map the attack surface. This project details a programmatic approach to network reconnaissance, host discovery, and port enumeration. Utilizing Nmap and Bash automation within an isolated virtual laboratory, I simulated a target perimeter audit to discover live hosts, identify running services, and cross-reference active application banners against public CVE vulnerability databases for risk analysis.

---

## 🛠️ Lab Architecture & Toolset
* **Host Platform:** VirtualBox (Isolated Host-Only Subnet)
* **Attacker System:** Kali Linux 2026.x (`192.168.56.102`)
* **Target Environment:** Intentional Multi-Service Vulnerable Host (`192.168.56.101`)
* **Primary Engines:** Nmap (Network Mapper), Bash CLI, Searchsploit

---

## 🚀 Reconnaissance Methodology & Execution

### Phase 1: Live Host Discovery (Ping Sweep)
To map the active architecture without triggering noisy defensive alerts, a rapid ICMP ping sweep was initiated across the entire local subnet range to identify responsive hosts.

```bash
nmap -sn 192.168.56.0/24
```
* **Outcome:** Identified target endpoint live at IP address `192.168.56.101`.

### Phase 2: Comprehensive Enumeration & Service Detection
Following discovery, an aggressive enumeration scan was launched against the target IP to audit open TCP ports, determine exact service versions, and attempt operating system fingerprinting.

```bash
nmap -sV -sC -O -T4 192.168.56.101
```
* `-sV`: Probes open ports to determine service/version info.
* `-sC`: Runs default Nmap NSE scripts to test for common vulnerabilities.
* `-O`: Enables Operating System detection.

### Phase 3: Technical Findings & Banner Analysis
The scan revealed a heavily exposed perimeter. Below are the critical exposure vectors detected during the audit:

| Port / Protocol | Service State | Software Version | Identified Risk Parameter |
| :--- | :--- | :--- | :--- |
| `21 / TCP` | Open | `vsftpd 2.3.4` | High: Well-known command execution backdoor exploit. |
| `22 / TCP` | Open | `OpenSSH 4.7p1` | Medium: Outdated version susceptible to user enumeration. |
| `80 / TCP` | Open | `Apache httpd 2.2.8` | Medium: Vulnerable to denial of service and misconfigurations. |

---

## 🤖 Bash Automation Scripting
To convert manual steps into a repeatable, defensive baseline assessment process, I compiled a simple automation script (`recon.sh`) that accepts a target IP range, exports clean output documentation, and isolates critical risks automatically.

```bash
#!/bin/bash
# Automated Perimeter Audit Script
TARGET=\$1
echo "======= Starting Audit on Target: \$TARGET ======="
nmap -sV --script vuln \$TARGET -oN recon_output.txt
echo "======= Critical Analysis Phase ======="
grep "VULNERABLE" recon_output.txt
```

---

## 🔒 Defensive Hardening Recommendations
Based on the service flags discovered, the target architecture should immediately adopt the following perimeter controls:
1. **Disable Legacy Protocol Access:** Instantly stop or replace unencrypted legacy communication interfaces (like FTP on Port 21) with secure mechanisms like SFTP (SSH File Transfer Protocol).
2. **Implement Regular Patch Schedules:** Update the baseline host operating system dependencies and core software binaries (Apache, OpenSSH) to current stable releases to mitigate known public exploits.
3. **Deploy Network Firewalls:** Implement strict internal host-based or network firewalls to reject unauthorized traffic sweeping and restrict access exclusively to verified source IP networks.
