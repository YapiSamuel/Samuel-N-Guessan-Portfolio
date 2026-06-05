# Phase 2 — Active Directory Structure

## Overview
This guide covers the complete Active Directory structure built for the SOC lab including OUs, users, groups, and service accounts.

---

## OU Structure

```
corp.local
└── SOC-Lab
    ├── Workstations
    │   └── WIN10-01
    ├── Servers
    │   └── DC01
    ├── Users
    │   ├── IT
    │   │   ├── itadmin
    │   │   └── helpdesk
    │   ├── HR
    │   │   └── jsmith
    │   └── Finance
    │       └── jdoe
    ├── Groups
    │   ├── IT-Admins
    │   ├── HR-Users
    │   ├── Finance-Users
    │   ├── SOC-Analysts
    │   └── Help-Desk
    └── Service Accounts
        ├── svc.sql (Kerberoasting target)
        └── svc.backup
```

---

## PowerShell Commands

### Create OU Structure
```powershell
# Root OU
New-ADOrganizationalUnit -Name "SOC-Lab" -Path "DC=corp,DC=local"

# Child OUs
New-ADOrganizationalUnit -Name "Workstations" -Path "OU=SOC-Lab,DC=corp,DC=local"
New-ADOrganizationalUnit -Name "Servers" -Path "OU=SOC-Lab,DC=corp,DC=local"
New-ADOrganizationalUnit -Name "Users" -Path "OU=SOC-Lab,DC=corp,DC=local"
New-ADOrganizationalUnit -Name "Groups" -Path "OU=SOC-Lab,DC=corp,DC=local"
New-ADOrganizationalUnit -Name "Service Accounts" -Path "OU=SOC-Lab,DC=corp,DC=local"

# Users sub-OUs
New-ADOrganizationalUnit -Name "IT" -Path "OU=Users,OU=SOC-Lab,DC=corp,DC=local"
New-ADOrganizationalUnit -Name "HR" -Path "OU=Users,OU=SOC-Lab,DC=corp,DC=local"
New-ADOrganizationalUnit -Name "Finance" -Path "OU=Users,OU=SOC-Lab,DC=corp,DC=local"
```

### Create Users
```powershell
# HR User - standard victim
New-ADUser -Name "John Smith" -GivenName "John" -Surname "Smith" `
  -SamAccountName "jsmith" -UserPrincipalName "jsmith@corp.local" `
  -Path "OU=HR,OU=Users,OU=SOC-Lab,DC=corp,DC=local" `
  -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) `
  -Enabled $true -Department "HR"

# Finance User
New-ADUser -Name "Jane Doe" -GivenName "Jane" -Surname "Doe" `
  -SamAccountName "jdoe" -UserPrincipalName "jdoe@corp.local" `
  -Path "OU=Finance,OU=Users,OU=SOC-Lab,DC=corp,DC=local" `
  -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) `
  -Enabled $true -Department "Finance"

# IT Admin - high value target
New-ADUser -Name "IT Admin" -GivenName "IT" -Surname "Admin" `
  -SamAccountName "itadmin" -UserPrincipalName "itadmin@corp.local" `
  -Path "OU=IT,OU=Users,OU=SOC-Lab,DC=corp,DC=local" `
  -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) `
  -Enabled $true -Department "IT"

# Help Desk - lateral movement stepping stone
New-ADUser -Name "Help Desk" -GivenName "Help" -Surname "Desk" `
  -SamAccountName "helpdesk" -UserPrincipalName "helpdesk@corp.local" `
  -Path "OU=IT,OU=Users,OU=SOC-Lab,DC=corp,DC=local" `
  -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) `
  -Enabled $true -Department "IT"

# Service Account - SQL (Kerberoasting target)
New-ADUser -Name "SQL Service" -GivenName "SQL" -Surname "Service" `
  -SamAccountName "svc.sql" -UserPrincipalName "svc.sql@corp.local" `
  -Path "OU=Service Accounts,OU=SOC-Lab,DC=corp,DC=local" `
  -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) `
  -Enabled $true

# Service Account - Backup
New-ADUser -Name "Backup Admin" -GivenName "Backup" -Surname "Admin" `
  -SamAccountName "svc.backup" -UserPrincipalName "svc.backup@corp.local" `
  -Path "OU=Service Accounts,OU=SOC-Lab,DC=corp,DC=local" `
  -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) `
  -Enabled $true
```

### Set SPN for Kerberoasting
```powershell
# Register SQL SPN on svc.sql - makes it a Kerberoasting target
Set-ADUser -Identity "svc.sql" -ServicePrincipalNames @{Add="MSSQLSvc/dc01.corp.local:1433"}
```

### Create Groups
```powershell
New-ADGroup -Name "IT-Admins" -GroupScope Global -Path "OU=Groups,OU=SOC-Lab,DC=corp,DC=local"
New-ADGroup -Name "HR-Users" -GroupScope Global -Path "OU=Groups,OU=SOC-Lab,DC=corp,DC=local"
New-ADGroup -Name "Finance-Users" -GroupScope Global -Path "OU=Groups,OU=SOC-Lab,DC=corp,DC=local"
New-ADGroup -Name "SOC-Analysts" -GroupScope Global -Path "OU=Groups,OU=SOC-Lab,DC=corp,DC=local"
New-ADGroup -Name "Help-Desk" -GroupScope Global -Path "OU=Groups,OU=SOC-Lab,DC=corp,DC=local"
```

### Add Members to Groups
```powershell
Add-ADGroupMember -Identity "CN=IT-Admins,OU=Groups,OU=SOC-Lab,DC=corp,DC=local" -Members "itadmin"
Add-ADGroupMember -Identity "CN=HR-Users,OU=Groups,OU=SOC-Lab,DC=corp,DC=local" -Members "jsmith"
Add-ADGroupMember -Identity "CN=Finance-Users,OU=Groups,OU=SOC-Lab,DC=corp,DC=local" -Members "jdoe"
Add-ADGroupMember -Identity "CN=Help-Desk,OU=Groups,OU=SOC-Lab,DC=corp,DC=local" -Members "helpdesk"
Add-ADGroupMember -Identity "Domain Admins" -Members "itadmin"
```

### Move Computers to Correct OUs
```powershell
Move-ADObject -Identity "CN=WIN10-01,CN=Computers,DC=corp,DC=local" `
  -TargetPath "OU=Workstations,OU=SOC-Lab,DC=corp,DC=local"

Move-ADObject -Identity "CN=DC01,OU=Domain Controllers,DC=corp,DC=local" `
  -TargetPath "OU=Servers,OU=SOC-Lab,DC=corp,DC=local"
```

---

## User Accounts Summary

| Name | Username | Department | Privilege | Attack Scenario |
|---|---|---|---|---|
| John Smith | jsmith | HR | Standard | Phishing, credential theft |
| Jane Doe | jdoe | Finance | Standard | Phishing target |
| IT Admin | itadmin | IT | Domain Admin | Ultimate attack target |
| Help Desk | helpdesk | IT | Limited Admin | Lateral movement pivot |
| SQL Service | svc.sql | Service | Standard | Kerberoasting target |
| Backup Admin | svc.backup | Service | Elevated | Lateral movement target |

---

## Key Concepts

### What is an SPN?
A Service Principal Name (SPN) links a service to a user account in Active Directory. When Kerberos issues a ticket for that service, it encrypts the ticket with the service account's password hash — making it crackable offline (Kerberoasting attack).

### Why Service Accounts are High Value Targets
- Often have weak passwords set once and never changed
- Run silently with no interactive login — nobody notices anomalies
- Frequently have elevated privileges for their service function
- SPN registration makes their tickets requestable by any domain user
