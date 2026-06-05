# Phase 1 — Network & Domain Setup

## Overview
This guide covers the complete network configuration for the SOC lab including VirtualBox setup, static IP configuration, and domain creation.

---

## VirtualBox Network Configuration

### Internal Network Setup
All VMs communicate through an internal VirtualBox network named `intnet`.

| VM | Adapter 1 | Adapter 2 |
|---|---|---|
| DC01 | Internal Network (intnet) | — |
| WIN10-01 | Internal Network (intnet) | — |
| Splunk-Ubuntu | NAT (internet) | Internal Network (intnet) |
| Kali-Attacker | NAT (internet) | Internal Network (intnet) |

---

## DC01 — Static IP Configuration

### Problem Encountered
The sconfig GUI menu failed to save static IP settings due to DHCP override. Resolved using PowerShell directly.

### Solution
```powershell
# Disable DHCP on the internal adapter
Set-NetIPInterface -InterfaceAlias "Ethernet 2" -Dhcp Disabled

# Set static IP
New-NetIPAddress -InterfaceAlias "Ethernet 2" -IPAddress 192.168.10.10 -PrefixLength 24

# Set DNS to loopback (DC points to itself)
Set-DnsClientServerAddress -InterfaceAlias "Ethernet 2" -ServerAddresses 127.0.0.1

# Verify
ipconfig /all
```

### Why 127.0.0.1 for DNS on DC
The Domain Controller runs its own DNS server. Pointing DNS to 127.0.0.1 (loopback) ensures the DC resolves domain names through itself, which is more reliable during boot than pointing to its own NIC IP.

---

## WIN10-01 — Static IP Configuration

```powershell
# Disable DHCP
Set-NetIPInterface -InterfaceAlias "Ethernet" -Dhcp Disabled

# Set static IP
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.10.20 -PrefixLength 24

# Set DNS to DC01
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.10.10

# Verify
ipconfig /all
```

---

## Splunk-Ubuntu — Static IP Configuration

### Netplan Configuration
```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      addresses:
        - 192.168.10.40/24
      nameservers:
        addresses:
          - 192.168.10.10
```

```bash
# Fix permissions and apply
sudo chmod 600 /etc/netplan/01-netcfg.yaml
sudo netplan apply

# Verify
ip a show enp0s8
```

---

## Domain Controller Setup

### Install AD DS Role
```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```

### Promote to Domain Controller
```powershell
Install-ADDSForest `
  -DomainName "corp.local" `
  -DomainNetbiosName "CORP" `
  -ForestMode "WinThreshold" `
  -DomainMode "WinThreshold" `
  -InstallDns:$true `
  -Force:$true
```

---

## Join WIN10 to Domain

```powershell
Add-Computer -DomainName "corp.local" -Credential corp\Administrator -Restart
```

---

## Connectivity Verification

```powershell
# From WIN10, ping DC01
ping 192.168.10.10

# From DC01, ping WIN10
ping 192.168.10.20

# DNS resolution test
nslookup corp.local 192.168.10.10
```

---

## Key Lessons Learned

1. **DHCP overrides manual settings** — Always disable DHCP before setting a static IP
2. **APIPA addresses (169.254.x.x)** — Indicate no DHCP lease was obtained; machine assigned itself a fallback address
3. **PowerShell vs sconfig** — PowerShell commands operate at a lower level and are more reliable for persistent network changes in VM environments
4. **DC DNS should point to itself** — Use 127.0.0.1, not the DC's own NIC IP, to avoid resolution issues during boot
