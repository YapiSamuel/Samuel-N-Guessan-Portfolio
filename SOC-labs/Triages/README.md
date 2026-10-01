# 🔍 Triage — SOC Lab Attack & Detection Write-ups

A collection of hands-on security write-ups documenting attacks, detections, and remediations performed in my home SOC lab. Each write-up follows the full cycle: **execute the attack → detect it → understand it → fix it → validate the fix.**

**Author:** Samuel N'Guessan
Aspiring SOC Analyst | MS Cybersecurity, Minnesota State University Moorhead

---

## About This Repository

Every entry here is a self-contained lab exercise run against my VirtualBox SOC environment (`corp.local` domain). The focus isn't just "how to run the attack" — it's the **blue-team perspective**: what the attack looks like in the SIEM, which log sources catch it, and how to remediate and verify.

Each write-up is mapped to the [MITRE ATT&CK](https://attack.mitre.org) framework.

> **Note:** All credentials, hashes, and sensitive values in these write-ups are redacted or replaced with placeholders. The lab runs on an isolated internal network with throwaway accounts — nothing here is real or reachable.

---

## Lab Environment

| Machine | Role | Purpose |
|---|---|---|
| DC01 | Windows Server 2022 | Domain Controller, DNS, AD DS |
| WIN10-01 | Windows 10 | Domain client / attack target |
| Splunk-Ubuntu | Ubuntu + Splunk 9.3 | SIEM (log collection & detection) |
| Kali-Attacker | Kali Linux | Attacker machine |

All endpoints run Sysmon and forward logs to Splunk via Universal Forwarders. GPOs enforce audit logging, PowerShell logging, and attack-surface reduction.

---

## Write-ups

| # | Write-up | MITRE ATT&CK | Status |
|---|---|---|---|
| 01 | [Kerberoasting — Attack, Detection & Remediation](https://github.com/YapiSamuel/Samuel-N-Guessan-Portfolio/blob/main/SOC-labs/Triages/Kerberoasting-writeup.md) | T1558.003 | ✅ Complete |
| 02 | AS-REP Roasting | T1558.004 | 🔄 Planned |
| 03 | Pass-the-Hash / Lateral Movement | T1550.002 | 🔄 Planned |
| 04 | LLMNR/NBT-NS Poisoning | T1557.001 | 🔄 Planned |
| 05 | AD Attack Path Mapping (BloodHound) | — | 🔄 Planned |

---

## Write-up Format

Each write-up follows a consistent structure:

1. **Objective** — what the exercise sets out to prove
2. **Lab Setup** — any configuration specific to the attack
3. **The Attack** — execution and how it works
4. **Detection** — Splunk SPL queries and key log indicators
5. **Analysis** — understanding what happened and why
6. **Remediation** — how to fix the vulnerability
7. **Validation** — re-running the attack to confirm the fix
8. **Key Takeaways** — the lessons that transfer to a real SOC

---

## Skills Demonstrated

- Active Directory attack techniques and defense
- SIEM detection engineering with Splunk (SPL)
- Windows Event Log & Sysmon analysis
- MITRE ATT&CK mapping
- Vulnerability remediation and validation
- Security documentation and reporting

---

## Related

- Main SOC Lab build & architecture: https://github.com/YapiSamuel/Samuel-N-Guessan-Portfolio/tree/main/SOC-labs/SOC-Lab-Env-Setup
