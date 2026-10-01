# Kerberoasting — Attack, Detection & Remediation

**Author:** Samuel N'Guessan
**Environment:** VirtualBox SOC Home Lab (`corp.local`)
**MITRE ATT&CK:** T1558.003 — Kerberoasting

> **Note:** All credentials shown in this write-up are placeholders. Actual lab values have been redacted. This lab runs entirely on an isolated internal network with throwaway accounts — no real systems or secrets are involved.

---

## Objective

Execute a full Kerberoasting attack chain against the lab domain, detect it in Splunk, crack the captured hash, then remediate the vulnerability and validate the fix. The goal was to understand the complete offensive-to-defensive cycle: break it, see it, fix it, prove the fix.

---

## Lab Setup for This Exercise

A service account (`svc.sql`) was created in Active Directory with a registered Service Principal Name:

```
MSSQLSvc/dc01.corp.local:1433
```

Note: no actual SQL Server was installed. The SPN alone is sufficient to make the account a Kerberoasting target, because the attack targets the account's password — not the service behind it. This is an important concept: **a stale or unused SPN is still exploitable.**

---

## Step 1 — The Attack

From the Kali attacker machine, authenticated as a standard low-privilege domain user (`jsmith`):

```bash
impacket-GetUserSPNs corp.local/jsmith:<lab-password> -dc-ip 192.168.10.10 -request
```

**How it works:**
1. Any authenticated domain user can request a service ticket (TGS) for any account with an SPN.
2. The Domain Controller issues the ticket encrypted with the service account's password hash.
3. The attacker captures the ticket and cracks it offline — no account lockout risk, no admin rights required.

### Troubleshooting note
The initial attempt failed with `invalidCredentials: data 532` (expired password). Resolved on the DC:

```powershell
Set-ADAccountPassword -Identity "jsmith" -NewPassword (ConvertTo-SecureString "<lab-password>" -AsPlainText -Force) -Reset
Set-ADUser -Identity "jsmith" -PasswordNeverExpires $true
```

---

## Step 2 — Detection in Splunk

The attack generated a clear, detectable signature. Splunk query:

```splunk
index=wineventlog EventCode=4769
| table _time, Account_Name, Service_Name, Client_Address, Ticket_Encryption_Type
```

**Key indicator:** `Ticket_Encryption_Type=0x17` (RC4). Attackers request RC4-encrypted tickets because they are far easier to crack offline than modern AES. A standard user requesting an RC4 service ticket for a service account is a high-fidelity detection.

### Contrast with reconnaissance
An earlier Nmap scan against the environment produced almost no useful host logs. This reinforced a core SOC lesson: **reconnaissance is hard to detect with host-based logging, while authentication-based attacks generate rich, actionable telemetry.** Network-level detection (IDS) would be needed to reliably catch scanning.

---

## Step 3 — Cracking the Hash

The captured hash (format `$krb5tgs$23$...`) was cracked with Hashcat:

```bash
hashcat -m 13100 kerberoast.txt <wordlist>
```

- `-m 13100` = Kerberos 5 TGS-REP (etype 23 / RC4) — the exact format of the captured ticket.
- The `23` in the hash and the `0x17` in the Splunk log are the same thing (RC4) in different notation.

The weak password was recovered successfully, demonstrating the full compromise: from a powerless HR user to a service account's credentials.

---

## Step 4 — Remediation

### Intended fix: Group Managed Service Account (gMSA)
The enterprise-grade solution is a gMSA, where Active Directory manages a 120+ character password that auto-rotates every 30 days and is never known to any human — making offline cracking infeasible.

**Blocker encountered:** `Add-KdsRootKey` failed repeatedly with `HRESULT 0x80070020` (ERROR_SHARING_VIOLATION — "file in use by another process"), persisting across service reconfiguration and full reboots. AD itself was healthy (NTDS running, domain responsive). The most likely causes in a single-DC nested VM are VirtualBox disk locking or Windows Defender real-time scanning locking the AD database file during the write.

### Applied fix: AES encryption + strong password
Pivoted to the equivalent hardening, which achieves the same security outcome — an uncrackable service account:

```powershell
# Long random password (40+ characters)
Set-ADAccountPassword -Identity "svc.sql" -NewPassword (ConvertTo-SecureString "<40-char-random-password>" -AsPlainText -Force) -Reset

# Enforce AES, remove RC4
Set-ADUser -Identity "svc.sql" -KerberosEncryptionType AES128,AES256

# Verify
Get-ADUser -Identity "svc.sql" -Properties KerberosEncryptionType | Select-Object Name, KerberosEncryptionType
```

---

## Step 5 — Validating the Fix

Re-running the attack after remediation:

| | Before | After |
|---|---|---|
| Ticket encryption | RC4 (`$krb5tgs$23$`) | AES256 (`$krb5tgs$18$`) |
| Password | weak, dictionary-based | 40-char random |
| Crack result | Recovered in seconds | Not cracked (infeasible) |

The combination of AES-only encryption and a long random password rendered the account resistant to offline cracking, closing the vulnerability.

---

## Detection Rule (Keep Active After Remediation)

Defense in depth means hardening the target **and** monitoring for attempts. With AES enforced, any RC4 request becomes even higher signal:

```splunk
index=wineventlog EventCode=4769 Ticket_Encryption_Type=0x17
| stats count by Account_Name, Service_Name, Client_Address
```

---

## Key Takeaways

- Kerberoasting targets the **account password**, not the service — an SPN alone is enough to be exploited.
- RC4 ticket requests (`0x17` / etype 23) are the signature to alert on.
- Host logs catch authentication attacks well; reconnaissance needs network-level detection.
- The vulnerability is a weak, static service account password. gMSA solves this automatically; AES + a long random password solves it manually.
- When the ideal tool is blocked by environmental constraints, knowing an equivalent path to the same security control is what matters.

---

## Tools & Techniques

| Category | Detail |
|---|---|
| Attack | Impacket `GetUserSPNs` |
| Cracking | Hashcat (mode 13100 / 19700) |
| Detection | Splunk SPL, Windows Event 4769 |
| Remediation | AES Kerberos encryption, strong password (gMSA attempted) |
| Framework | MITRE ATT&CK T1558.003 |
