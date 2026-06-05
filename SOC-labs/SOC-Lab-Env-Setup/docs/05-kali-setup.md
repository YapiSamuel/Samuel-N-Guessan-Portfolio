# Phase 5 — Kali Linux Attacker Setup

## Overview
This guide covers Kali Linux installation, network configuration, and attack tool verification for the SOC lab attacker machine.

---

## VirtualBox Configuration

### Network Adapters
| Adapter | Type | Purpose |
|---|---|---|
| Adapter 1 (eth0) | NAT | Internet access for tool updates |
| Adapter 2 (eth1) | Internal Network (intnet) | Lab attack network |

---

## Installation

### Disk Partitioning
- Method: Guided — use entire disk
- Scheme: All files in one partition
- Size: 50GB virtual disk

### Software Selection
- Desktop environment: Xfce (default Kali)
- Tools: top10 (most popular attack tools)
- Default recommended tools

---

## Network Configuration

### Static IP for Internal Adapter (eth1)
```bash
sudo nano /etc/network/interfaces
```

```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
    address 192.168.10.50
    netmask 255.255.255.0
```

```bash
# Apply configuration
sudo ifup eth1

# Verify
ip a show eth1
```

### DNS Configuration
```bash
sudo nano /etc/resolv.conf
```

```
nameserver 8.8.8.8
nameserver 8.8.4.4
```

```bash
# Make permanent
sudo chattr +i /etc/resolv.conf
```

### Package Repository
```bash
sudo nano /etc/apt/sources.list
```

Add:
```
deb http://http.kali.org/kali kali-rolling main non-free contrib
```

```bash
sudo apt update && sudo apt upgrade -y
```

---

## Attack Tools Installation

```bash
sudo apt install -y \
  nmap \
  crackmapexec \
  responder \
  impacket-scripts \
  hashcat \
  bloodhound \
  neo4j
```

### Tool Reference

| Tool | Purpose | Phase 6 Usage |
|---|---|---|
| nmap | Network scanner | Reconnaissance |
| responder | LLMNR/NBT-NS poisoning | Hash capture |
| impacket | AD attack suite | Kerberoasting, PTH |
| crackmapexec | Network attack framework | Lateral movement |
| hashcat | Password cracking | Crack captured hashes |
| bloodhound | AD attack path mapping | Privilege escalation paths |
| neo4j | Graph database | Bloodhound backend |

---

## Connectivity Verification

```bash
# Test DC01
ping -c 4 192.168.10.10

# Test WIN10
ping -c 4 192.168.10.20

# Test Splunk
ping -c 4 192.168.10.40

# Test internet
ping -c 4 google.com
```

---

## Lab Network Summary from Kali's Perspective

```
Kali (192.168.10.50)
├── → 192.168.10.10 (DC01) - Primary target (Domain Controller)
├── → 192.168.10.20 (WIN10) - Secondary target (Domain client)
└── → 192.168.10.40 (Splunk) - SIEM (monitoring our attacks)
```

> Note: All attacks Kali performs are monitored by Splunk. This is intentional — the lab is designed to both simulate attacks AND detect them.
