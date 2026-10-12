
# Lab 02 : Group Policy Management (GPO)

## Overview

This lab focuses on configuring, managing, and verifying **Group Policy Objects (GPOs)** in a Windows Server Active Directory environment.

Group Policy enables administrators to centrally manage user and computer settings across a domain. This lab demonstrates the fundamentals of Group Policy Management, policy configuration, and verification of policy application.

This lab is part of the [Active Directory Labs](../README.md) project.

## Objectives

- Understand the purpose of Group Policy in Active Directory.
- Explore the Group Policy Management Console (GPMC).
- Create and configure Group Policy Objects.
- Link Group Policy Objects to the appropriate domain or Organizational Unit.
- Understand policy scope and inheritance.
- Verify policy application using Group Policy tools.
- Review policy results and troubleshoot common issues.

## Technologies and Tools

- Microsoft Windows Server
- Active Directory Domain Services (AD DS)
- Group Policy Management Console (GPMC)
- Windows domain client
- Group Policy Results (`gpresult`)
- Group Policy Results Wizard
- Active Directory Users and Computers

## Lab Environment

| Component | Details |
|---|---|
| Directory Service | Active Directory Domain Services |
| Domain | `ZROOTS.local` or the actual configured domain |
| Domain Controller | Windows Server |
| Policy Management | Group Policy Management Console |
| Test Account | Domain user account |
| Client System | Domain-joined Windows client, if applicable |

*Update the domain name and environment details if your actual configuration differs.*

## Implementation Steps

### 1. Open Group Policy Management

Open Server Manager and launch **Group Policy Management**. Review the forest, domain, Organizational Units, and existing Group Policy Objects.

### 2. Create or Configure a GPO

Create a new Group Policy Object or edit an existing one. Configure the required user or computer policy settings according to the lab objectives.

### 3. Link the GPO

Link the Group Policy Object to the appropriate domain or Organizational Unit. Confirm that the target users or computers are within the intended scope.

### 4. Apply and Refresh Policies

On the target Windows client, refresh Group Policy using:

```cmd
gpupdate /force
```

Allow the update to complete. Some settings may require the user to sign out or the computer to restart.

### 5. Verify Policy Application

Use the appropriate verification method:

```cmd
gpresult /r
```

For a detailed HTML report, run:

```cmd
gpresult /h "%USERPROFILE%\Desktop\gpresult.html"
```

Review the results to determine which policies were applied and whether the expected GPO appears in the report.

For computer-scope results, run Command Prompt as an administrator if elevated permissions are required:

```cmd
gpresult /r /scope computer
```

### 6. Review Group Policy Results

Use the Group Policy Results Wizard in GPMC to inspect the applied policies for the relevant user and computer. Investigate missing policies, filtering, permissions, and other reported issues as necessary.

## Verification Checklist

- [ ] Group Policy Management Console opens successfully.
- [ ] The intended GPO exists and is configured.
- [ ] The GPO is linked to the correct domain or Organizational Unit.
- [ ] Group Policy refresh completes successfully.
- [ ] Applied policies are reviewed using `gpresult` or GPMC.
- [ ] Expected policy behavior is verified on the target system.

*Mark each item as complete only after confirming the result in your lab environment.*

## Troubleshooting Notes

If a policy does not appear to apply, check:

- Whether the GPO is linked to the correct domain or Organizational Unit.
- Whether the target user or computer is in scope.
- Security filtering and relevant permissions.
- Whether the policy is enabled and the client can contact a Domain Controller.
- Group Policy refresh status and relevant event logs.
- Whether the selected setting requires sign-out or restart.

## Lab Report

The detailed lab report documents the configuration process and verification evidence.

**[View Lab 02 Report (PDF)](Lab-02-Report.pdf)**

## Learning Outcomes

After completing this lab, the key learning areas include:

- Centralized policy management in Active Directory.
- Creating, editing, and linking GPOs.
- Understanding policy scope and inheritance.
- Verifying policy application with Windows administration tools.
- Troubleshooting Group Policy processing issues.

## Security Considerations

- Apply policies according to the principle of least privilege.
- Test new GPOs in an appropriate lab scope before wider deployment.
- Avoid unnecessarily restrictive policies that could disrupt administration.
- Document configuration changes and verification results.

## Related Labs

- [Main Repository — Active Directory Labs](../README.md)
- [Lab 01 — Active Directory Installation](../Lab-01-Active-Directory-Installation/README.md)

---

**Project:** Active Directory Labs  
**Lab Number:** 02  
**Topic:** Group Policy Management  
**Platform:** Windows Server and Active Directory  
**Status:** Completed lab documentation
