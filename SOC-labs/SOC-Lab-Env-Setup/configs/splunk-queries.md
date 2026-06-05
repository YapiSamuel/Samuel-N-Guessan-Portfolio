# Splunk Detection Queries — SOC Lab
# Author: Samuel N'Guessan
# All queries use index=wineventlog

# ============================================================
# BASIC VERIFICATION
# ============================================================

# Verify logs are ingesting
index=wineventlog | head 20

# Count events by host
index=wineventlog | stats count by host

# Events by source type
index=wineventlog | stats count by source


# ============================================================
# AUTHENTICATION MONITORING
# ============================================================

# All successful logins
index=wineventlog EventCode=4624
| table _time, Account_Name, Logon_Type, IpAddress, host

# Failed logins (brute force indicator)
index=wineventlog EventCode=4625
| stats count by Account_Name, IpAddress
| where count > 5
| sort -count

# Failed logins over time
index=wineventlog EventCode=4625
| timechart count by Account_Name

# Logins with explicit credentials (pass-the-hash indicator)
index=wineventlog EventCode=4648
| table _time, Account_Name, TargetServerName, IpAddress

# Admin privilege use
index=wineventlog EventCode=4672
| table _time, Account_Name, host


# ============================================================
# ACCOUNT MANAGEMENT
# ============================================================

# New user accounts created
index=wineventlog EventCode=4720
| table _time, SAMAccountName, SubjectUserName, host

# Users added to security groups
index=wineventlog EventCode=4732
| table _time, MemberSid, GroupName, SubjectUserName

# Users added to Domain Admins specifically
index=wineventlog EventCode=4732 GroupName="Domain Admins"
| table _time, MemberSid, SubjectUserName


# ============================================================
# KERBEROS ATTACKS
# ============================================================

# Kerberoasting detection (RC4 encrypted tickets)
index=wineventlog EventCode=4769 Ticket_Encryption_Type=0x17
| table _time, Account_Name, Service_Name, Client_Address

# Multiple TGS requests (Kerberoasting pattern)
index=wineventlog EventCode=4769
| stats count by Account_Name, Client_Address
| where count > 3
| sort -count

# AS-REP Roasting (accounts with no pre-auth required)
index=wineventlog EventCode=4768 Pre_Authentication_Type=0
| table _time, Account_Name, Client_Address


# ============================================================
# LATERAL MOVEMENT
# ============================================================

# Network logons (lateral movement indicator)
index=wineventlog EventCode=4624 Logon_Type=3
| search NOT Account_Name="*$"
| table _time, Account_Name, IpAddress, host

# PsExec / remote service installation
index=wineventlog EventCode=7045
| table _time, host, Service_Name, Service_File_Name

# Remote scheduled task creation
index=wineventlog EventCode=4698
| table _time, host, TaskName, SubjectUserName


# ============================================================
# POWERSHELL MONITORING
# ============================================================

# All PowerShell script blocks
index=wineventlog EventCode=4104
| table _time, host, ScriptBlockText

# Suspicious PowerShell keywords
index=wineventlog EventCode=4104
| search ScriptBlockText="*mimikatz*" OR ScriptBlockText="*invoke-expression*" OR ScriptBlockText="*downloadstring*" OR ScriptBlockText="*bypass*"
| table _time, host, ScriptBlockText


# ============================================================
# SYSMON QUERIES
# ============================================================

# All process creations
index=wineventlog source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=1
| table _time, host, ParentImage, Image, CommandLine

# Network connections
index=wineventlog source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=3
| table _time, host, Image, DestinationIp, DestinationPort

# Connections to Kali (attacker IP)
index=wineventlog source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=3
DestinationIp="192.168.10.50"
| table _time, host, Image, DestinationIp, DestinationPort

# DNS queries
index=wineventlog source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=22
| table _time, host, Image, QueryName

# File creation events
index=wineventlog source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=11
| table _time, host, Image, TargetFilename


# ============================================================
# DASHBOARD QUERIES
# ============================================================

# Top 10 failed login accounts (last 24h)
index=wineventlog EventCode=4625 earliest=-24h
| stats count by Account_Name
| sort -count
| head 10

# Event timeline (last 1 hour)
index=wineventlog earliest=-1h
| timechart count by EventCode

# Unique source IPs connecting to DC
index=wineventlog host=DC01 EventCode=4624 earliest=-24h
| stats dc(IpAddress) as unique_ips, count by Account_Name
| sort -count
