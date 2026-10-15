# Lab 03 — Organizational Unit (OU) Management
## Windows Server 2019 | Active Directory | `zroots.local`

**Lab Status:** Completed  
**Domain Controller:** DC01  
**Operating System:** Windows Server 2019  
**Active Directory Domain:** `zroots.local`  
**Primary Tools:** Windows PowerShell, Active Directory Domain Services, Group Policy Management  
**Lab Network:** `10.1.1.0/24`

---

## Table of Contents

1. [Lab Overview](#1-lab-overview)
2. [Objectives](#2-objectives)
3. [Lab Environment and Topology](#3-lab-environment-and-topology)
4. [Prerequisites](#4-prerequisites)
5. [Organizational Units Explained](#5-organizational-units-explained)
6. [Lab Tasks and Procedures](#6-lab-tasks-and-procedures)
7. [Organizational Unit Inventory](#7-organizational-unit-inventory)
8. [Computer Account Placement](#8-computer-account-placement)
9. [Group Policy Inheritance](#9-group-policy-inheritance)
10. [Accidental Deletion Protection](#10-accidental-deletion-protection)
11. [Verification Results](#11-verification-results)
12. [Troubleshooting Notes](#12-troubleshooting-notes)
13. [Security Lessons Learned](#13-security-lessons-learned)
14. [Knowledge Check](#14-knowledge-check)
15. [Lab Conclusion](#15-lab-conclusion)
16. [Appendix: Useful Commands](#16-appendix-useful-commands)

---

## 1. Lab Overview

This lab demonstrates how to organize an Active Directory domain using Organizational Units (OUs). OUs provide a logical structure for managing users, computers, servers, service accounts, and groups. They also allow administrators to apply Group Policy Objects (GPOs) to specific parts of the directory.

The lab was performed on the Windows Server 2019 domain controller **DC01** in the `zroots.local` domain.

The work included:

- Inspecting the existing OU structure.
- Verifying OU distinguished names and deletion-protection settings.
- Creating a `Linux` OU beneath `Computer Accounts`.
- Confirming the locations of domain-joined computer accounts.
- Inspecting GPO links and inheritance.
- Enabling accidental-deletion protection for the `Domain Controllers` OU.
- Verifying the resulting configuration with PowerShell.

> **Scope note:** The commands and results in this report are based on the lab outputs recorded during the session. Where an actual result was not recorded, it is identified as a recommended validation rather than a confirmed result.

## 2. Objectives

By completing this lab, the learner should be able to:

1. Explain the purpose of Organizational Units in Active Directory.
2. Inspect OUs and their properties using PowerShell.
3. Understand distinguished names (DNs) and the OU hierarchy.
4. Create a child OU in an existing directory structure.
5. Locate computer accounts within the domain.
6. Inspect Group Policy links and inheritance.
7. Enable and verify protection against accidental OU deletion.
8. Document configuration evidence and identify security implications.

## 3. Lab Environment and Topology

### 3.1 Environment details

| Component | Configuration |
|---|---|
| Domain controller | DC01 |
| Server operating system | Windows Server 2019 |
| Active Directory domain | `zroots.local` |
| Domain controller LAN address | `10.1.1.10` |
| Private lab network | `10.1.1.0/24` |
| Administration tool | Windows PowerShell |
| Directory service | Active Directory Domain Services |
| Policy management | Group Policy Management |
| Lab topic | Organizational Unit Management |

### 3.2 Logical topology

```text
Internet
   |
Physical Router
   |
VMware VMnet0 — Bridged WAN
   |
DC01 — Windows Server 2019
   |  AD DS
   |  DNS
   |  RRAS/NAT
   |  LAN: 10.1.1.10
   |
MyLabNet — Host-only Network
10.1.1.0/24
   |
   +-- Windows 10 client: 10.1.1.20
   +-- Kali Linux:        10.1.1.30
   +-- BlackArch:         10.1.1.40
   +-- Windows 11 client: 10.1.1.50
   +-- Metasploitable:    10.1.1.60
```

**Important:** The listed endpoints are part of the wider lab environment. This OU lab specifically verifies the recorded Active Directory objects and their directory locations; it does not claim that every listed endpoint is domain-joined.

### 3.3 Simplified OU hierarchy

```text
zroots.local
|
+-- Domain Controllers
|   +-- DC01
|
+-- Computer Accounts
|   +-- Windows 10
|   |   +-- WIN10-01
|   |
|   +-- Windows 11
|   |   +-- WINDOWS11
|   |
|   +-- Servers
|   |
|   +-- Linux
|
+-- User Accounts
|   +-- Staff
|   +-- Students
|
+-- Administrators
+-- Groups
+-- Service Accounts
```

This is a simplified representation of the relevant OU structure. The placement of `DC01` under `Domain Controllers` is confirmed by its recorded distinguished name. Not every OU shown above necessarily contains objects.

## 4. Prerequisites

Before starting this lab, ensure that:

- Windows Server 2019 is installed and operational.
- Active Directory Domain Services is installed and configured.
- The `zroots.local` domain is available.
- You can sign in to DC01 with an account authorized to manage OUs.
- The Active Directory PowerShell module is available.
- You have completed the earlier Active Directory and Group Policy labs.
- You have recorded the current configuration before making changes.

Open PowerShell as an administrator on DC01.

Check the computer name and domain:

```powershell
hostname
Get-ADDomain | Select-Object DNSRoot, NetBIOSName, DomainMode
```

**Expected result:** The server name is `DC01`, and the DNS domain is `zroots.local`.

**Evidence status:** The recorded lab environment confirms `DC01` and `zroots.local`.

## 5. Organizational Units Explained

An Organizational Unit is a container within an Active Directory domain. It can contain users, computers, groups, and other OUs.

### 5.1 Why OUs are important

OUs help administrators:

- Organize directory objects by department, function, device type, or administrative responsibility.
- Delegate administrative permissions to selected teams.
- Apply GPOs to a defined group of users or computers.
- Separate server and workstation administration.
- Improve the clarity and maintainability of the directory.

### 5.2 OU versus security group

An OU is primarily a container used for organization, delegation, and Group Policy scope. A security group is primarily used to assign permissions and control access to resources.

For example, an OU named `Staff` can contain staff user accounts, while a security group named `Staff-File-Access` can be used to grant access to a shared folder.

### 5.3 Understanding distinguished names

A distinguished name identifies an object and describes its location in the directory hierarchy.

Example:

```text
OU=Windows 10,OU=Computer Accounts,DC=zroots,DC=local
```

This means:

- `OU=Windows 10` — the child OU.
- `OU=Computer Accounts` — its parent OU.
- `DC=zroots,DC=local` — the domain components of `zroots.local`.

A computer object may have a DN such as:

```text
CN=WIN10-01,OU=Windows 10,OU=Computer Accounts,DC=zroots,DC=local
```

Here, `CN=WIN10-01` identifies the computer object.

## 6. Lab Tasks and Procedures

### Task 1 — Inspect existing Organizational Units

Run:

```powershell
Get-ADOrganizationalUnit -Filter * |
    Select-Object Name, DistinguishedName
```

This command lists the OUs visible in the domain and their distinguished names.

To inspect deletion-protection settings:

```powershell
Get-ADOrganizationalUnit -Filter * `
    -Properties ProtectedFromAccidentalDeletion |
    Select-Object Name, DistinguishedName, ProtectedFromAccidentalDeletion
```

**Expected result:** A list of OUs, their DNs, and their accidental-deletion protection settings.

**Recorded result:** Twelve OUs were enumerated during the lab. Their names and recorded protection settings are listed in Section 7.

**Evidence to capture:** A screenshot of the PowerShell output showing OU names and protection settings.

### Task 2 — Create the Linux OU

The `Linux` OU was created beneath the existing `Computer Accounts` OU.

Command used:

```powershell
New-ADOrganizationalUnit `
    -Name "Linux" `
    -Path "OU=Computer Accounts,DC=zroots,DC=local" `
    -ProtectedFromAccidentalDeletion $true
```

Explanation:

- `New-ADOrganizationalUnit` creates an OU.
- `-Name "Linux"` sets the new OU's name.
- `-Path` identifies its parent container.
- `-ProtectedFromAccidentalDeletion $true` enables the protection setting.

The intended distinguished name is:

```text
OU=Linux,OU=Computer Accounts,DC=zroots,DC=local
```

**Recorded result:** The command was executed on DC01. The OU was subsequently included in the recorded OU inventory with deletion protection enabled.

Verify it with:

```powershell
Get-ADOrganizationalUnit `
    -Identity "OU=Linux,OU=Computer Accounts,DC=zroots,DC=local" `
    -Properties ProtectedFromAccidentalDeletion |
    Select-Object Name, DistinguishedName, ProtectedFromAccidentalDeletion
```

**Expected result:** The object name is `Linux`, the DN matches the path above, and `ProtectedFromAccidentalDeletion` is `True`.

**Evidence to capture:** The command and its returned properties.

### Task 3 — Inspect computer accounts

Use the following command to list computer objects and their locations:

```powershell
Get-ADComputer -Filter * `
    -Properties DistinguishedName |
    Select-Object Name, DistinguishedName
```

To inspect the Windows 10 computer:

```powershell
Get-ADComputer -Identity "WIN10-01" |
    Select-Object Name, DistinguishedName
```

To inspect the Windows 11 computer:

```powershell
Get-ADComputer -Identity "WINDOWS11" |
    Select-Object Name, DistinguishedName
```

To inspect the domain controller:

```powershell
Get-ADComputer -Identity "DC01" |
    Select-Object Name, DistinguishedName
```

Recorded distinguished names:

```text
WIN10-01:
CN=WIN10-01,OU=Windows 10,OU=Computer Accounts,DC=zroots,DC=local

WINDOWS11:
CN=WINDOWS11,OU=Windows 11,OU=Computer Accounts,DC=zroots,DC=local

DC01:
CN=DC01,OU=Domain Controllers,DC=zroots,DC=local
```

**Result:** The recorded computer-object locations match the intended OU hierarchy.

**Important distinction:** An AD computer object in a particular OU does not, by itself, prove that the device is currently online or that a particular GPO has been successfully applied.

### Task 4 — Inspect Group Policy inheritance

Group Policy Objects can be linked to a domain, site, or OU. Policies linked to a parent container may be inherited by objects in child OUs unless inheritance is blocked or other policy settings affect processing.

Inspect the Windows 10 OU:

```powershell
Get-GPInheritance `
    -Target "OU=Windows 10,OU=Computer Accounts,DC=zroots,DC=local"
```

**Recorded result:**

- `GpoInheritanceBlocked`: `No`
- Direct GPO links: `Windows 10 Security Policy`
- Inherited GPO links: `Windows 10 Security Policy` and `Default Domain Policy`

Inspect the Linux OU:

```powershell
Get-GPInheritance `
    -Target "OU=Linux,OU=Computer Accounts,DC=zroots,DC=local"
```

**Recorded result:**

- `GpoInheritanceBlocked`: `No`
- Direct GPO links: None recorded
- Inherited GPO links: `Default Domain Policy`

Inspect the Windows 10 policy object:

```powershell
Get-GPO -Name "Windows 10 Security Policy" |
    Select-Object DisplayName, Id, GpoStatus
```

**Recorded result:**

- Name: `Windows 10 Security Policy`
- GPO ID: `3a184189-759b-420b-883e-9b67a50c78da`
- GPO status: `AllSettingsEnabled`

**Interpretation:** The policy is linked to the Windows 10 OU, and the GPO object reports that its settings are enabled. This does not prove that a client has received or applied the policy. Client-side verification requires checking policy results on the target computer.

For a domain-joined Windows client, use:

```cmd
gpupdate /force
gpresult /r
```

For a detailed HTML report:

```cmd
gpresult /h "%USERPROFILE%\Desktop\gpresult.html"
```

Run these client-side commands only on a suitable target client and review the output to confirm actual policy application.

### Task 5 — Inspect Domain Controllers OU protection

Inspect the Domain Controllers OU:

```powershell
Get-ADOrganizationalUnit `
    -Identity "OU=Domain Controllers,DC=zroots,DC=local" `
    -Properties ProtectedFromAccidentalDeletion |
    Select-Object Name, DistinguishedName, ProtectedFromAccidentalDeletion
```

The recorded initial result showed:

```text
ProtectedFromAccidentalDeletion : False
```

The OU contains the `DC01` computer account, according to the recorded DN:

```text
CN=DC01,OU=Domain Controllers,DC=zroots,DC=local
```

Because this OU is important to domain controller organization, the lab enabled accidental-deletion protection.

Command executed:

```powershell
Set-ADOrganizationalUnit `
    -Identity "OU=Domain Controllers,DC=zroots,DC=local" `
    -ProtectedFromAccidentalDeletion $true
```

Verify the setting:

```powershell
Get-ADOrganizationalUnit `
    -Identity "OU=Domain Controllers,DC=zroots,DC=local" `
    -Properties ProtectedFromAccidentalDeletion |
    Select-Object Name, DistinguishedName, ProtectedFromAccidentalDeletion
```

**Recorded result:** The verification returned `True`.

**Impact:** This changes an Active Directory OU property. It does not require a server restart, and no restart was performed as part of this change.

## 7. Organizational Unit Inventory

The following table records the twelve OUs enumerated during the lab.

| Organizational Unit | Distinguished Name | Accidental-Deletion Protection |
|---|---|---|
| Administrators | `OU=Administrators,OU=User Accounts,DC=zroots,DC=local` | Enabled |
| Computer Accounts | `OU=Computer Accounts,DC=zroots,DC=local` | Enabled |
| Domain Controllers | `OU=Domain Controllers,DC=zroots,DC=local` | Enabled after the lab change |
| Groups | `OU=Groups,DC=zroots,DC=local` | Enabled |
| Linux | `OU=Linux,OU=Computer Accounts,DC=zroots,DC=local` | Enabled |
| Servers | `OU=Servers,OU=Computer Accounts,DC=zroots,DC=local` | Enabled |
| Service Accounts | `OU=Service Accounts,DC=zroots,DC=local` | Enabled |
| Staff | `OU=Staff,OU=User Accounts,DC=zroots,DC=local` | Enabled |
| Students | `OU=Students,OU=User Accounts,DC=zroots,DC=local` | Enabled |
| User Accounts | `OU=User Accounts,DC=zroots,DC=local` | Enabled |
| Windows 10 | `OU=Windows 10,OU=Computer Accounts,DC=zroots,DC=local` | Enabled |
| Windows 11 | `OU=Windows 11,OU=Computer Accounts,DC=zroots,DC=local` | Enabled |

**Note:** The protection values reflect the recorded lab outputs. Re-run the inventory command if you need to confirm the live state after subsequent administrative changes.

## 8. Computer Account Placement

| Computer | Recorded distinguished name | Interpretation |
|---|---|---|
| `WIN10-01` | `CN=WIN10-01,OU=Windows 10,OU=Computer Accounts,DC=zroots,DC=local` | Located in the Windows 10 OU |
| `WINDOWS11` | `CN=WINDOWS11,OU=Windows 11,OU=Computer Accounts,DC=zroots,DC=local` | Located in the Windows 11 OU |
| `DC01` | `CN=DC01,OU=Domain Controllers,DC=zroots,DC=local` | Located in the Domain Controllers OU |

Separating workstation OUs helps administrators apply different policy settings to different device categories. For example, a workstation security GPO may be linked to the Windows 10 OU without linking that same GPO directly to the Windows 11 OU.

Before moving any computer object, confirm the target OU, existing GPO links, administrative permissions, and the possible impact on policy processing.

## 9. Group Policy Inheritance

### 9.1 Recorded policy structure

```text
zroots.local
|
+-- Default Domain Policy
|
+-- Computer Accounts
    |
    +-- Windows 10
    |   +-- Windows 10 Security Policy
    |   +-- Inherits Default Domain Policy
    |
    +-- Linux
        +-- No direct GPO link recorded
        +-- Inherits Default Domain Policy
```

This diagram represents the recorded link and inheritance information for the two inspected OUs. It is not a complete representation of every GPO in the domain.

### 9.2 Key concepts

- **Direct link:** A GPO linked directly to a particular domain, site, or OU.
- **Inheritance:** GPO links from parent containers that flow down to child OUs.
- **Blocked inheritance:** A setting that prevents normal inheritance from parent containers, subject to policy enforcement and processing rules.
- **GPO status:** Indicates whether settings in a GPO are enabled or disabled; it does not prove that a target computer successfully applied them.
- **Resultant policy:** The effective policy for a user or computer after applicable policy-processing rules have been evaluated.

### 9.3 Verification guidance

To validate policy application, inspect a target Windows client:

```cmd
gpresult /r
```

For a detailed report:

```cmd
gpresult /h "%USERPROFILE%\Desktop\gpresult.html"
```

Review the applied GPOs, denied GPOs, scope, and any reported errors. Save the report as evidence if client-side policy verification is part of the lab.

## 10. Accidental Deletion Protection

### 10.1 Purpose

Accidental-deletion protection helps prevent an OU from being deleted inadvertently through common Active Directory administrative tools. It is a safety mechanism, not a substitute for least privilege, backups, or change control.

### 10.2 Verification command

```powershell
Get-ADOrganizationalUnit -Filter * `
    -Properties ProtectedFromAccidentalDeletion |
    Select-Object Name, ProtectedFromAccidentalDeletion
```

### 10.3 Enabling protection

For an individual OU:

```powershell
Set-ADOrganizationalUnit `
    -Identity "OU=Domain Controllers,DC=zroots,DC=local" `
    -ProtectedFromAccidentalDeletion $true
```

### 10.4 Operational considerations

- Verify the OU identity before changing its properties.
- Protect critical OUs from accidental deletion.
- Restrict who can administer the directory.
- Keep regular, tested Active Directory backups.
- Do not disable protection simply to bypass an administrative error.
- If an authorized deletion is required, follow an approved change process and verify the exact target first.

## 11. Verification Results

| Test | Recorded result | Status |
|---|---|---|
| Enumerate existing OUs | Twelve OUs recorded | Passed |
| Create Linux OU | Command executed; OU appears in inventory | Passed |
| Verify Linux OU DN | `OU=Linux,OU=Computer Accounts,DC=zroots,DC=local` | Passed |
| Verify Windows 10 computer location | `WIN10-01` located in Windows 10 OU | Passed |
| Verify Windows 11 computer location | `WINDOWS11` located in Windows 11 OU | Passed |
| Verify domain controller location | `DC01` located in Domain Controllers OU | Passed |
| Inspect Windows 10 GPO inheritance | Direct Windows 10 GPO and inherited Default Domain Policy recorded | Passed |
| Inspect Linux GPO inheritance | No direct GPO link recorded; Default Domain Policy inherited | Passed |
| Inspect Windows 10 GPO object | `AllSettingsEnabled` | Passed |
| Enable Domain Controllers OU protection | Property changed from `False` to `True` | Passed |
| Verify actual GPO application on clients | Client-side `gpresult` output not recorded in this lab | Not verified |

### Result interpretation

The core OU management tasks were completed and verified using the recorded PowerShell output. The lab confirmed the relevant OU structure, computer account locations, selected GPO links, and the protection setting for the Domain Controllers OU.

Client-side application of the Windows 10 security policy remains a separate verification task because a GPO's existence and link do not prove successful application to a client.

## 12. Troubleshooting Notes

### Problem: An OU cannot be found

**Possible causes:**

- The distinguished name is incorrect.
- The OU is located under a different parent.
- The wrong domain or directory server is being queried.
- The account lacks sufficient permissions.

**Diagnostic command:**

```powershell
Get-ADOrganizationalUnit -Filter * |
    Select-Object Name, DistinguishedName
```

Copy the exact distinguished name from the results before using it with `-Identity`.

### Problem: A command reports that the Active Directory cmdlet is unavailable

Check whether the Active Directory module is loaded:

```powershell
Get-Module ActiveDirectory -ListAvailable
```

Import it if available:

```powershell
Import-Module ActiveDirectory
```

On a server where the module is not installed, confirm the required management tools and server features before installing anything.

### Problem: A GPO does not appear to apply

Check the following:

1. Is the computer account in the intended OU?
2. Is the GPO linked to the correct domain, site, or OU?
3. Is the GPO link enabled?
4. Is inheritance blocked or is another policy setting affecting processing?
5. Does the client have connectivity to the domain controller and DNS?
6. Does the client-side `gpresult /r` output list the GPO as applied or denied?

Do not conclude that a GPO is effective solely because `Get-GPO` returns the object.

### Problem: An OU protection value is unexpected

Inspect the exact OU:

```powershell
Get-ADOrganizationalUnit `
    -Identity "OU=Domain Controllers,DC=zroots,DC=local" `
    -Properties ProtectedFromAccidentalDeletion |
    Format-List Name, DistinguishedName, ProtectedFromAccidentalDeletion
```

If the value is not as expected, verify the target identity and authorization before changing it.

## 13. Security Lessons Learned

1. **Use a logical directory structure.** Separate workstation, server, user, group, and service-account objects where appropriate.
2. **Apply least privilege.** Delegate only the administrative rights required for each role.
3. **Protect critical OUs.** Accidental-deletion protection adds a useful safety layer for important directory containers.
4. **Treat GPO links as part of security design.** Link policies to the intended scope and verify their effects on representative clients.
5. **Validate policy application.** A GPO being present and enabled is not proof that its settings reached a computer.
6. **Maintain backups.** OU protection does not replace tested Active Directory recovery procedures.
7. **Document changes.** Record the command, target DN, previous state, new state, and verification output.
8. **Avoid unnecessary disruption.** The OU protection change in this lab did not require a domain controller restart.

## 14. Knowledge Check

Answer the following questions without referring to the earlier sections.

1. What is the primary purpose of an Organizational Unit in Active Directory?
2. What is the difference between an OU and a security group?
3. What does the DN `OU=Linux,OU=Computer Accounts,DC=zroots,DC=local` describe?
4. Why might an administrator separate Windows 10 and Windows 11 computer accounts into different OUs?
5. What does `ProtectedFromAccidentalDeletion = True` help prevent?
6. What is the difference between a direct GPO link and an inherited GPO link?
7. Does `GpoStatus = AllSettingsEnabled` prove that a client applied every setting? Why or why not?
8. Which command can display the GPOs applied to a Windows client?
9. Why is it important to verify an OU's distinguished name before changing its properties?
10. Why should OU deletion protection be used alongside backups and least-privilege administration?

### Answer key

1. To organize directory objects, support delegation, and provide a scope for Group Policy.
2. An OU is a directory container; a security group is commonly used to assign permissions and control access.
3. It identifies the Linux OU beneath the Computer Accounts OU in the `zroots.local` domain.
4. To support different administrative requirements and policy scopes.
5. It helps prevent inadvertent deletion of the protected OU through common administrative operations.
6. A direct link is attached to the container itself; an inherited link comes from a parent container.
7. No. It describes the GPO object's settings status, not successful client-side application.
8. `gpresult /r` or a detailed report created with `gpresult /h`.
9. To avoid changing the wrong directory object.
10. Because protection is only one safeguard and does not replace access controls or recovery capability.

## 15. Lab Conclusion

Lab 03 demonstrated the practical management of Organizational Units in the `zroots.local` Active Directory domain.

The lab recorded twelve OUs, created the `Linux` OU beneath `Computer Accounts`, verified the recorded locations of the Windows 10, Windows 11, and domain controller computer objects, and inspected Group Policy inheritance for the Windows 10 and Linux OUs.

The lab also identified that the `Domain Controllers` OU initially had accidental-deletion protection disabled. Protection was enabled and subsequently verified as `True`.

The main lesson is that a well-organized OU hierarchy supports manageable policy scope and delegated administration, while verification and documentation help prevent accidental changes. The next recommended step is **Lab 04 — User and Group Management**, building on the directory structure established here.

### Completion checklist

- [x] Enumerate and inspect OUs.
- [x] Create the Linux OU under Computer Accounts.
- [x] Inspect computer account locations.
- [x] Review GPO links and inheritance for selected OUs.
- [x] Enable Domain Controllers OU accidental-deletion protection.
- [x] Verify the protection setting.
- [x] Document results and security lessons.
- [ ] Capture and attach final screenshots to the handbook.
- [ ] Save client-side `gpresult` evidence if policy application verification is required.

## 16. Appendix: Useful Commands

### List OUs

```powershell
Get-ADOrganizationalUnit -Filter * |
    Select-Object Name, DistinguishedName
```

### List OU protection settings

```powershell
Get-ADOrganizationalUnit -Filter * `
    -Properties ProtectedFromAccidentalDeletion |
    Select-Object Name, ProtectedFromAccidentalDeletion
```

### Create a protected OU

```powershell
New-ADOrganizationalUnit `
    -Name "Linux" `
    -Path "OU=Computer Accounts,DC=zroots,DC=local" `
    -ProtectedFromAccidentalDeletion $true
```

### Inspect computer objects

```powershell
Get-ADComputer -Filter * `
    -Properties DistinguishedName |
    Select-Object Name, DistinguishedName
```

### Inspect GPO inheritance

```powershell
Get-GPInheritance `
    -Target "OU=Windows 10,OU=Computer Accounts,DC=zroots,DC=local"
```

### Inspect a GPO

```powershell
Get-GPO -Name "Windows 10 Security Policy" |
    Select-Object DisplayName, Id, GpoStatus
```

### Enable OU deletion protection

```powershell
Set-ADOrganizationalUnit `
    -Identity "OU=Domain Controllers,DC=zroots,DC=local" `
    -ProtectedFromAccidentalDeletion $true
```

### Verify policy on a Windows client

```cmd
gpresult /r
gpresult /h "%USERPROFILE%\Desktop\gpresult.html"
```

---

**Documentation:** Private Cyber Lab Handbook  
**Lab:** 03 — Organizational Unit (OU) Management  
**Domain:** `zroots.local`  
**Server:** DC01 — Windows Server 2019  
**Next lab:** Lab 04 — User and Group Management
