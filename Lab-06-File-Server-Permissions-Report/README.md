
# Lab 06: File Server and Permissions

**Windows Server 2019 File Services and Active Directory Access Control**

**Project:** Private Cybersecurity Lab  
**Domain:** `zroots.local`  
**Domain Controller:** `DC01`  
**Lab Status:** Functionally Completed

---

## Table of Contents

1. Executive Summary
2. Lab Objectives and Scope
3. Lab Environment and Endpoint Addressing
4. Network Topology
5. Directory Structure and File Server Preparation
6. Active Directory Security Groups and User Assignments
7. SMB Share Configuration
8. NTFS Permissions and Access Control
9. Functional Access Testing and Validation Results
10. Troubleshooting and Problem Resolution
11. Security Lessons and Hardening Recommendations
12. Knowledge Check and Answers
13. Appendix: Important Commands
14. Lab Completion Summary

---

## 1. Executive Summary

This technical report documents the implementation and validation of file sharing and access control on Windows Server 2019 within a private Active Directory environment.

The lab uses the `zroots.local` domain, with `DC01` providing centralized identity management and hosting a dedicated SMB network share. Active Directory security groups were used to assign different levels of file access to domain users.

The lab included creating a dedicated directory structure, configuring the `Lab06Data` SMB share, applying NTFS permissions, and validating access from a domain-joined Windows 10 client.

Three role-based access scenarios were tested:

- `student01`: Read access was permitted, while write access was denied.
- `staff.one`: File modification was permitted.
- `ahmed.shah`: Management full access was verified after correcting an NTFS permission issue.

During troubleshooting, the Management group had Full access at the SMB share level but lacked an explicit NTFS Allow rule. This mismatch caused an access-denied error. Adding the Management permission to the dedicated Lab06 directory resolved the issue.

The recorded tests passed, demonstrating the practical application of centralized access control, least privilege, and defense-in-depth principles for Windows file services.

## 2. Lab Objectives and Scope

The objectives of this lab were to:

- Create a dedicated directory structure for file-sharing exercises.
- Create and use Active Directory security groups for role-based access.
- Configure an SMB network share on Windows Server 2019.
- Apply NTFS permissions to control access to files and folders.
- Understand the difference between SMB share permissions and NTFS permissions.
- Validate read, modify, and full-access operations using separate domain accounts.
- Confirm that read-only users cannot modify protected files.
- Troubleshoot an access-denied error caused by missing NTFS permissions.
- Record actual results, evidence requirements, and security lessons.

The lab was limited to a dedicated directory created for this exercise. Existing domain services and unrelated file shares were not intended to be modified.

## 3. Lab Environment and Endpoint Addressing

The lab operates in VMware using a private host-only network named `MyLabNet`. Windows Server 2019 provides Active Directory Domain Services, DNS, and RRAS/NAT.

| Endpoint | IPv4 Address | Configuration / Role |
|---|---|---|
| DC01 | `10.1.1.10` | Windows Server 2019; AD DS, DNS, RRAS/NAT, and Lab06 file share |
| DC01 WAN | `192.168.0.144` | Bridged interface |
| WIN10-01 | `10.1.1.20` | Domain-joined Windows 10 client |
| Kali Linux | `10.1.1.30` | Security-testing endpoint |
| BlackArch | `10.1.1.40` | Security-testing endpoint |
| Windows 11 | `10.1.1.50` | Domain-joined Windows 11 client |
| Metasploitable | `10.1.1.60` | Vulnerable lab target |

### Domain and file-sharing parameters

| Parameter | Value |
|---|---|
| Active Directory domain | `zroots.local` |
| NetBIOS domain | `ZROOTS` |
| Domain controller | `DC01` |
| Private subnet | `10.1.1.0/24` |
| File server address | `10.1.1.10` |
| SMB share name | `Lab06Data` |
| Network share path | `\\10.1.1.10\Lab06Data` |
| Local directory | `C:\Lab06-FileServer\CompanyData` |
| Test file | `Lab06-Test.txt` |

## 4. Network Topology

The file server operates within the existing private lab network. Domain users access the shared files from the Windows 10 client through SMB over the internal network.

**Network path:**

```text
Internet
   |
Physical Router (192.168.0.1)
   |
VMware VMnet0 — Bridged
   |
DC01 — Windows Server 2019
   |-- WAN: 192.168.0.144
   |-- LAN: 10.1.1.10/24
   |-- Active Directory Domain Services
   |-- DNS
   |-- RRAS/NAT
   |-- SMB File Share: Lab06Data
   |
MyLabNet — VMware Host-only Network
10.1.1.0/24
   |
   |-- WIN10-01       10.1.1.20
   |-- Kali Linux     10.1.1.30
   |-- BlackArch      10.1.1.40
   |-- Windows 11     10.1.1.50
   |-- Metasploitable 10.1.1.60
````

The SMB share is hosted on DC01 and accessed using the server's internal IP address. Authentication is handled through the existing Active Directory domain.

Design consideration: This lab uses a dedicated directory to isolate file-permission experiments from other server resources. Changes to the Lab06 directory should not affect unrelated shares.

## 5. Directory Structure and File Server Preparation

A dedicated directory was used to keep Lab06 resources separate from existing file-sharing configurations.

### 5.1 Directory structure

```
C:\Lab06-FileServer\
    CompanyData\
        IT\
        HR\
        Lab06-Test.txt
```

The primary share directory was:

```
C:\Lab06-FileServer\CompanyData
```

The test file was:

```
C:\Lab06-FileServer\CompanyData\Lab06-Test.txt
```

The initial test-file content was:

```
Lab 06 file access verification.
```

The `IT` and `HR` directories were created as subdirectories of the dedicated Lab06 directory. Their existence does not mean that separate SMB shares were created for them.

An additional directory, `C:\Users\Administrator\Lab06-FileServer`, was accidentally created during the lab. It was left untouched and was not used as the share path.

### 5.2 Verifying the directory

The directory was checked using PowerShell:

```
Test-Path "C:\Lab06-FileServer"
Test-Path "C:\Lab06-FileServer\CompanyData"
```

The primary Lab06 directory and CompanyData path were confirmed to exist.

### 5.3 Purpose of the test file

The test file provides a simple way to verify whether a domain account can read or modify shared content.

Using one file for all three access tests makes it possible to compare the actual results without creating unnecessary test data.

## 6. Active Directory Security Groups and User Assignments

Active Directory security groups were used to manage access by role rather than assigning separate permissions to each user.

### 6.1 Security groups

The following groups were configured for the lab:

| Security group             | Intended access  |
| -------------------------- | ---------------- |
| `ZROOTS\Lab06-File-Read`   | Read-only access |
| `ZROOTS\Lab06-File-Modify` | Modify access    |
| `ZROOTS\Management`        | Full access      |

### 6.2 User assignments

| Domain account | Group membership    | Intended role                    |
| -------------- | ------------------- | -------------------------------- |
| `student01`    | `Lab06-File-Read`   | Read-only user                   |
| `staff.one`    | `Lab06-File-Modify` | File modification user           |
| `ahmed.shah`   | `Management`        | Management user with full access |

The Ahmed Shah account was verified as enabled and unlocked during troubleshooting.

### 6.3 Why security groups are important

Security groups make access management more consistent and easier to maintain.

For example, if another employee needs modify access, an administrator can add the employee to `Lab06-File-Modify` rather than creating a new set of individual ACL entries.

This approach supports:

- Centralized administration.
- Role-based access control.
- Easier account onboarding and offboarding.
- Consistent permission assignment.
- Reduced risk of inconsistent individual permissions.

## 7. SMB Share Configuration

SMB allows users to access files and folders over a network using a share name.

The share created for this lab was named `Lab06Data`.

### 7.1 Creating the SMB share

The following command was executed on DC01 in PowerShell:

```
New-SmbShare -Name "Lab06Data" `
    -Path "C:\Lab06-FileServer\CompanyData" `
    -FullAccess "ZROOTS\Management" `
    -ChangeAccess "ZROOTS\Lab06-File-Modify" `
    -ReadAccess "ZROOTS\Lab06-File-Read"
```

### 7.2 Share permission assignments

| Principal                  | SMB share permission | Purpose                        |
| -------------------------- | -------------------- | ------------------------------ |
| `ZROOTS\Management`        | Full                 | Full access at the share layer |
| `ZROOTS\Lab06-File-Modify` | Change               | Read and modify shared content |
| `ZROOTS\Lab06-File-Read`   | Read                 | Read shared content            |

The share was accessed through:

```
\\10.1.1.10\Lab06Data
```

### 7.3 Verifying share permissions

The configured share permissions were inspected using:

```
Get-SmbShareAccess -Name "Lab06Data"
```

The recorded output showed the expected principals and access levels:

- `ZROOTS\Management` — Full.
- `ZROOTS\Lab06-File-Modify` — Change.
- `ZROOTS\Lab06-File-Read` — Read.

Result: The SMB share was created, and its share-level permissions were verified.

## 8. NTFS Permissions and Access Control

SMB share permissions control access through the network share. NTFS permissions control access to the underlying files and directories on the Windows file system.

For remote SMB access, both permission layers must permit the requested operation. A user who has Full access at the share level can still receive Access is denied if the NTFS ACL does not allow the operation.

### 8.1 Intended NTFS permissions

The Lab06 root directory used the following permission design:

| Principal                  | NTFS permission | Purpose                     |
| -------------------------- | --------------- | --------------------------- |
| `NT AUTHORITY\SYSTEM`      | Full Control    | System operations           |
| `BUILTIN\Administrators`   | Full Control    | Local server administration |
| `ZROOTS\Lab06-File-Read`   | ReadAndExecute  | Read-only access            |
| `ZROOTS\Lab06-File-Modify` | Modify          | Read and modify access      |
| `ZROOTS\Management`        | FullControl     | Full access for Management  |

The group permissions were configured to inherit to child directories and files.

### 8.2 Permission inheritance

The NTFS rules for the Lab06 root were intended to apply to child files and folders through inheritance.

Inheritance helps maintain consistent access control across the directory tree. Without a deliberate inheritance design, a child folder can have permissions that differ from those of its parent.

### 8.3 The Management access-denied issue

During testing, `ahmed.shah` received Access is denied while attempting to access the test file.

The investigation identified a mismatch:

- The SMB share granted `ZROOTS\Management` Full access.
- The NTFS ACL did not initially contain an Allow rule granting Management access.

Therefore, the SMB permission alone was insufficient.

### 8.4 Correcting the NTFS ACL

The following targeted correction was executed on DC01 in elevated PowerShell:

```
$path = "C:\Lab06-FileServer\CompanyData"

$acl = Get-Acl -Path $path

$rule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "ZROOTS\Management",
    [System.Security.AccessControl.FileSystemRights]::FullControl,
    [System.Security.AccessControl.InheritanceFlags]"ContainerInherit, ObjectInherit",
    [System.Security.AccessControl.PropagationFlags]::None,
    [System.Security.AccessControl.AccessControlType]::Allow
)

$acl.AddAccessRule($rule)

Set-Acl -Path $path -AclObject $acl
```

The command added a Management Allow rule to the dedicated Lab06 directory, with inheritance for child folders and files.

It was targeted at the Lab06 directory rather than the entire drive or unrelated file shares.

### 8.5 Verifying the NTFS ACL

The ACL was inspected using:

```
(Get-Acl "C:\Lab06-FileServer\CompanyData").Access |
    Format-Table IdentityReference, FileSystemRights,
        AccessControlType, IsInherited -AutoSize
```

The expected design includes the system, administrators, read group, modify group, and Management group.

The subsequent successful Management write and read tests confirmed that the correction resolved the access problem for the tested file.

Important scope restriction: The existing `C:\CompanyData` directory and its pre-existing `IT` share are separate from Lab06. Their permissions must not be changed as part of this exercise.

## 9. Functional Access Testing and Validation Results

Functional tests were conducted from the domain-joined Windows 10 client, WIN10-01, using the intended domain accounts.

### 9.1 Test A — Read access as student01

The account `ZROOTS\student01` was used to verify read-only access.

Commands:

```
whoami

Get-Content "\\10.1.1.10\Lab06Data\Lab06-Test.txt"
```

Recorded results:

- The account identity was verified.
- The shared test file was opened successfully.
- The file content was displayed.

Result: PASS — read access was permitted.

### 9.2 Test B — Modify access as staff.one

The account `ZROOTS\staff.one` was used to test modification permissions.

Commands:

```
whoami

Add-Content "\\10.1.1.10\Lab06Data\Lab06-Test.txt" `
    "Modify permission test by staff.one."

Get-Content "\\10.1.1.10\Lab06Data\Lab06-Test.txt"
```

Recorded results:

- The append operation succeeded.
- The updated file could be read.
- The staff test line appeared twice because the append command was executed twice.

Result: PASS — modification access was permitted.

### 9.3 Test C — Enforcing read-only restrictions

The account `ZROOTS\student01` attempted to append content to the same file.

Command:

```
Add-Content "\\10.1.1.10\Lab06Data\Lab06-Test.txt" `
    "This should be denied for student01."
```

Recorded result:

Windows returned an Access is denied error.

This was the expected result because the account was intended to have read-only access.

Result: PASS — the write restriction was successfully enforced.

A denied write is not a failed security test when the account is supposed to be read-only. It demonstrates that the permission boundary is working as intended.

### 9.4 Test D — Management full access as ahmed.shah

The account `ZROOTS\ahmed.shah` was used to validate Management access after the NTFS correction.

Commands:

```
whoami

Add-Content "\\10.1.1.10\Lab06Data\Lab06-Test.txt" `
    "Full access test by ahmed.shah."

Get-Content "\\10.1.1.10\Lab06Data\Lab06-Test.txt"
```

Recorded results:

- `whoami` displayed `zroots\ahmed.shah`.
- The append operation succeeded after the NTFS Allow rule was added.
- The file content was displayed successfully.

Result: PASS — the tested Management account could read and modify the file.

### 9.5 Test-results summary

| Test                  | Account      | Expected behavior        | Recorded result                            | Status |
| --------------------- | ------------ | ------------------------ | ------------------------------------------ | ------ |
| Read test             | `student01`  | Read permitted           | File read successfully                     | PASS   |
| Modify test           | `staff.one`  | Read and write permitted | Append and read succeeded                  | PASS   |
| Read-only enforcement | `student01`  | Write denied             | Access is denied                           | PASS   |
| Management access     | `ahmed.shah` | Read and write permitted | Append and read succeeded after correction | PASS   |

### 9.6 Final observed test-file content

The final recorded content was:

```
Lab 06 file access verification.
Modify permission test by staff.one.
Modify permission test by staff.one.
Full access test by ahmed.shah.
```

The duplicate `staff.one` line is expected because the modification command was executed twice.

No `student01` write-test text appeared in the recorded final content because that write attempt was denied.

## 10. Troubleshooting and Problem Resolution

This section documents the actual permission issue encountered during Lab06 and provides diagnostic guidance for future file-sharing problems.

### 10.1 Access is denied despite SMB Full access

Observed symptom

The Management user could not access the test file even though the SMB share granted Management Full access.

Root cause

The NTFS ACL did not initially include an Allow rule for `ZROOTS\Management`.

Correction

A FullControl NTFS Allow rule was added to:

```
C:\Lab06-FileServer\CompanyData
```

The rule was configured to inherit to child files and folders.

Verification

The test was repeated as `zroots\ahmed.shah`. Both the append and read operations succeeded.

Outcome: Resolved.

### 10.2 Read-only account cannot append content

Observed symptom

The `student01` account received Access is denied when attempting to append text.

Interpretation

The account was assigned the read-only role. The denied write operation was therefore expected.

Outcome: PASS — the intended security restriction worked.

### 10.3 Modify permission test produces duplicate lines

Observed symptom

The staff modification line appeared twice in the test file.

Cause

The append command was executed twice during testing.

Interpretation

`Add-Content` appends new text and does not replace existing file content. Running the same command twice creates two copies of the line.

Outcome: Expected behavior; no permission correction was necessary.

### 10.4 Administrator access does not match role-based testing

Troubleshooting consideration

An administrator account may not have the intended access through a restricted SMB share simply because it is an administrator. Effective remote access depends on the permissions granted by the share and the NTFS ACL, along with applicable security-token and policy behavior.

For reliable role testing, use the intended test accounts and verify their identities with `whoami`.

Do not broaden permissions indiscriminately to make an administrative test succeed.

### 10.5 Password forgotten during account testing

The `ahmed.shah` account password was reset during troubleshooting. The account was verified as enabled and unlocked.

Passwords must not be recorded in screenshots, reports, or public repositories. Use a secure administrative procedure to reset a forgotten password, then verify the account status and sign-in behavior.

### 10.6 Recommended diagnostic commands

Inspect share permissions:

```
Get-SmbShareAccess -Name "Lab06Data"
```

Inspect NTFS permissions:

```
(Get-Acl "C:\Lab06-FileServer\CompanyData").Access |
    Format-Table IdentityReference, FileSystemRights,
        AccessControlType, IsInherited -AutoSize
```

Verify Active Directory group membership:

```
Get-ADGroupMember -Identity "Lab06-File-Read"

Get-ADGroupMember -Identity "Lab06-File-Modify"

Get-ADGroupMember -Identity "Management"
```

Verify the test account:

```
Get-ADUser -Identity "ahmed.shah" `
    -Properties Enabled, LockedOut |
    Format-List SamAccountName, Enabled, LockedOut
```

Verify the test file:

```
Test-Path "C:\Lab06-FileServer\CompanyData\Lab06-Test.txt"
```

Troubleshooting conclusion: The actual Management access-denied issue was resolved by correcting the NTFS permissions on the dedicated Lab06 directory. All four functional access tests then passed.

## 11. Security Lessons and Hardening Recommendations

### 11.1 Apply the principle of least privilege

Users should receive only the permissions required for their responsibilities.

The read-only account should not be able to modify shared files. Staff members who need to edit files should receive Modify rather than unrestricted Full Control.

### 11.2 Understand both permission layers

SMB share permissions and NTFS permissions work together for remote file access.

A share-level Full permission does not automatically override the NTFS ACL. Administrators must verify both layers when troubleshooting access issues.

### 11.3 Prefer role-based security groups

Assigning permissions to groups makes access management easier and more consistent.

When a user's responsibilities change, group membership can be updated without redesigning the entire folder ACL.

### 11.4 Test with actual user accounts

Testing with the intended role identities provides stronger evidence than testing only with an administrator account.

Use `whoami` to verify which account is performing each operation.

### 11.5 Preserve unrelated resources

The Lab06 directory is separate from the existing `C:\CompanyData` directory and pre-existing `IT` share.

Permission experiments should remain confined to the designated lab path. This reduces the risk of disrupting existing services or changing permissions outside the scope of the exercise.

### 11.6 Review inheritance and explicit permissions

Inheritance simplifies ACL management but should be checked carefully.

Unexpected explicit Deny entries, disabled inheritance, or child-specific ACLs can change effective access. Review the actual ACL before making corrective changes.

### 11.7 Collect reliable evidence

For each access test, record:

- The account identity.
- The command executed.
- The actual output.
- Whether the result matched the expected permission.
- Any troubleshooting change applied.

Do not publish passwords or other sensitive authentication data in screenshots.

### 11.8 Use dedicated test resources

A dedicated test directory and test file allow access-control experiments without modifying unrelated company or system data.

This is a safer approach for repeated practice in a private cybersecurity lab.

## 12. Knowledge Check and Answers

Q1. What is the difference between SMB share permissions and NTFS permissions?

SMB permissions control access through a network share. NTFS permissions control access to files and directories on the underlying Windows file system. Both affect remote SMB access.

Q2. Why did ahmed.shah receive Access is denied initially?

The SMB share granted Management Full access, but the NTFS ACL did not initially include an Allow rule for `ZROOTS\Management`.

Q3. Which group provides read-only access in this lab?

`ZROOTS\Lab06-File-Read`.

Q4. Why did student01 receive Access is denied when using Add-Content?

The account was intended to have read-only access. Denying the write attempt confirmed that the restriction was enforced.

Q5. Why are security groups preferable to individual ACL entries for every user?

Groups simplify administration, support consistent role-based access, and make it easier to manage changing user responsibilities.

Q6. Why should administrators verify both the share and NTFS ACLs?

Because the requested operation must be permitted by the applicable permission layers. A permissive share ACL alone is insufficient if NTFS permissions do not allow access.

Q7. Why was a dedicated Lab06 directory used?

It isolates the exercise and reduces the risk of changing unrelated files, folders, or existing SMB shares.

Q8. What does the duplicate staff.one line demonstrate?

The `Add-Content` command was executed twice. It appends content rather than replacing existing content.

## 13. Appendix: Important Commands

### 13.1 Verify the directory and test file

```
Test-Path "C:\Lab06-FileServer"

Test-Path "C:\Lab06-FileServer\CompanyData"

Test-Path "C:\Lab06-FileServer\CompanyData\Lab06-Test.txt"
```

### 13.2 Create the SMB share

```
New-SmbShare -Name "Lab06Data" `
    -Path "C:\Lab06-FileServer\CompanyData" `
    -FullAccess "ZROOTS\Management" `
    -ChangeAccess "ZROOTS\Lab06-File-Modify" `
    -ReadAccess "ZROOTS\Lab06-File-Read"
```

This command documents the original share-creation configuration. Do not rerun it blindly if the share already exists.

### 13.3 Verify SMB share permissions

```
Get-SmbShareAccess -Name "Lab06Data"
```

### 13.4 Inspect NTFS permissions

```
(Get-Acl "C:\Lab06-FileServer\CompanyData").Access |
    Format-Table IdentityReference, FileSystemRights,
        AccessControlType, IsInherited -AutoSize
```

### 13.5 Verify Active Directory group membership

```
Get-ADGroupMember -Identity "Lab06-File-Read"

Get-ADGroupMember -Identity "Lab06-File-Modify"

Get-ADGroupMember -Identity "Management"
```

### 13.6 Test read access

Run from a client signed in as the read-only user:

```
whoami

Get-Content "\\10.1.1.10\Lab06Data\Lab06-Test.txt"
```

### 13.7 Test modify access

Run from a client signed in as the modify user:

```
whoami

Add-Content "\\10.1.1.10\Lab06Data\Lab06-Test.txt" `
    "Modify permission test by staff.one."

Get-Content "\\10.1.1.10\Lab06Data\Lab06-Test.txt"
```

### 13.8 Test read-only enforcement

Run from a client signed in as the read-only user:

```
Add-Content "\\10.1.1.10\Lab06Data\Lab06-Test.txt" `
    "This should be denied for student01."
```

The expected result is Access is denied.

### 13.9 Test Management access

Run from a client signed in as the Management user:

```
whoami

Add-Content "\\10.1.1.10\Lab06Data\Lab06-Test.txt" `
    "Full access test by ahmed.shah."

Get-Content "\\10.1.1.10\Lab06Data\Lab06-Test.txt"
```

### 13.10 Verify the Management account

```
Get-ADUser -Identity "ahmed.shah" `
    -Properties Enabled, LockedOut |
    Format-List SamAccountName, Enabled, LockedOut
```

Never place account passwords in scripts, screenshots, or report files.

## 14. Lab Completion Summary

| Lab component                  | Recorded result                  |
| ------------------------------ | -------------------------------- |
| Dedicated Lab06 directory      | Created and verified             |
| Active Directory access groups | Configured                       |
| SMB share `Lab06Data`          | Created and verified             |
| Share-level permissions        | Verified                         |
| NTFS permissions               | Configured and corrected         |
| `student01` read test          | Successful                       |
| `student01` write restriction  | Successfully enforced            |
| `staff.one` modification test  | Successful                       |
| `ahmed.shah` Management test   | Successful after NTFS correction |
| Final test-file content        | Verified                         |

### Final Assessment

Overall status: Lab 06 functionally completed.

All four functional access tests produced the intended results:

- `student01` could read the file but could not write to it.
- `staff.one` could append content to the file.
- `ahmed.shah` could append and read content after the Management NTFS permission was corrected.

The lab demonstrated how Active Directory security groups, SMB share permissions, and NTFS ACLs work together to provide role-based access control.

### Evidence Checklist

- Screenshot of the Lab06 directory structure.
- Screenshot of `Get-SmbShareAccess -Name "Lab06Data"`.
- Screenshot of the final NTFS ACL.
- Screenshot of successful read access as `student01`.
- Screenshot of denied write access as `student01`.
- Screenshot of successful modification as `staff.one`.
- Screenshot of successful read and write access as `ahmed.shah`.
- Screenshot showing the final test-file content.

Only mark evidence items complete after the corresponding screenshots have actually been captured.

### Next Lab

Lab 07 — Account Lockout and Password Policies, following the established lab sequence.

Private Cybersecurity Lab Handbook — Lab 06 Report

Documentation note: This report reflects the recorded lab configuration and test results. Repeat the verification commands when performing a current-state audit. Do not include passwords or other authentication secrets in public documentation.

```
```
