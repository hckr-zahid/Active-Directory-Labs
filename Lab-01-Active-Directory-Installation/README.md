# Lab 01 — Active Directory Installation and Configuration

## Overview

This lab focuses on installing and configuring **Active Directory Domain Services (AD DS)** in a Windows Server environment. It demonstrates the initial setup of a Windows domain infrastructure and the promotion of a Windows Server machine to a Domain Controller.

This lab is part of the [Active Directory Labs](../README.md) project, which documents hands-on Windows Server administration, domain management, Group Policy, and security configurations.

## Objectives

- Understand the fundamentals of Active Directory Domain Services.
- Install the AD DS server role.
- Promote Windows Server to a Domain Controller.
- Configure a new Active Directory domain.
- Understand the relationship between a Domain Controller, domain, and DNS.
- Verify the Active Directory installation and domain configuration.

## Technologies and Tools

- Microsoft Windows Server
- Active Directory Domain Services (AD DS)
- DNS Server
- Server Manager
- Active Directory administrative tools
- Virtualization environment, if applicable

## Lab Environment

| Component | Details |
|---|---|
| Operating System | Windows Server |
| Server Role | Domain Controller |
| Directory Service | Active Directory Domain Services |
| DNS | Active Directory-integrated DNS, if configured |
| Domain Name | As configured in the lab |
| Environment | Isolated lab environment |

*Note: Replace generic environment details with your actual server configuration where appropriate.*

## Implementation Steps

### 1. Prepare the Windows Server

- Configure the server's network settings.
- Assign a static IP address where appropriate.
- Set an appropriate computer name.
- Verify network connectivity and DNS settings.

### 2. Install Active Directory Domain Services

- Open Server Manager.
- Select **Add Roles and Features**.
- Choose the Active Directory Domain Services role.
- Install the required management tools and dependencies.

### 3. Promote the Server to a Domain Controller

- Select the notification flag in Server Manager.
- Open the Active Directory Domain Services configuration wizard.
- Select the appropriate deployment option, such as adding a new forest if creating a new domain.
- Configure the domain name and required settings.
- Set the Directory Services Restore Mode (DSRM) password.
- Complete prerequisite checks and promote the server.

### 4. Verify the Installation

- Confirm that the server restarts successfully.
- Sign in using the appropriate domain administrator credentials.
- Open **Active Directory Users and Computers**.
- Verify the domain and default directory containers.
- Check that the required Active Directory and DNS services are running.

## Verification

The installation should be considered successful when the following checks pass:

- [ ] The AD DS role is installed.
- [ ] The server has been promoted to a Domain Controller.
- [ ] The configured domain is accessible.
- [ ] Active Directory administrative tools open successfully.
- [ ] DNS configuration supports domain name resolution.
- [ ] Relevant services and event logs show no unresolved critical installation errors.

*Mark each item only after verifying it in your lab environment.*

## Screenshots and Evidence

Store relevant screenshots in the `screenshots/` directory.

Suggested evidence includes:

1. Server Manager showing the installed AD DS role.
2. The Domain Controller promotion configuration.
3. Active Directory Users and Computers showing the domain.
4. DNS Manager showing the relevant DNS zones.
5. Successful domain login or other appropriate verification.

## Lab Report

For detailed implementation notes, configuration information, and verification evidence, see:

**[View Lab 01 Report (PDF)](Lab-01-Report.pdf)**

## Learning Outcomes

Through this lab, the key learning areas include:

- Understanding Active Directory's role in centralized identity management.
- Installing and configuring AD DS.
- Promoting a Windows Server to a Domain Controller.
- Understanding the importance of DNS in Active Directory.
- Verifying domain infrastructure after installation.

## Security Considerations

- Use strong, unique administrative passwords.
- Keep the lab environment isolated from unauthorized access.
- Avoid including passwords, recovery credentials, or other secrets in screenshots and documentation.
- Apply security updates and follow appropriate Windows Server hardening practices.

## Related Labs

- [Main Repository — Active Directory Labs](../README.md)
- [Lab 02 — Group Policy Management](../Lab-02-Group-Policy-Management/README.md)

---

**Project:** Active Directory Labs  
**Lab:** 01 — Active Directory Installation and Configuration  
**Purpose:** Hands-on Windows Server administration and learning
