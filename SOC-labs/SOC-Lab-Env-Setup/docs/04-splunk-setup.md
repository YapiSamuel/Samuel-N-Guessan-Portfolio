# Phase 4 — Splunk SIEM Deployment

## Overview
This guide covers Splunk Enterprise installation on Ubuntu, Universal Forwarder deployment on Windows endpoints, and log ingestion verification.

---

## Architecture

```
DC01 (192.168.10.10)
  └── Splunk Universal Forwarder
      └── Security, System, Application, Sysmon logs
          └──► port 9997 ──► Splunk Enterprise (192.168.10.40)

WIN10-01 (192.168.10.20)
  └── Splunk Universal Forwarder
      └── Security, System, Application, Sysmon logs
          └──► port 9997 ──► Splunk Enterprise (192.168.10.40)

Splunk Web UI: http://192.168.10.40:8000
```

---

## Splunk Enterprise — Ubuntu Installation

### Download and Install
```bash
# Download Splunk 9.3
wget -O /tmp/splunk.deb "https://download.splunk.com/products/splunk/releases/9.3.0/linux/splunk-9.3.0-51ccf43db5bd-linux-2.6-amd64.deb"

# Install
sudo dpkg -i /tmp/splunk.deb

# First start (creates admin account)
sudo /opt/splunk/bin/splunk start --accept-license

# Enable auto-start on boot
sudo /opt/splunk/bin/splunk enable boot-start
```

### Configure Log Receiving Port
```bash
# Edit inputs.conf directly
sudo nano /opt/splunk/etc/system/local/inputs.conf
```

Add:
```
[splunktcp://9997]
connection_host = ip
```

```bash
# Restart Splunk
sudo /opt/splunk/bin/splunk restart

# Verify port 9997 is listening
sudo ss -tlnp | grep 9997
```

### Configure Firewall
```bash
sudo ufw allow 8000/tcp   # Web interface
sudo ufw allow 9997/tcp   # Log receiving
sudo ufw enable
sudo ufw status
```

### Create wineventlog Index
```bash
sudo /opt/splunk/bin/splunk add index wineventlog -auth admin:YourPassword
```

---

## Splunk Universal Forwarder — Windows Installation

### Download
```
https://download.splunk.com/products/universalforwarder/releases/9.3.0/windows/splunkforwarder-9.3.0-51ccf43db5bd-x64-release.msi
```

### Silent Installation
```powershell
msiexec.exe /i "splunkforwarder-9.3.0-51ccf43db5bd-x64-release.msi" `
  RECEIVING_INDEXER="192.168.10.40:9997" `
  WINEVENTLOG_SEC_ENABLE=1 `
  WINEVENTLOG_SYS_ENABLE=1 `
  WINEVENTLOG_APP_ENABLE=1 `
  AGREETOLICENSE=Yes /quiet
```

### Configure outputs.conf
```powershell
$outputsConf = @"
[tcpout]
defaultGroup = splunk-server

[tcpout:splunk-server]
server = 192.168.10.40:9997
"@

Set-Content "C:\Program Files\SplunkUniversalForwarder\etc\system\local\outputs.conf" $outputsConf
```

### Configure inputs.conf
```powershell
$inputsConf = @"
[WinEventLog://Security]
index = wineventlog
disabled = 0

[WinEventLog://System]
index = wineventlog
disabled = 0

[WinEventLog://Application]
index = wineventlog
disabled = 0

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = wineventlog
disabled = 0
"@

Set-Content "C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf" $inputsConf
```

### Start Forwarder
```powershell
& "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" restart
```

### Verify Connection
```powershell
# Check forwarder logs for connection confirmation
Get-Content "C:\Program Files\SplunkUniversalForwarder\var\log\splunk\splunkd.log" -Tail 20
# Look for: Connected to idx=192.168.10.40:9997
```

---

## Splunk SPL Detection Queries

### Basic Verification
```splunk
# Verify logs are coming in
index=wineventlog | head 20

# Count events by source machine
index=wineventlog | stats count by host
```

### Security Monitoring
```splunk
# Failed logins (brute force detection)
index=wineventlog EventCode=4625
| stats count by Account_Name, IpAddress
| where count > 5
| sort -count

# Successful logins
index=wineventlog EventCode=4624
| table _time, Account_Name, Logon_Type, IpAddress

# Admin group changes
index=wineventlog EventCode=4732
| table _time, MemberSid, GroupName

# New user accounts created
index=wineventlog EventCode=4720
| table _time, SAMAccountName, SubjectUserName
```

### Attack Detection
```splunk
# Kerberoasting - RC4 encryption ticket requests
index=wineventlog EventCode=4769 Ticket_Encryption_Type=0x17
| table _time, Account_Name, Service_Name, Client_Address

# Pass-the-Hash - network logons without password
index=wineventlog EventCode=4624 Logon_Type=3
| search NOT Account_Name="*$"
| table _time, Account_Name, IpAddress

# Lateral movement - PsExec service installation
index=wineventlog EventCode=7045
| search Service_Name="*psexec*" OR Service_Name="*PSEXESVC*"

# PowerShell script block logging
index=wineventlog EventCode=4104
| table _time, host, ScriptBlockText
```

### Sysmon Queries
```splunk
# All process creations
index=wineventlog source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=1
| table _time, host, ParentImage, Image, CommandLine

# Network connections
index=wineventlog source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=3
| table _time, host, Image, DestinationIp, DestinationPort

# DNS queries
index=wineventlog source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=22
| table _time, host, Image, QueryName
```

---

## Port Reference

| Port | Service | Direction |
|---|---|---|
| 8000 | Splunk Web UI | Inbound (analysts) |
| 8089 | Splunk Management | Internal |
| 9997 | Log forwarding | Inbound (forwarders) |

---

## Key Concepts

### Why Two Ports?
- **Port 8000**: Human access — serves the web dashboard for SOC analysts
- **Port 9997**: Machine access — receives raw log data from Universal Forwarders
Separating these ensures web traffic and log traffic don't interfere with each other.

### What is a Universal Forwarder?
A lightweight agent installed on endpoints that:
- Monitors configured log sources continuously
- Compresses and forwards data to Splunk
- Uses minimal CPU/RAM (purpose-built for forwarding only)
- Configured entirely through text files (inputs.conf, outputs.conf)

### What is an Index?
A Splunk index is a dedicated storage partition for a specific type of data. The `wineventlog` index stores all Windows Event Log and Sysmon data from the lab endpoints.
