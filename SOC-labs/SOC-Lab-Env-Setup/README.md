# 🛡️ Home SOC Lab — Built by Samuel N'Guessan

A fully functional Security Operations Center (SOC) home lab built from scratch using VirtualBox. This lab simulates a real enterprise environment with Active Directory, endpoint monitoring, SIEM, and attack simulation capabilities.

---

## 📋 Table of Contents
- [Lab Overview](#lab-overview)
- [Architecture](#architecture)
- [Virtual Machines](#virtual-machines)
- [Technologies Used](#technologies-used)
- [Lab Phases](#lab-phases)
- [Setup Guides](#setup-guides)
- [Attack Scenarios](#attack-scenarios)
- [Detection Queries](#detection-queries)
- [Skills Demonstrated](#skills-demonstrated)

---

## 🔍 Lab Overview

This lab was designed to practice real-world SOC skills including:
- Active Directory administration and security
- Endpoint detection and response (EDR) concepts
- SIEM deployment and log management
- Threat detection using Splunk SPL queries
- Attack simulation using Kali Linux
- Windows event log analysis
- Sysmon-based threat hunting

---

## 🏗️ Architecture

```
Internal Network (192.168.10.0/24)
┌─────────────────────────────────────────────────┐
│                                                 │
│   DC01              WIN10-01                    │
│   192.168.10.10     192.168.10.20               │
│   Windows Srv 2022  Windows 10                  │
│   Domain Controller Domain Client               │
│                                                 │
│   Splunk-Ubuntu     Kali-Attacker               │
│   192.168.10.40     192.168.10.50               │
│   Ubuntu + Splunk   Kali Linux                  │
│   SIEM Server       Attacker Machine            │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

## 💻 Virtual Machines

| VM | OS | IP | Role | RAM |
|---|---|---|---|---|
| DC01 | Windows Server 2022 | 192.168.10.10 | Domain Controller, DNS, AD DS | 4GB |
| WIN10-01 | Windows 10 | 192.168.10.20 | Domain Client, Attack Target | 4GB |
| Splunk-Ubuntu | Ubuntu 22.04 | 192.168.10.40 | SIEM Server (Splunk Enterprise) | 4GB |
| Kali-Attacker | Kali Linux | 192.168.10.50 | Attacker Machine | 2GB |

---

## 🛠️ Technologies Used

| Category | Technology |
|---|---|
| Virtualization | Oracle VirtualBox |
| Domain Controller | Windows Server 2022 + AD DS |
| SIEM | Splunk Enterprise 9.3 |
| Log Forwarding | Splunk Universal Forwarder |
| Endpoint Monitoring | Sysmon (SwiftOnSecurity/olafhartong config) |
| Attack Tools | Nmap, Responder, Impacket, CrackMapExec, Hashcat, Bloodhound |
| Scripting | PowerShell, Bash |

---

## 📅 Lab Phases

### ✅ Phase 1 — Network & Domain Setup
- Configured internal network (192.168.10.0/24)
- Installed and configured Windows Server 2022
- Promoted server to Domain Controller
- Created domain: `corp.local`
- Configured DNS
- Joined Windows 10 client to domain

### ✅ Phase 2 — Active Directory Structure
- Created OU hierarchy (SOC-Lab > Workstations, Servers, Users, Groups, Service Accounts)
- Created realistic user accounts (HR, Finance, IT, Help Desk)
- Created security groups with proper membership
- Set up service accounts with SPNs for Kerberoasting simulation
- Promoted itadmin to Domain Admins

### ✅ Phase 3 — Endpoint Monitoring (Sysmon + GPO)
- Installed Sysmon on DC01 and WIN10 with olafhartong modular config
- Configured Group Policy Objects:
  - Audit Policy (logon, account management, process creation)
  - PowerShell Script Block + Module Logging
  - Disable LLMNR (Responder attack prevention)
  - Disable NBT-NS (Responder attack prevention)
- Moved computers to correct OUs for GPO application

### ✅ Phase 4 — Splunk SIEM Deployment
- Installed Splunk Enterprise 9.3 on Ubuntu
- Configured port 9997 for log ingestion
- Created `wineventlog` index
- Installed Splunk Universal Forwarder on DC01 and WIN10
- Configured inputs.conf (Security, System, Application, Sysmon logs)
- Configured outputs.conf (forwarding to 192.168.10.40:9997)
- Verified log ingestion from both Windows machines

### ✅ Phase 5 — Kali Linux Setup
- Installed Kali Linux 2024
- Configured dual network adapters (NAT + Internal)
- Installed attack tools: nmap, responder, impacket, crackmapexec, hashcat, bloodhound
- Verified connectivity to all lab machines

### 🔄 Phase 6 — Attack Simulation & Detection (In Progress)
- Reconnaissance with Nmap
- LLMNR/NBT-NS Poisoning with Responder
- Kerberoasting with Impacket
- Pass-the-Hash attacks
- Lateral Movement simulation

---

## 📁 Setup Guides

| Guide | Description |
|---|---|
| [01-network-setup.md](docs/01-network-setup.md) | VirtualBox network configuration and static IP setup |
| [02-active-directory.md](docs/02-active-directory.md) | AD DS installation, domain creation, and structure |
| [03-sysmon-gpo.md](docs/03-sysmon-gpo.md) | Sysmon installation and GPO configuration |
| [04-splunk-setup.md](docs/04-splunk-setup.md) | Splunk Enterprise and Universal Forwarder setup |
| [05-kali-setup.md](docs/05-kali-setup.md) | Kali Linux configuration and tool installation |
| [06-attack-detection.md](docs/06-attack-detection.md) | Attack scenarios and Splunk detection queries |

---

## ⚔️ Attack Scenarios

| Attack | Tool | Splunk Detection |
|---|---|---|
| Network Reconnaissance | nmap | Event ID 4625 (failed connections) |
| LLMNR Poisoning | Responder | Event ID 4648, network logs |
| Kerberoasting | Impacket GetUserSPNs | Event ID 4769 |
| Pass-the-Hash | CrackMapExec | Event ID 4624 Type 3 |
| Lateral Movement | PsExec | Event ID 7045 (service install) |

---

## 🔎 Detection Queries (Splunk SPL)

```splunk
# Failed login attempts (brute force)
index=wineventlog EventCode=4625 | stats count by Account_Name, src_ip | where count > 5

# Kerberoasting detection
index=wineventlog EventCode=4769 Ticket_Encryption_Type=0x17

# Pass-the-Hash detection
index=wineventlog EventCode=4624 Logon_Type=3 | search NOT Account_Name="*$"

# PowerShell suspicious commands
index=wineventlog source="WinEventLog:Security" EventCode=4104

# Sysmon process creation
index=wineventlog source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=1
```

---

## 🎯 Skills Demonstrated

- **Active Directory** — OU design, user/group management, GPO configuration, SPN management
- **Windows Security** — Event log analysis, audit policy, PowerShell logging
- **SIEM** — Splunk deployment, log forwarding, index management, SPL queries
- **Endpoint Security** — Sysmon deployment and configuration
- **Threat Detection** — Building detection rules for common attacks
- **Attack Simulation** — Kerberoasting, Pass-the-Hash, LLMNR poisoning
- **Linux Administration** — Ubuntu server setup, network configuration, service management
- **Networking** — VLAN concepts, static IP configuration, DNS setup
- **Scripting** — PowerShell automation for AD tasks

---

## 👤 Author

**Samuel N'Guessan**
- Aspiring SOC Analyst
- Building hands-on cybersecurity skills through home lab practice

---

## 📚 Resources

- [Splunk Documentation](https://docs.splunk.com)
- [Sysmon Config by olafhartong](https://github.com/olafhartong/sysmon-modular)
- [MITRE ATT&CK Framework](https://attack.mitre.org)
- [VirtualBox Documentation](https://www.virtualbox.org/manual)
