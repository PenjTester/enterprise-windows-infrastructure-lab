# Enterprise IT Lab (VirtualBox)

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
| 08 – Group Policy Configuration & Workstation Management |  Upcoming | *(coming soon)* |

---

## What This Project Covers

- Installing and configuring a Windows Server domain controller  
- AD DS, DNS, and DHCP setup  
- Joining Windows clients to the domain  
- Creating an OU structure  
- Managing users, groups, and permissions  
- NTFS and share permissions  
- DNS forward lookup, reverse lookup, SRV records, and scavenging  
- DHCP scopes, leases, and dynamic registration  
- Troubleshooting using built-in tools (`ipconfig`, `nslookup`, `whoami`, etc.)  
- Documenting each administrative task with screenshots  
- Keeping configuration changes organized by milestone

The documentation folder contains the full walkthroughs.

---

## Next Steps

The next milestone will focus on **Group Policy**, including workstation configuration, security settings, user experience settings, and domain-wide controls.

---

## Notes

This README is intentionally simple.  
Most of the detail lives in the milestone documentation files.  
As milestones are completed, the table above can be updated as needed.

