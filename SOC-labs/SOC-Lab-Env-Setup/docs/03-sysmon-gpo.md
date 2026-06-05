# Phase 3 — Sysmon & GPO Configuration

## Overview
This guide covers Sysmon installation on all Windows endpoints and Group Policy configuration for comprehensive security logging.

---

## Sysmon Installation

### What is Sysmon?
Sysmon (System Monitor) is a Microsoft Sysinternals tool that provides deep visibility into endpoint activity beyond standard Windows Event Logs. It monitors process creation, network connections, file operations, registry changes, and more.

### Downloads Required
- **Sysmon**: https://download.sysinternals.com/files/Sysmon.zip
- **Config**: https://raw.githubusercontent.com/olafhartong/sysmon-modular/master/sysmonconfig.xml

### Installation on DC01
```powershell
# Extract Sysmon
Expand-Archive -Path "C:\Sysmon.zip" -DestinationPath "C:\Sysmon" -Force

# Copy config
Copy-Item "sysmonconfig.xml" "C:\Sysmon\"

# Install with config
cd C:\Sysmon
.\Sysmon64.exe -accepteula -i sysmonconfig.xml

# Verify running
Get-Service Sysmon64

# Verify generating logs
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 5
```

### Installation on WIN10
```powershell
# Same process as DC01
cd C:\Sysmon
.\Sysmon64.exe -accepteula -i sysmonconfig.xml

# Verify
Get-Service Sysmon64
```

### Key Sysmon Event IDs
| Event ID | Description | SOC Value |
|---|---|---|
| 1 | Process creation | Detect malware execution |
| 3 | Network connection | Detect C2 communication |
| 7 | Image loaded (DLL) | Detect DLL injection |
| 8 | CreateRemoteThread | Detect process injection |
| 10 | ProcessAccess | Detect credential dumping |
| 11 | File created | Detect dropper activity |
| 12/13 | Registry events | Detect persistence |
| 22 | DNS query | Detect suspicious DNS |

---

## GPO Configuration

### GPO 1 — Audit Policy
```powershell
New-GPO -Name "Audit Policy" | New-GPLink -Target "OU=SOC-Lab,DC=corp,DC=local"

auditpol /set /subcategory:"Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Account Lockout" /success:enable /failure:enable
auditpol /set /subcategory:"User Account Management" /success:enable /failure:enable
auditpol /set /subcategory:"Security Group Management" /success:enable /failure:enable
auditpol /set /subcategory:"Process Creation" /success:enable /failure:enable
auditpol /set /subcategory:"Credential Validation" /success:enable /failure:enable
```

**Purpose**: Enables Windows Security Event Log entries for critical security events. Without this, most security events are not recorded.

### GPO 2 — PowerShell Script Block Logging
```powershell
New-GPO -Name "PowerShell Logging" | New-GPLink -Target "OU=SOC-Lab,DC=corp,DC=local"

Set-GPRegistryValue -Name "PowerShell Logging" `
  -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" `
  -ValueName "EnableScriptBlockLogging" -Type DWord -Value 1

Set-GPRegistryValue -Name "PowerShell Logging" `
  -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging" `
  -ValueName "EnableModuleLogging" -Type DWord -Value 1
```

**Purpose**: Logs every PowerShell command executed, even obfuscated code. Critical for detecting PowerShell-based attacks like Empire, Invoke-Mimikatz, and similar tools.

### GPO 3 — Disable LLMNR
```powershell
New-GPO -Name "Disable LLMNR" | New-GPLink -Target "OU=SOC-Lab,DC=corp,DC=local"

Set-GPRegistryValue -Name "Disable LLMNR" `
  -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient" `
  -ValueName "EnableMulticast" -Type DWord -Value 0
```

**Purpose**: Disables Link-Local Multicast Name Resolution. Prevents Responder tool from capturing NTLMv2 hashes through LLMNR poisoning attacks.

### GPO 4 — Disable NBT-NS
```powershell
New-GPO -Name "Disable NBT-NS" | New-GPLink -Target "OU=SOC-Lab,DC=corp,DC=local"

Set-GPRegistryValue -Name "Disable NBT-NS" `
  -Key "HKLM\SYSTEM\CurrentControlSet\Services\NetBT\Parameters" `
  -ValueName "NodeType" -Type DWord -Value 2
```

**Purpose**: Disables NetBIOS Name Service. Works alongside LLMNR disable to fully prevent Responder-based hash capture attacks.

### Apply GPOs
```powershell
# Force immediate GPO application on all machines
gpupdate /force

# Verify GPOs applied
gpresult /r
```

---

## Troubleshooting

### GPOs Not Applying
**Symptom**: `gpresult /r` doesn't show custom GPOs

**Cause**: Computers were not inside the SOC-Lab OU

**Fix**:
```powershell
# Verify computer location
Get-ADComputer -Filter * | Select-Object Name, DistinguishedName

# Move computers to correct OUs
Move-ADObject -Identity "CN=WIN10-01,CN=Computers,DC=corp,DC=local" `
  -TargetPath "OU=Workstations,OU=SOC-Lab,DC=corp,DC=local"

Move-ADObject -Identity "CN=DC01,OU=Domain Controllers,DC=corp,DC=local" `
  -TargetPath "OU=Servers,OU=SOC-Lab,DC=corp,DC=local"

gpupdate /force
```

**Key Lesson**: GPOs only apply to objects **inside** the linked OU. If a computer is in the default `CN=Computers` container or another OU, GPOs linked to SOC-Lab have no effect.

---

## Key Windows Event IDs for SOC

| Event ID | Description | Attack Correlation |
|---|---|---|
| 4624 | Successful logon | Baseline, lateral movement |
| 4625 | Failed logon | Brute force detection |
| 4648 | Logon with explicit credentials | Pass-the-hash |
| 4672 | Special privileges assigned | Privilege escalation |
| 4698 | Scheduled task created | Persistence |
| 4720 | User account created | Unauthorized account |
| 4732 | Member added to security group | Privilege escalation |
| 4769 | Kerberos service ticket request | Kerberoasting |
| 7045 | New service installed | Lateral movement (PsExec) |
| 4104 | PowerShell script block logged | Malicious PS execution |
