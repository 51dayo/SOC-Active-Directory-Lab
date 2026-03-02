# 🛡️ Active Directory Home Lab – Attack & Detection with Splunk

![Lab Status](https://img.shields.io/badge/Status-Complete-brightgreen)
![Platform](https://img.shields.io/badge/Platform-VMware-blue)
![SIEM](https://img.shields.io/badge/SIEM-Splunk-orange)
![OS](https://img.shields.io/badge/OS-Windows%20%7C%20Ubuntu%20%7C%20Kali-informational)

---

## 📌 Project Overview

This project is a fully functional **private Security Operations Center (SOC) home lab** built using VMware virtualization. It simulates a small enterprise environment with an Active Directory domain, a SIEM solution, endpoint telemetry collection, and an attacker machine — all operating within an isolated network.

The lab demonstrates how an attacker can target and manipulate an Active Directory environment, and how a defender can detect these activities using Splunk. Threat emulation was performed using Atomic Red Team mapped to the **MITRE ATT&CK framework**.

---

## 🎯 Objectives

- Build and configure a realistic Active Directory environment with domain-joined clients
- Deploy and configure Splunk as a SIEM to collect and analyze endpoint telemetry
- Implement Sysmon for enhanced Windows event logging
- Simulate a brute force attack using Hydra and detect it through Splunk
- Emulate MITRE ATT&CK techniques using Atomic Red Team and validate detection in Splunk
- Develop practical skills in threat detection, log analysis, and attack simulation

---

## 🗂️ Table of Contents

1. [Lab Architecture](#-lab-architecture)
2. [Network Diagram](#-network-diagram)
3. [Tools & Technologies](#-tools--technologies)
4. [Lab Setup](#-lab-setup)
   - [Ubuntu Server – Splunk SIEM](#1-ubuntu-server--splunk-siem)
   - [Windows Client – Endpoint Configuration](#2-windows-client--endpoint-configuration)
   - [Windows Server – Active Directory & DC](#3-windows-server--active-directory--dc)
   - [Kali Linux – Attacker Machine](#4-kali-linux--attacker-machine)
5. [Attack Simulations](#-attack-simulations)
   - [Brute Force Attack with Hydra](#-brute-force-attack-with-hydra)
   - [Atomic Red Team – T1136.001 Local Account Creation](#-atomic-red-team--t1136001-local-account-creation)
   - [Atomic Red Team – T1059.001 PowerShell Execution](#-atomic-red-team--t1059001-powershell-execution)
6. [Detection & Analysis in Splunk](#-detection--analysis-in-splunk)
7. [Key Takeaways](#-key-takeaways)
8. [Future Improvements](#-future-improvements)
9. [References](#-references)

---

## 🏗️ Lab Architecture

The lab consists of four virtual machines connected within the same isolated subnet (`192.168.10.0/24`), all configured with static IP addresses.

| Machine | Role | OS | IP Address |
|---|---|---|---|
| Ubuntu Server | Splunk SIEM | Ubuntu Server | 192.168.10.10 |
| Windows Client | Domain-Joined Endpoint | Windows 10/11 | 192.168.10.3 |
| Windows Server | Active Directory / Domain Controller | Windows Server 2022 | 192.168.10.5 |
| Kali Linux | Attacker Machine | Kali Linux | 192.168.10.200 |

All machines can communicate with each other within the subnet.

---

## 🗺️ Network Diagram

> 📸 *Screenshot: Add your network topology diagram here*

```
192.168.10.0/24 — VMware NAT/Host-Only Network

┌─────────────────────────────────────────────────────┐
│                                                     │
│  [Kali - Attacker]        [Ubuntu - Splunk SIEM]    │
│   192.168.10.200           192.168.10.10            │
│         │                        ▲                  │
│         │   Brute Force /        │ Logs forwarded   │
│         │   Attack traffic       │ via UF           │
│         ▼                        │                  │
│  [Windows Client]         [Windows Server - DC]     │
│   192.168.10.3             192.168.10.5             │
│   Sysmon + UF              Sysmon + UF              │
│   Domain: ben.local        Domain: ben.local        │
└─────────────────────────────────────────────────────┘
```

![Hydra Brute Force Attack](assets/images/hydra-bruteforce.png)
---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|---|---|
| VMware Workstation | Virtualization platform |
| Splunk Enterprise | SIEM – log collection, indexing, and analysis |
| Splunk Universal Forwarder | Forwards endpoint logs to Splunk |
| Sysmon (System Monitor) | Enhanced Windows event logging |
| Active Directory Domain Services | Domain management |
| Hydra | Brute force attack tool |
| Atomic Red Team | MITRE ATT&CK-based threat emulation |
| RDP (Remote Desktop Protocol) | Attack target service |

---

## ⚙️ Lab Setup

### 1. Ubuntu Server – Splunk SIEM

Splunk Enterprise was installed on the Ubuntu server to serve as the central log aggregation and analysis platform.

- Configured to receive logs on port `9997`
- Created an index named `endpoint` for storing Windows telemetry
- Accessible via web interface on port `8000`

> 📸 *Screenshot: `assets/images/splunk-dashboard.png` — Splunk web interface showing the endpoint index*

**Key Splunk configuration:**
- Index created: `endpoint`
- Listening port: `9997`

---

### 2. Windows Client – Endpoint Configuration

The Windows client machine (`192.168.10.3`) was configured as a domain-joined endpoint with full telemetry collection.

**Installed and configured:**
- **Sysmon** – installed with a custom configuration file for enhanced event logging
- **Splunk Universal Forwarder** – configured to forward logs to `192.168.10.10:9997`
- **Atomic Red Team** – installed for threat emulation
- **RDP** – enabled for remote access (used as brute force target)

**`inputs.conf` configured to collect the following event logs:**

```ini
[WinEventLog://Application]
index = endpoint
disabled = false

[WinEventLog://Security]
index = endpoint
disabled = false

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = endpoint
disabled = false

[WinEventLog://System]
index = endpoint
disabled = false
```

> 📸 *Screenshot: `assets/images/inputs-conf-client.png` — inputs.conf configuration on Windows client*

---

### 3. Windows Server – Active Directory & DC

The Windows Server (`192.168.10.5`) was configured as a **Domain Controller** for the domain `ben.local`.

**Steps completed:**

1. Installed **Active Directory Domain Services (AD DS)**
2. Promoted the server to a **Domain Controller**
3. Domain configured: `ben.local`
4. Created two **Organizational Units (OUs)**:
   - `IT`
   - `HR`
5. Created one user account in each OU
6. Configured **Splunk Universal Forwarder** with the same `inputs.conf` endpoints as the client
7. Joined the Windows client (`192.168.10.3`) to the `ben.local` domain

> 📸 *Screenshot: `assets/images/ad-setup.png` — Active Directory Users and Computers showing OUs*

> 📸 *Screenshot: `assets/images/ou-users.png` — IT and HR OUs with user accounts*

After domain join, Splunk was already receiving logs from **two hosts**:

> 📸 *Screenshot: `assets/images/splunk-two-hosts.png` — Splunk showing both hosts forwarding logs*

---

### 4. Kali Linux – Attacker Machine

The Kali machine (`192.168.10.200`) was used to simulate attacks against the Active Directory environment.

**Installed tools:**
- `hydra` – for brute force attacks

---

## 💥 Attack Simulations

### 🔐 Brute Force Attack with Hydra

**Objective:** Simulate a credential brute force attack against the Windows client via RDP using Hydra.

**Target:** Windows Client – `192.168.10.3`  
**Service:** RDP (port 3389)  
**Tool:** Hydra with a subset of the rockyou wordlist

**Command used:**

```bash
hydra -l <username> -P /usr/share/wordlists/rockyou.txt rdp://192.168.10.3
```

> 📸 *Screenshot: `assets/images/hydra-bruteforce.png` — Hydra running the brute force attack*

**Result:** The attack was **successful** — valid credentials were found.

> 📸 *Screenshot: `assets/images/hydra-success.png` — Hydra output showing successful login*

**Detection in Splunk:**

The brute force activity generated a high volume of **Event ID 4625** (failed logon attempts) followed by **Event ID 4624** (successful logon) in the Security event log, visible in the `endpoint` index.

> 📸 *Screenshot: `assets/images/splunk-bruteforce-logs.png` — Splunk showing burst of failed logon events from the brute force*

---

### 🧪 Atomic Red Team – T1136.001 Local Account Creation

**MITRE ATT&CK Technique:** [T1136.001 – Create Account: Local Account](https://attack.mitre.org/techniques/T1136/001/)  
**Tactic:** Persistence

This technique simulates an attacker creating a local user account on the compromised machine to maintain persistence.

**Command run on Windows Client (PowerShell):**

```powershell
Invoke-AtomicTest T1136.001
```

> 📸 *Screenshot: `assets/images/art-T1136-001.png` — Atomic Red Team executing T1136.001*

**Result:** A new local user account was successfully created on the Windows client.

**Detection in Splunk:**

The account creation generated **Event ID 4720** (user account was created), which was captured by Sysmon and forwarded to Splunk.

> 📸 *Screenshot: `assets/images/splunk-T1136-001.png` — Splunk logs confirming new local account creation*

---

### 🧪 Atomic Red Team – T1059.001 PowerShell Execution

**MITRE ATT&CK Technique:** [T1059.001 – Command and Scripting Interpreter: PowerShell](https://attack.mitre.org/techniques/T1059/001/)  
**Tactic:** Execution

This technique simulates an attacker using PowerShell to execute commands or scripts on the compromised system.

**Command run on Windows Client (PowerShell):**

```powershell
Invoke-AtomicTest T1059.001
```

> 📸 *Screenshot: `assets/images/art-T1059-001.png` — Atomic Red Team executing T1059.001*

**Result:** PowerShell execution activity was generated as expected.

**Detection in Splunk:**

Sysmon captured the PowerShell activity (**Event ID 1 – Process Create** and PowerShell script block logging events), which appeared in Splunk under the `endpoint` index.

> 📸 *Screenshot: `assets/images/splunk-T1059-001.png` — Splunk logs showing PowerShell execution telemetry*

---

## 📊 Detection & Analysis in Splunk

All attack activities were successfully detected in Splunk. Below is a summary of the key events generated:

| Attack | Event ID | Log Source | Index |
|---|---|---|---|
| Brute Force (failed attempts) | 4625 | Windows Security Log | endpoint |
| Brute Force (successful logon) | 4624 | Windows Security Log | endpoint |
| Local Account Creation (T1136.001) | 4720 | Windows Security Log | endpoint |
| PowerShell Execution (T1059.001) | Event ID 1 | Sysmon | endpoint |

> 📸 *Screenshot: `assets/images/splunk-summary.png` — Splunk search showing events across all attack scenarios*

**Sample Splunk SPL query to detect brute force:**

```spl
index=endpoint EventCode=4625 
| stats count by src_ip, user 
| where count > 10
| sort -count
```

**Sample SPL query to detect new local account creation:**

```spl
index=endpoint EventCode=4720
| table _time, user, src_user, host
```

---

## 💡 Key Takeaways

- A properly configured SIEM with Sysmon telemetry provides deep visibility into endpoint activity, making it possible to detect even subtle attacker behavior.
- Brute force attacks against RDP generate distinctive log patterns (burst of Event ID 4625) that are straightforward to detect with Splunk.
- Atomic Red Team is a valuable tool for validating detection capabilities against real-world MITRE ATT&CK techniques in a safe, controlled environment.
- Sysmon configuration significantly enhances the quality of logs beyond what Windows default auditing provides.
- Static IP configuration and isolated subnetting are critical for reliable lab communication and realistic traffic analysis.

---

## 🚀 Future Improvements

- [ ] Add a firewall (pfSense) to the network for traffic filtering and analysis
- [ ] Configure Splunk alerting rules for automated threat notification
- [ ] Expand threat emulation with additional MITRE ATT&CK techniques (e.g., lateral movement, credential dumping)
- [ ] Integrate a ticketing system (TheHive or JIRA) for incident response workflow
- [ ] Add a network packet capture tool (Wireshark / Zeek) for network-level detection
- [ ] Simulate a full attack chain from initial access to persistence and exfiltration

---

## 📚 References

- [Splunk Documentation](https://docs.splunk.com/)
- [Sysmon – Microsoft Sysinternals](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [Atomic Red Team](https://github.com/redcanaryco/atomic-red-team)
- [MITRE ATT&CK Framework](https://attack.mitre.org/)
- [Hydra – THC](https://github.com/vanhauser-thc/thc-hydra)
- [Active Directory Documentation – Microsoft](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/)

---

## 👤 Author

**Ben**  
Cybersecurity Enthusiast | SOC Analyst in Training  
> *"Building labs, breaking things, and learning how defenders think."*

---

> ⭐ *If you found this project helpful or interesting, feel free to star the repository!*
