# Enterprise Windows Lab (VirtualBox)

This repository contains a small virtual enterprise environment built in Oracle VirtualBox.  
The goal is to set up a realistic Windows domain with a domain controller, DNS, DHCP, Active Directory structure, and a Windows 10 client. Each milestone focuses on a common administrative task, with step-by-step documentation and screenshots stored in the `documentation` folder.

This project is meant to be practical: build the environment, configure the services, and verify that everything works. The documentation reflects what was done, why it was done, and how it behaves.

---

## Lab Environment

| Component | Details |
|----------|---------|
| Virtualization | Oracle VirtualBox |
| Domain Name | `lab.local` |
| Server | Windows Server 2019 (Domain Controller / DNS / DHCP) |
| Workstation | Windows 10 Pro (Domain-joined client) |
| Network Mode | Internal Network (`lab-network`) |

The lab runs fully isolated inside VirtualBox and is used for practicing common enterprise administration skills.

---

## Milestones

Each milestone has its own detailed documentation file under `/documentation/`.

| Milestone | Status | Documentation |
|----------|--------|---------------|
| 01 – Domain Controller Setup |  Completed | `/documentation/01-domain-controller-setup.md` |
| 02 – Client Domain Join |  Completed | `/documentation/02-client-domain-join.md` |
| 03 – Active Directory Identity Management |  Completed | `/documentation/03-active-directory-identity-management.md` |
| 04 – Group & Access Control |  Completed | `/documentation/04-group-management-and-access-control.md` |
| 05 – File Sharing & NTFS Permissions |  Completed | `/documentation/05-file-sharing-permissions-and-ntfs-management.md` |
| 06 – DHCP Configuration (Dynamic IP Assignment) |  Completed | `/documentation/06-dhcp-configuration-dynamic-ip-assignment.md` |
| 07 – DNS Management & Domain Service Reliability |  Completed | `/documentation/07-dns-management-and-domain-service-reliability.md` |
| 08 – Workstation OU Structure & Baseline GPO Deployment |  Completed | `/documentation/08-workstation-OU-structure-and-baseline-GPO-deployment.md` |
| 09 – Advanced Workstation Hardening | Completed | `/documentation/09-advanced-workstation-hardening.md` |
| 10 – Domain Admin & Privileged Access Hardening | Completed | `/documentation/10-privileged-access-management-and-administrative-account-hardening.md` |
| 11 – User Lifecycle & Access Operations | Completed | `/documentation/11-user-lifecycle-and-access-operations.md` |
| 12 – Break-Glass Access for Hardened Workstations | Completed | `/documentation/12-break-glass-access-for-hardened-workstations.md` |
| 13 – Patch Management & System Maintenance | Upcoming | `/documentation/13-patch-management-and-system-maintenance.md` |
| 14 – Monitoring & Event Review | Upcoming | `/documentation/14-monitoring-and-event-review.md` |
| 15 – Backup & Recovery Procedures | Upcoming | `/documentation/15-backup-and-recovery-procedures.md` |
| 16 – Enterprise Administration Capstone & Final Documentation | Upcoming | `/documentation/16-enterprise-administration-capstone-and-final-documentation.md` |

---

## What This Project Covers

- Installing and configuring a Windows Server domain controller  
- Configuring Active Directory Domain Services (AD DS)  
- Setting up DNS zones, records, scavenging, and domain service reliability  
- Configuring DHCP scopes, leases, reservations, and dynamic DNS registration  
- Joining Windows 10 clients to the domain  
- Creating and structuring Organizational Units (OUs) for enterprise management  
- Managing users, groups, roles, and access control with ADUC  
- Implementing NTFS and SMB share permissions  
- Deploying baseline Group Policy Objects (GPOs) for workstation configuration  
- Enforcing security hardening via GPO (LAN Manager levels, SMB signing, NTLMv2 requirements)  
- Restricting credential delegation (default, saved, and fresh credentials)  
- Hardening local accounts, including blocking network logon for local users  
- Enforcing LDAP signing and secure authentication flows  
- Enabling enterprise-grade PowerShell visibility (script block logging, module logging, transcription)  
- Restricting inbound and outbound NTLM traffic  
- Implementing workstation hardening aligned with enterprise security principles  
- Verifying security posture using GPResult, Local Security Policy, Event Viewer, NETLOGON tests, and SMB/ICMP behavior  
- Documenting each configuration step with screenshots and organized milestone folders  
- Maintaining clean, milestone-based configuration drift control throughout the project  


*The documentation folder contains the full walkthroughs.*

---


## Notes

This README is intentionally simple.  
Most of the detail lives in the milestone documentation files.  
As milestones are completed, the table above can be updated as needed.

