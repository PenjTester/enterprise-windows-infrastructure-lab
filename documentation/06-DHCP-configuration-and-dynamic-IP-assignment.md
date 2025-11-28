# 06 – DHCP Configuration & Dynamic IP Assignment

**Date Started:** 11/28/2025  
**Server:** LAB-DC01 (Windows Server 2019)  
**Client:** LAB-CLIENT01 (Windows 10 Pro)

---

## Goal of This Milestone

This milestone introduces **Dynamic Host Configuration Protocol (DHCP)** into the Active Directory lab environment. DHCP is responsible for automatically assigning IP addresses, DNS settings, and lease information to client systems on a network.  

Until now, client systems used *static addressing* to join the domain and communicate with LAB-DC01. In an enterprise environment, this does not scale. DHCP enables:

- Automated IP management  
- Consistent DNS distribution  
- Reduced administrative overhead  
- Centralized lease tracking  
- Reliable communication for domain-joined machines  

After completing this milestone, the lab environment transitions from static IP configuration to a **fully automated, domain-integrated DHCP infrastructure**.

---

# 1. Installing the DHCP Server Role (LAB-DC01)

The DHCP Server role must be installed on LAB-DC01 before the server can issue dynamic IP addresses or be authorized in Active Directory.

### Why This Matters
- DHCP must run on a server that is always online and trusted by the domain.  
- In AD environments, DHCP servers must be **authorized** to prevent rogue DHCP servers from issuing addresses.  
- Installing this role is the foundation of centralized IP management.

---

### Screenshot — Server Manager Dashboard (Starting Point)
<img width="1731" height="878" alt="Screenshot 2025-11-28 124830" src="https://github.com/user-attachments/assets/0e6849bb-e12a-4e68-9a5f-b4e4732adefa" />

### Screenshot — Add Roles and Features Wizard (Before You Begin)
<img width="787" height="555" alt="Screenshot 2025-11-28 124858" src="https://github.com/user-attachments/assets/d5773d2b-591f-47df-904e-aea019498a61" />

### Screenshot — Installation Type (Role-Based Installation)
<img width="786" height="556" alt="Screenshot 2025-11-28 124942" src="https://github.com/user-attachments/assets/acf8c9e6-ad37-4fdd-a1e8-f383b8cedbc8" />

### Screenshot — Server Selection (LAB-DC01)
<img width="781" height="556" alt="Screenshot 2025-11-28 125023" src="https://github.com/user-attachments/assets/19569534-abe7-4ff6-ba70-360d48791b94" />

### Screenshot — DHCP Server Role Selected
<img width="784" height="557" alt="Screenshot 2025-11-28 125131" src="https://github.com/user-attachments/assets/e5f91668-bbc4-498b-8fd9-00965d81b358" />

### Screenshot — Confirm Installation
<img width="784" height="555" alt="Screenshot 2025-11-28 125415" src="https://github.com/user-attachments/assets/d39adc04-4c28-4327-9dbf-dd42eaf0788b" />

### Screenshot — Installation Succeeded
<img width="332" height="158" alt="Screenshot 2025-11-28 12582233" src="https://github.com/user-attachments/assets/3bfdb4ff-5f81-43bd-b6c4-b67536adebf0" />

---

# 2. DHCP Post-Installation Tasks & Authorization

After DHCP is installed, Windows requires a **post-installation configuration** to:

- Create DHCP security groups  
- Grant DHCP privileges  
- Authorize the server in Active Directory  

### Why Authorization Matters  
In Active Directory, **unauthorized DHCP servers are blocked** to prevent accidental or malicious IP address assignments. Authorizing LAB-DC01 ensures it is trusted to issue leases on the domain.

---

### Screenshot — Server Manager Notification: Complete DHCP Configuration
<img width="332" height="112" alt="Screenshot 2025-11-28 12582234" src="https://github.com/user-attachments/assets/b6689565-c359-4d48-b470-1f78dde200b6" />

### Screenshot — DHCP Post-Install Wizard Introduction
<img width="759" height="561" alt="Screenshot 2025-11-28 125843" src="https://github.com/user-attachments/assets/9c3e7909-1bbe-41f3-87af-d52f9e315af1" />

### Screenshot — Authorization Credentials (LAB/Administrator)
<img width="760" height="555" alt="Screenshot 2025-11-28 125926" src="https://github.com/user-attachments/assets/307a3870-3ff2-4926-842f-a6fa091ea8bd" />

### Screenshot — Post-Install Configuration Completed Successfully  
<img width="756" height="558" alt="Screenshot 2025-11-28 130128" src="https://github.com/user-attachments/assets/eb3b9093-4681-4319-b77a-b93dd22b5060" />

### Screenshot — DHCP Tool Now Available Under Server Manager → Tools
<img width="370" height="810" alt="Screenshot 2025-11-28 130203" src="https://github.com/user-attachments/assets/2da549b6-c211-40f4-9da5-009a491dbba2" />

---

# 3. Creating the DHCP Scope (10.0.2.0/24 Network)

With DHCP installed and authorized, the next step is to create a **DHCP Scope**, which defines the usable IP range for LAB-CLIENT01 and future clients.

## 3.1 Scope Overview

| Setting | Value |
|--------|-------|
| Network | 10.0.2.0/24 |
| Scope Name | Lab_Network_Scope |
| Start IP | 10.0.2.100 |
| End IP | 10.0.2.200 |
| Subnet Mask | 255.255.255.0 |
| Exclusions | None |
| Default Gateway | *(blank — Internal Network)* |
| DNS Server | 10.0.2.10 (LAB-DC01) |

### Why This Configuration?
- **10.0.2.0/24** is the VirtualBox Internal Network used throughout the project.  
- LAB-DC01 (10.0.2.10) must remain static; therefore the dynamic range begins at 100.  
- No default gateway exists because the lab network is intentionally **isolated** and not connected to the internet.  
- DNS must point to LAB-DC01 for domain resolution.

---

### Screenshot — DHCP Console (LAB-DC01 Expanded)
<img width="807" height="572" alt="Screenshot 2025-11-28 130456" src="https://github.com/user-attachments/assets/c3628644-c7b3-4202-a9f1-d4fae3b4e924" />

### Screenshot — Creating a New Scope
<img width="230" height="355" alt="Screenshot 2025-11-28 1306120" src="https://github.com/user-attachments/assets/d790f5e7-45e5-4a23-b22c-c252eb56c26f" />

### Screenshot — New Scope Wizard Introduction
<img width="515" height="420" alt="Screenshot 2025-11-28 130626" src="https://github.com/user-attachments/assets/41c022d3-dcbd-41be-8f76-e92a00f844ae" />

### Screenshot — Scope Name and Description
<img width="513" height="422" alt="Screenshot 2025-11-28 130713" src="https://github.com/user-attachments/assets/75ba5e49-b5c3-437e-8e37-827a9325a419" />

### Screenshot — IP Address Range (10.0.2.100 → 10.0.2.200)
<img width="514" height="423" alt="Screenshot 2025-11-28 133006" src="https://github.com/user-attachments/assets/49ec8dbb-75b5-4bab-9415-b29404cec6cc" />

### Screenshot — Exclusions Screen (None Configured)
<img width="520" height="424" alt="Screenshot 2025-11-28 133025" src="https://github.com/user-attachments/assets/2396a464-ed3b-4881-a062-d563379e9302" />

### Screenshot — Lease Duration Screen
<img width="515" height="421" alt="Screenshot 2025-11-28 133043" src="https://github.com/user-attachments/assets/2a0a6fd7-fe55-43c6-92a9-93d637cd7746" />

### Screenshot — Configure DHCP Options Now
<img width="516" height="423" alt="Screenshot 2025-11-28 133059" src="https://github.com/user-attachments/assets/f97b0df9-5f11-45b9-b15f-695af591deb5" />

### Screenshot — Default Gateway (Left Blank)
<img width="517" height="374" alt="Screenshot 2025-11-28 133133" src="https://github.com/user-attachments/assets/faf603f6-344a-4b77-8441-de7724223ee6" />

### Screenshot — DNS Configuration (lab.local → 10.0.2.10)
<img width="513" height="421" alt="Screenshot 2025-11-28 133436" src="https://github.com/user-attachments/assets/dbb2ba59-8c79-489d-8bd4-e68b665cd995" />

### Screenshot — WINS (Left Blank)
<img width="513" height="422" alt="Screenshot 2025-11-28 133458" src="https://github.com/user-attachments/assets/085edefe-5b7e-497b-a40b-e00ce283c733" />

### Screenshot — Scope Activation Confirmation
<img width="513" height="424" alt="Screenshot 2025-11-28 133517" src="https://github.com/user-attachments/assets/64134b20-98e4-4849-ae06-178dfe47bc22" />

### Screenshot — Active Scope in DHCP Console
<img width="776" height="330" alt="Screenshot 2025-11-28 133601" src="https://github.com/user-attachments/assets/49b5f6c8-b927-41e3-b1c6-5612b0828a9b" />

---

# 4. DHCP Client Validation (LAB-CLIENT01)

Now that LAB-DC01 is prepared to issue dynamic leases, the client machine must switch from its previous static configuration to DHCP.

## Why This Step Is Critical
- Domain clients must use **LAB-DC01** as DNS to function correctly  
- DHCP will automatically provide DNS settings going forward  
- Validating DHCP ensures the scope and server configuration works end-to-end

---

## 4.1 Switching IPv4 Settings to DHCP

### Screenshot — IPv4 Properties Set to “Obtain Automatically”
<img width="396" height="454" alt="Screenshot 2025-11-28 141302" src="https://github.com/user-attachments/assets/8616c5cb-b8d4-4ad9-ac07-84b3aba3bd35" />

---

## 4.2 Renewing DHCP Lease

`ipconfig /release` removes the previous address.  
`ipconfig /renew` requests a new address from the DHCP Server (LAB-DC01).

### Screenshot — ipconfig /release Output
<img width="569" height="302" alt="Screenshot 2025-11-28 141357" src="https://github.com/user-attachments/assets/50bf7a9d-bf0f-480d-b512-2214ac552f6f" />

### Screenshot — ipconfig /renew Output (Assigned 10.0.2.100)
<img width="572" height="214" alt="Screenshot 2025-11-28 141547" src="https://github.com/user-attachments/assets/2e454c53-441b-48d2-a838-499c5e1ff5c7" />

---

## 4.3 Full IP Configuration Verification

### Key Expected Results
- DHCP Enabled: Yes  
- IPv4 Address: 10.0.2.100  
- DNS Server: 10.0.2.10  
- Default Gateway: *(blank – correct for internal networks)*  
- DHCP Server: 10.0.2.10  

### Screenshot — ipconfig /all Showing Valid DHCP Details
<img width="710" height="302" alt="Screenshot 2025-11-28 141715" src="https://github.com/user-attachments/assets/c4fc5186-c4ef-4fa1-9e48-2de79da71aa4" />

---

## 4.4 DNS Resolution Test

A successful ping to LAB-DC01 confirms:

- DNS assignment from DHCP  
- Client-to-server connectivity  
- Domain name resolution functioning properly

### Screenshot — Successful ping to lab-dc01
<img width="540" height="216" alt="Screenshot 2025-11-28 141740" src="https://github.com/user-attachments/assets/8d5088be-35d0-43db-ad8e-3aa2f41b0e42" />

---

# 5. Validating DHCP Lease From the Server (LAB-DC01)

The final verification is performed directly on LAB-DC01 by checking the **DHCP Address Leases** section.

This ensures:

- LAB-CLIENT01 requested a lease  
- LAB-DC01 issued and recorded the lease  
- DHCP scope tracking is working correctly  

---

### Screenshot — DHCP Console with Scope Expanded
<img width="813" height="267" alt="Screenshot 2025-11-28 142102" src="https://github.com/user-attachments/assets/a276b791-dee0-438d-9f24-2ff0f80481dd" />

### Screenshot — Address Lease Showing LAB-CLIENT01 → 10.0.2.100
<img width="1170" height="243" alt="Screenshot 2025-11-28 142241" src="https://github.com/user-attachments/assets/c492a0a7-75a2-49dc-a0d8-865406501476" />

---

# 6. Results Summary

| Verification Step | Result |
|-------------------|--------|
| DHCP role installed on LAB-DC01 | ✔️ |
| DHCP server authorized in AD | ✔️ |
| Scope created for 10.0.2.0/24 | ✔️ |
| DNS configured to 10.0.2.10 | ✔️ |
| LAB-CLIENT01 successfully received DHCP lease | ✔️ |
| DNS queries resolved correctly | ✔️ |
| Lease visible in DHCP console | ✔️ |

---

# Final Outcome

Dynamic IP addressing is now fully operational within the lab environment. LAB-DC01 acts as the enterprise DHCP Server, distributing IP address leases, DNS configuration, and subnet details to domain-joined clients.  

This milestone establishes a foundation for upcoming tasks involving:

- DNS management  
- Group Policy  
- Centralized workstation configuration  
- Network services publishing  
- And future enterprise systems requiring automated IP addressing  

With DHCP now active and validated, the lab environment has progressed to a more scalable and enterprise-authentic configuration.

