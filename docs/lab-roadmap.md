# Active Directory Labs — Roadmap

## Overview

This roadmap outlines the learning path for the Active Directory Labs project. It organizes practical exercises from initial Windows Server and Active Directory configuration to Group Policy administration, infrastructure services, and security hardening.

The goal is to build a structured, hands-on portfolio demonstrating Windows Server administration and Active Directory management skills.

## Lab Progress

| Lab | Topic | Status |
|---|---|---|
| Lab 01 | Active Directory Installation and Configuration | Completed |
| Lab 02 | Group Policy Management (GPO) | Completed |
| Lab 03 | Organizational Unit (OU) Management | Planned |
| Lab 04 | User and Group Management | Planned |
| Lab 05 | DNS Configuration and Troubleshooting | Planned |
| Lab 06 | DHCP Server Configuration | Planned |
| Lab 07 | File Server and NTFS Permissions | Planned |
| Lab 08 | Password and Account Lockout Policies | Planned |
| Lab 09 | Group Policy Troubleshooting | Planned |
| Lab 10 | Active Directory Security and Hardening | Planned |
| Lab 11 | Active Directory Backup and Recovery | Planned |
| Lab 12 | Active Directory Replication and Sites | Planned |

*Statuses will be updated as each lab is implemented, tested, and documented.*

## Phase 1 — Active Directory Fundamentals

### Lab 01: Active Directory Installation and Configuration
**Status:** Completed

Objectives:
- Install Active Directory Domain Services.
- Promote Windows Server to a Domain Controller.
- Configure the domain environment.
- Verify Active Directory and DNS functionality.

### Lab 02: Group Policy Management
**Status:** Completed

Objectives:
- Explore Group Policy Management Console.
- Create and configure Group Policy Objects.
- Link GPOs to the appropriate scope.
- Verify policy application using Group Policy tools.

## Phase 2 — Directory and Identity Management

### Lab 03: Organizational Unit Management
**Status:** Planned

Objectives:
- Create and organize Organizational Units.
- Understand OU hierarchy and delegation.
- Link policies to specific OUs.
- Verify the scope of delegated administration.

### Lab 04: User and Group Management
**Status:** Planned

Objectives:
- Create and manage domain user accounts.
- Create security groups.
- Assign users to appropriate groups.
- Understand group-based access management.
- Practice account lifecycle administration.

## Phase 3 — Network and Resource Services

### Lab 05: DNS Configuration and Troubleshooting
**Status:** Planned

Objectives:
- Explore DNS zones and records.
- Understand DNS name resolution in Active Directory.
- Verify SRV records and domain-controller discovery.
- Troubleshoot common DNS issues.

### Lab 06: DHCP Server Configuration
**Status:** Planned

Objectives:
- Install and configure the DHCP Server role.
- Create and configure an IP address scope.
- Configure exclusions, reservations, and options.
- Verify IP address assignment on a client.

### Lab 07: File Server and NTFS Permissions
**Status:** Planned

Objectives:
- Configure shared folders.
- Set share and NTFS permissions.
- Apply group-based access controls.
- Test authorized and unauthorized access.

## Phase 4 — Group Policy and Account Security

### Lab 08: Password and Account Lockout Policies
**Status:** Planned

Objectives:
- Configure password policy settings.
- Configure account lockout thresholds and durations.
- Understand domain password policy scope.
- Verify policy behavior using test accounts.

### Lab 09: Group Policy Troubleshooting
**Status:** Planned

Objectives:
- Investigate GPO processing failures.
- Review `gpresult` reports.
- Use Group Policy Results in GPMC.
- Check security filtering, inheritance, and event logs.
- Document corrective actions and verification results.

## Phase 5 — Security and Recovery

### Lab 10: Active Directory Security and Hardening
**Status:** Planned

Objectives:
- Review privileged group membership.
- Apply least-privilege principles.
- Review account and authentication security.
- Evaluate security-related Group Policy settings.
- Document security configuration checks.

### Lab 11: Active Directory Backup and Recovery
**Status:** Planned

Objectives:
- Understand System State backup.
- Review Domain Controller recovery requirements.
- Practice recovery procedures in an isolated lab.
- Document backup and restoration verification.

### Lab 12: Active Directory Replication and Sites
**Status:** Planned

Objectives:
- Understand Active Directory sites and subnets.
- Explore replication concepts.
- Review replication health with appropriate tools.
- Investigate common replication issues in a multi-DC lab, if available.

## Documentation Standards

Each lab should include, where applicable:

- `README.md` — Objectives, prerequisites, implementation steps, and verification.
- A lab report in PDF format.
- A `screenshots/` directory containing relevant evidence.
- Configuration notes and troubleshooting observations.
- A verification checklist showing the actual results.

## Recommended Repository Structure

```text
Active-Directory-Labs/
├── README.md
├── .gitignore
├── docs/
│   └── lab-roadmap.md
├── Lab-01-Active-Directory-Installation/
│   ├── README.md
│   ├── Lab-01-Report.pdf
│   └── screenshots/
└── Lab-02-Group-Policy-Management/
    ├── README.md
    ├── Lab-02-Report.pdf
    └── screenshots/
```

Additional lab folders should be added as the roadmap progresses.

## Progress Tracking Principles

- Mark a lab **Completed** only after implementation and verification.
- Mark a lab **In Progress** when work has started but is not yet fully verified.
- Keep future exercises marked **Planned**.
- Update the main repository README when a new lab is completed.
- Do not include passwords, private keys, or other sensitive information in reports or screenshots.

## Project Goal

Build a documented, hands-on Active Directory portfolio that demonstrates progressively stronger skills in Windows Server administration, identity management, Group Policy, network services, troubleshooting, and security.

---

**Project:** Active Directory Labs  
**Current Progress:** Labs 01 and 02 documented as completed  
**Next Recommended Lab:** Lab 03 — Organizational Unit Management
