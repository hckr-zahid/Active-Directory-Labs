# Lab 04 — User and Group Management in Active Directory

## 1. Lab Information

- **Domain:** `zroots.local`
- **Domain Controller:** `DC01`
- **Operating System:** Windows Server 2019
- **Tools:** Windows PowerShell and the Active Directory PowerShell module
- **Status:** Completed — practical exercises and verification commands executed successfully

## 2. Objectives

- Inspect existing Active Directory user accounts.
- Inspect security groups and their memberships.
- Add a user to a security group.
- Modify user profile attributes.
- Disable and re-enable a test account.
- Inspect the domain password policy and reset a test user's password.
- Verify account state and group membership.

## 3. Environment and Prerequisites

The lab uses the existing private VMware environment and the Active Directory domain `zroots.local`. The commands were executed in an elevated PowerShell session on DC01.

Prerequisites include administrative authorization, the Active Directory PowerShell module, and an existing disposable practice account named `test.user`.

No domain-wide password policy changes were performed during this lab.

## 4. Exercise 1 — Inventory User Accounts

### Command

```powershell
Get-ADUser -Filter * -SearchBase "OU=User Accounts,DC=zroots,DC=local" `
    -Properties Enabled, Department, Title, MemberOf |
    Select-Object Name, SamAccountName, Enabled, Department, Title,
        DistinguishedName |
    Format-List
```

### Actual result

Six accounts were returned:

- `student01` — Student One
- `staff.one` — Staff One
- `admin.one` — Administrator One
- `ali.khan` — Ali Khan
- `ahmed.shah` — Ahmed Shah
- `test.user` — Test User

All six were shown as enabled. Their distinguished names placed them in the expected Students, Staff, Administrators, or User Accounts OU paths.

### Explanation

`Get-ADUser` retrieves user objects. `-SearchBase` limits the search to the specified OU and its descendants. The `Enabled` property indicates whether an account is enabled; it does not prove that a user can successfully log on.

## 5. Exercise 2 — Inspect Security Groups

### Command

```powershell
Get-ADGroup -Filter * -SearchBase "OU=Groups,DC=zroots,DC=local" |
    Select-Object Name, GroupScope, GroupCategory
```

### Actual result

Five Global Security groups were returned:

- `Students`
- `Staff`
- `Administratorss`
- `IT-HelpDesk`
- `Management`

The name `Administratorss` appears to contain an extra “s”. It was documented but not renamed or deleted.

### Explanation

Global security groups can organize users and be assigned permissions. Group scope and category should be selected according to the intended access-control design. Group names alone do not establish what permissions members possess.

## 6. Exercise 3 — Manage Group Membership

### Initial checks

```powershell
Get-ADGroupMember -Identity "Staff" |
    Select-Object Name, SamAccountName, ObjectClass

Get-ADGroupMember -Identity "Students" |
    Select-Object Name, SamAccountName, ObjectClass

Get-ADPrincipalGroupMembership -Identity "ali.khan" |
    Select-Object Name, GroupScope, GroupCategory
```

Initially, `Staff` contained `staff.one`, while `Students` contained `student01`. Ali Khan's memberships included `Domain Users` and `IT-HelpDesk`, but not `Staff`.

### Change performed

```powershell
Add-ADGroupMember -Identity "Staff" -Members "ali.khan"
```

### Verification

```powershell
Get-ADGroupMember -Identity "Staff" |
    Select-Object Name, SamAccountName, ObjectClass

Get-ADPrincipalGroupMembership -Identity "ali.khan" |
    Select-Object Name, GroupScope, GroupCategory
```

### Actual result

The `Staff` group contained:

- Staff One — `staff.one`
- Ali Khan — `ali.khan`

Ali Khan's group memberships subsequently included `Domain Users`, `Staff`, and `IT-HelpDesk`. The Students group continued to contain Student One (`student01`).

### Explanation

`Add-ADGroupMember` adds the specified account to a group. The verification commands confirm membership after the change. Group membership can affect access to resources, so membership should follow the organization's authorization requirements.

## 7. Exercise 4 — Update User Attributes

### Initial state

`test.user` was enabled. Its Department, Title, and Description fields were initially empty.

### Command

```powershell
Set-ADUser -Identity "test.user" `
    -Department "IT" `
    -Title "IT Support Trainee" `
    -Description "Lab 04 practice account"
```

### Verification

```powershell
Get-ADUser -Identity "test.user" -Properties `
    Department, Title, Description |
    Format-List Name, SamAccountName, Department, Title, Description
```

### Actual result

- Department: `IT`
- Title: `IT Support Trainee`
- Description: `Lab 04 practice account`

The changes were returned by the verification query.

### Explanation

`Set-ADUser` updates attributes on an existing user object. These attributes provide administrative context and can support directory searches and reporting. They do not automatically grant permissions.

## 8. Exercise 5 — Disable and Re-enable the Test Account

### Commands

```powershell
Get-ADUser -Identity "test.user" |
    Select-Object Name, SamAccountName, Enabled

Disable-ADAccount -Identity "test.user"

Get-ADUser -Identity "test.user" |
    Select-Object Name, SamAccountName, Enabled

Enable-ADAccount -Identity "test.user"

Get-ADUser -Identity "test.user" |
    Select-Object Name, SamAccountName, Enabled
```

### Actual result

The observed `Enabled` values were:

1. Before disabling: `True`
2. After disabling: `False`
3. After re-enabling: `True`

### Explanation

`Disable-ADAccount` disables the account, preventing normal new logons using that account. `Enable-ADAccount` re-enables it. Disabling an account should not be treated as a guaranteed way to terminate every existing session or revoke every previously issued access token immediately.

Only the disposable practice account was used for this exercise.

## 9. Exercise 6 — Inspect Password Policy

### Command

```powershell
Get-ADDefaultDomainPasswordPolicy -Identity "zroots.local" |
    Format-List MinPasswordLength, ComplexityEnabled,
        PasswordHistoryCount, MaxPasswordAge
```

### Actual result

- Minimum password length: `7`
- Password complexity: `True`
- Password history count: `24`
- Maximum password age: `42.00:00:00` (42 days)

### Explanation

The domain requires passwords to meet its configured length, complexity, and history rules. The displayed values are the domain's default password policy. Fine-grained password policies, if configured, may apply different requirements to particular users or groups.

The minimum length of seven characters is relatively weak by modern security standards. It is documented here as the observed lab configuration, not as a security recommendation. Any policy improvement should be planned and tested separately.

## 10. Exercise 7 — Reset the Test User's Password

### Command

```powershell
$newPassword = Read-Host "Enter a new lab-only password" -AsSecureString

Set-ADAccountPassword -Identity "test.user" -Reset -NewPassword $newPassword

Remove-Variable newPassword
```

### Actual result

The first attempt failed with an Active Directory password policy error indicating that the supplied password did not meet length, complexity, or history requirements.

A subsequent attempt completed without an error. The variable was removed, and the account was queried again.

### Verification command

```powershell
Get-ADUser -Identity "test.user" |
    Select-Object Name, SamAccountName, Enabled
```

### Verified result

- Name: `Test User`
- SamAccountName: `test.user`
- Enabled: `True`

The successful command execution supports recording the password reset as successful. An interactive logon using the new password was not demonstrated in the collected evidence.

The password itself was not recorded in this report.

## 11. Final Group Membership Verification

### Commands

```powershell
Get-ADGroupMember -Identity "Staff" |
    Select-Object Name, SamAccountName, ObjectClass

Get-ADGroupMember -Identity "Students" |
    Select-Object Name, SamAccountName, ObjectClass

Get-ADPrincipalGroupMembership -Identity "test.user" |
    Select-Object Name, GroupScope, GroupCategory
```

### Actual result

- Staff: `staff.one` and `ali.khan`
- Students: `student01`
- Test User: `Domain Users`

These results matched the expected membership state after the exercises.

## 12. Troubleshooting Notes

### Password reset rejected

The domain rejected the initial password because it did not meet one or more configured requirements. The domain policy was inspected, and a subsequent reset succeeded. Passwords should be entered interactively and never included in lab reports.

### PowerShell prompt and output pasted as commands

Some errors occurred when prompt text (`PS C:\Users\Administrator>`), continuation markers (`>>`), or displayed output was entered as command text. In PowerShell, enter only the command itself. Do not copy the prompt, output table, or error details back into the console.

### Safe recovery practice

When a command produces an error, stop, read the error, and run a fresh verification command. Avoid repeating state-changing commands blindly, particularly on production accounts or domain-wide settings.

## 13. Security Lessons

1. Follow least privilege when assigning users to groups.
2. Verify group membership after making a change.
3. Use dedicated test accounts for account-management exercises.
4. Disable accounts when access should be suspended, while separately managing active sessions and other credentials as needed.
5. Do not store plaintext passwords in scripts, screenshots, source control, or reports.
6. Treat password complexity, length, history, and expiration as policy settings that should be reviewed against current security guidance.
7. Record actual output and distinguish successful command execution from end-to-end logon verification.
8. Avoid renaming or deleting existing directory objects merely to correct a suspected naming error without checking dependencies.

## 14. Knowledge Check

1. What is the difference between a user account being enabled and a user successfully logging on?
2. Which command adds a user to an existing AD group?
3. Why should group membership be verified after an administrative change?
4. What does `-SearchBase` do in `Get-ADUser`?
5. Why should a password not be included in a lab report?
6. Does a successful password-reset command prove that an interactive logon works?

## 15. Lab Conclusion

The collected PowerShell evidence confirms successful user inventory, security-group inspection, group membership modification, user attribute updates, account disable/enable testing, password policy inspection, and a subsequent successful password-reset command for `test.user`.

The domain password policy was not modified. The password was not disclosed or recorded. The interactive logon was not tested as part of the evidence collected.

**Lab status: Completed for the documented exercises.**

## 16. Evidence Checklist

- [x] User account inventory output
- [x] Security group inventory output
- [x] Staff membership before and after adding Ali Khan
- [x] Students group membership output
- [x] Test User attribute update and verification
- [x] Account disable and re-enable output
- [x] Domain password policy output
- [x] Password reset success indicated by no error
- [x] Final account enabled-state output
- [x] Final group membership output
- [ ] Optional screenshot of the final PowerShell verification
- [ ] Optional interactive logon test using the new test-account password
