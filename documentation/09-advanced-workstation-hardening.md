# 09 – Advanced Workstation Hardening

**DATE:** 2025-12-03  
**SERVER:** LAB-DC01 (Windows Server 2019)  
**CLIENT:** LAB-CLIENT01 (Windows 10)

---

## Overview

In this milestone, the lab environment moves beyond basic domain and workstation configuration into **real security hardening** for domain-joined clients.

Instead of leaving LAB-CLIENT01 as a default Windows 10 box, this milestone introduces an **“Advanced Workstation Hardening”** Group Policy Object (GPO) targeted at the **Lab Workstations** Organizational Unit (OU). The goal is to make the workstation behave more like a locked-down enterprise endpoint:

- Stronger NTLM settings and session security  
- SMB signing enforced  
- Local accounts restricted from network use  
- PowerShell activity logged in detail  
- Reduced avenues for credential abuse and lateral movement  

By the end of this milestone, LAB-CLIENT01 is still usable for normal domain work, but much less friendly to attackers.

---

## Prerequisites

This milestone assumes the following items are already in place:

- A functional domain: **lab.local**
- Domain controller: **LAB-DC01** (DNS + DHCP + AD DS configured in earlier milestones)
- Domain-joined Windows 10 client: **LAB-CLIENT01**
- OU structure already created (from earlier work), including:  
  - `_Computers`  
  - `_Workstations`  
  - `Lab Workstations` (containing LAB-CLIENT01)
- Previous baseline GPOs (e.g., **Baseline Workstation Policy**) already linked to the workstation OU

---

## Goals of This Milestone

This milestone focuses on:

- Creating and linking a **dedicated hardening GPO** for domain workstations  
- Applying advanced security settings that affect NTLM, SMB, local accounts, and credential delegation  
- Enabling PowerShell logging for better visibility  
- Verifying that the policy settings are applied and that the workstation’s behavior has changed in measurable ways

---


## Phase 1 – Create and Link the “Advanced Workstation Hardening” GPO


The first step is to create a dedicated GPO for advanced hardening and link it only where it’s intended: the **Lab Workstations** OU.

## 1.1 – Open Active Directory Users and Computers and confirm OU structure

In **ADUC** on LAB-DC01:

- Expand: `lab.local`
- Confirm the presence of:  
  - `_Computers`  
  - `_Workstations`  
  - `Lab Workstations`  
- Confirm that **LAB-CLIENT01** resides in the **Lab Workstations** OU.


*Screenshot – Group Policy Management showing lab.local → _Computers → _Workstations → Lab Workstations with LAB-CLIENT01 inside*  
<img width="1187" height="498" alt="Screenshot 2025-12-03 163743" src="https://github.com/user-attachments/assets/ff752637-8a0d-4cc8-b892-c42bcec2f024" />

---

## 1.2 – Create and Confirm the “Advanced Workstation Hardening” GPO

In **Group Policy Management** on **LAB-DC01**:

1. Right-click **Lab Workstations** and select:  
   **Create a GPO in this domain, and Link it here…**

2. Name the GPO:  
   **Advanced Workstation Hardening**

3. Leave **Source Starter GPO** as **(none)**.

4. After creation, ensure the GPO now appears under the **Lab Workstations** OU.

5. With **Lab Workstations** selected in the left pane, confirm the link properties in the right pane:
   - **Link Enabled:** Yes  
   - **GPO Status:** Enabled  
   - Baseline Workstation Policy remains linked (should appear above or alongside the new GPO).


*Screenshot — Lab Workstations OU showing both **Baseline Workstation Policy** and **Advanced Workstation Hardening** linked,  
with **Link Enabled = Yes** and **GPO Status = Enabled***  
<img width="1186" height="437" alt="Screenshot 2025-12-03 130351" src="https://github.com/user-attachments/assets/7ddc5e8a-1a96-423d-9e4b-1853b3b2056f" />


---

## Phase 2 – Core Network & Authentication Hardening (Computer Configuration)

This phase focuses on the **Computer Configuration** section of the GPO, making changes that affect how the workstation participates in the network (NTLM, SMB, local accounts, etc.).

> All settings in this phase are configured in:  
> **Computer Configuration → Policies → Windows Settings → Security Settings**

---

### 2.1 – Enforce SMB Signing and LAN Manager Hardening

In the **Advanced Workstation Hardening** GPO editor:

1. Navigate to:  
   `Computer Configuration → Policies → Windows Settings → Security Settings → Local Policies → Security Options`

2. Configure the following settings:

- **Microsoft network client: Digitally sign communications (always)**  
  - Set to **Enabled**  
  - Effect: The workstation always signs SMB client traffic.

<img width="415" height="503" alt="Screenshot 2025-12-03 130932" src="https://github.com/user-attachments/assets/173195de-da6b-459f-8d4a-ecdbe2251f99" />

---

- **Microsoft network client: Digitally sign communications (if server agrees)**  
  - Set to **Enabled** (if configured)  
  - Effect: Will sign SMB traffic when supported by the server.

<img width="416" height="502" alt="image" src="https://github.com/user-attachments/assets/ff1f83e3-2ed8-4cf1-8042-d00e58c79761" />


---

- **Microsoft network server: Digitally sign communications (always)**  
  - Set to **Enabled**  
  - Effect: The workstation, acting as server, requires signed SMB connections.

 <img width="412" height="503" alt="image" src="https://github.com/user-attachments/assets/2886ab05-3617-48ad-9628-03b771786e8a" />


---

- **Network security: LAN Manager authentication level**  
  - Set to **Send NTLMv2 response only. Refuse LM & NTLM**  
  - Effect: Legacy LM and NTLMv1 are refused; only NTLMv2 is allowed.

*Screenshot – Security Options view showing SMB signing and LAN Manager authentication level configured*  
<img width="416" height="504" alt="Screenshot 2025-12-03 131343" src="https://github.com/user-attachments/assets/3d1359e8-c5ff-4189-a503-1212fa8f169f" />

---

### 2.2 – Harden NTLM Session Security

In the same **Security Options** view:

- **Network security: Minimum session security for NTLM SSP based (including secure RPC) clients**  
  - Set to **Enabled**  
  - Options checked:  
    - **Require NTLMv2 session security**  
    - **Require 128-bit encryption**

- **Network security: Minimum session security for NTLM SSP based (including secure RPC) servers**  
  - Set similarly to require NTLMv2 and 128-bit encryption (if configured).

This ensures both client and server components enforce strong session security for NTLM traffic.

*Screenshot – Security Options properties for “Minimum session security for NTLM SSP (clients)” showing NTLMv2 + 128-bit encryption required*  
<img width="414" height="499" alt="Screenshot 2025-12-03 161758" src="https://github.com/user-attachments/assets/002aac0f-8764-4c26-a18b-1df922062c3c" />

---

### 2.3 – Restrict Local Accounts from Network Use

Next, restrict local accounts so they cannot be used for remote network logons.

Navigate to:

`Computer Configuration → Policies → Windows Settings → Security Settings → Local Policies → User Rights Assignment`

Configure:

- **Deny access to this computer from the network**  
  - Check **Define these policy settings**  
  - Add:  
    - **LOCAL ACCOUNT**  
    - **LOCAL ACCOUNT AND MEMBER OF ADMINISTRATORS GROUP**  
  - Effect: Local accounts (including the local Administrator) cannot be used over the network.

*Screenshot – User Rights Assignment for “Deny access to this computer from the network” showing **Local Account** and **Local Account and Member of Administrators Group***  
<img width="414" height="505" alt="Screenshot 2025-12-03 134556" src="https://github.com/user-attachments/assets/1ec28b64-b378-4cfd-9ad7-cb0ba09ab542" />

---

## Phase 3 – Credential Delegation & PowerShell Logging

This phase focuses on reducing credential exposure and increasing visibility into PowerShell activity.

---

### 3.1 – Deny Delegation of Credentials

Navigate to:

`Computer Configuration → Policies → Administrative Templates → System → Credentials Delegation`

Configure:

- **Deny delegating default credentials**  
  - Set to **Enabled**  
  - Click **Show…** and add:  
    - `*`  
  - Effect: Prevents default credentials from being delegated to remote servers.

- **Deny delegating saved credentials**  
  - Set to **Enabled**  
  - Click **Show…** and add:  
    - `*`  
  - Effect: Prevents saved credentials from being delegated.

These settings reduce the risk of credentials being silently reused across systems.

*Screenshot – “Deny delegating default credentials” and “Deny delegating saved credentials” configured with a wildcard entry*  
<img width="817" height="598" alt="Screenshot 2025-12-03 140636" src="https://github.com/user-attachments/assets/b430666f-87ae-406f-acd6-b02aba508115" />

---

### 3.2 – Disable Windows Error Reporting (Optional Hardening)

Navigate to:

`Computer Configuration → Policies → Administrative Templates → Windows Components → Windows Error Reporting`

Configure:

- **Disable Windows Error Reporting**  
  - Set to **Enabled**  
  - Effect: Prevents sending error data to Microsoft, reducing information leakage.

*Screenshot – “Disable Windows Error Reporting” policy enabled*  
<img width="686" height="632" alt="image" src="https://github.com/user-attachments/assets/3193ce10-07c2-4163-953c-749194857a65" />

---

### 3.3 – Enable PowerShell Script Block and Module Logging

Navigate to:

`Computer Configuration → Policies → Administrative Templates → Windows Components → Windows PowerShell`

Configure:

- **Turn on PowerShell Script Block Logging**  
  - Set to **Enabled**  
  - Effect: Logs evaluated PowerShell script blocks to the event log.

- **Turn on PowerShell Module Logging**  
  - Set to **Enabled**  
  - Click **Show…** and add appropriate module patterns (e.g. `Microsoft.PowerShell.*`).  
  - Effect: Logs PowerShell module usage for specified modules.

- **Turn on Powershell Transcription**
  -Set to **Enabled**

These settings cause detailed PowerShell activity to be written into the **PowerShell Operational** event log.

*Screenshot – Windows PowerShell policy node showing Script Block Logging, and Module Logging, and Powershell Transcription enabled*  
<img width="936" height="587" alt="Screenshot 2025-12-03 135757" src="https://github.com/user-attachments/assets/88f7e789-30b9-4206-abd4-edcb48a4371c" />

*Screenshot – Module Logging “Show Contents” dialog with a wildcard entry (e.g., Microsoft.PowerShell.*)*  
<img width="492" height="328" alt="image" src="https://github.com/user-attachments/assets/befa574c-eb6b-4227-988d-a8c9542ce01f" />

---

## Phase 4 – Confirm GPO Application on LAB-CLIENT01

With the GPO configured, the next step is to confirm that LAB-CLIENT01 is actually receiving and applying the Advanced Workstation Hardening policies.

---

### 4.1 – Force Group Policy Update and Reboot

On **LAB-CLIENT01** (logged in as `LAB\Administrator`):

1. Open **Command Prompt** as Administrator.  
2. Run:  
   ```bash
   gpupdate /force

3. If prompted to restart, allow the workstation to reboot.

*Screenshot – gpupdate /force completing successfully*  
<img width="608" height="116" alt="Screenshot 2025-12-03 163035" src="https://github.com/user-attachments/assets/b9adbb59-8a39-41b1-8b6a-b5ae698b9862" />

---

### 4.2 – Verify GPOs via gpresult /r

On LAB-CLIENT01:

Run the following in an elevated command prompt:

gpresult /r

Confirm that the following appear under Computer Settings > Applied Group Policy Objects:
- Advanced Workstation Hardening
- Baseline Workstation Policy
- Default Domain Policy

*Screenshot – gpresult output showing Advanced Workstation Hardening applied*  
<img width="979" height="846" alt="Screenshot 2025-12-03 164836" src="https://github.com/user-attachments/assets/a79b58bb-2097-4cc7-b365-4a9b60fa9926" />

---

### 4.3 – Spot-check NTLM and SMB Settings in Local Security Policy

Open secpol.msc and navigate to:
Security Settings > Local Policies > Security Options

Verify that NTLM and SMB settings match the GPO configuration (NTLMv2 required, 128-bit encryption required, SMB signing enabled).

*Screenshot – Local Security Policy showing hardened NTLM/SMB values*  
<img width="1441" height="954" alt="Screenshot 2025-12-03 171407" src="https://github.com/user-attachments/assets/20cce926-746a-41e7-a617-6e1a2fe261a5" />

---

## Phase 5 – Behavioral Verification

These tests confirm that the workstation behaves differently as a result of the hardening.

---

### 5.1 – Verify PowerShell Logging in Event Viewer

On LAB-CLIENT01:

Run any PowerShell command (example: Get-Process | Select-Object -First 5)

Open Event Viewer:
Applications and Services Logs > Microsoft > Windows > PowerShell > Operational

Confirm new entries appear such as Event ID 4104.

*Screenshot – PowerShell Operational log showing script block logging*  
<img width="634" height="152" alt="Screenshot 2025-12-03 173416" src="https://github.com/user-attachments/assets/00eec5e1-4bf0-492e-9f35-e887f409b6b8" />

---

### 5.2 – Validate Outbound Communication Still Works

From LAB-CLIENT01, run:

ping lab-dc01

Successful replies confirm outbound workstation-to-DC communication is intact.

*Screenshot – LAB-CLIENT01 successfully pinging LAB-DC01*  
<img width="613" height="218" alt="image" src="https://github.com/user-attachments/assets/8684f47f-7223-4833-a7aa-33b95a1e30a3" />

---

### 5.3 – Validate Inbound Restrictions (Expected Behavior)

From LAB-DC01:

Attempt an SMB connection:

net use \\lab-client01\c$ /user:lab-client01\administrator *

Expected result:
System error 67 – The network name cannot be found

Attempt to ping LAB-CLIENT01:
Expected result: 100% loss

These failures confirm inbound SMB and ICMP are now blocked, and local accounts cannot be used for remote authentication.

*Screenshot – net use or ping showing expected failure*  
<img width="793" height="126" alt="image" src="https://github.com/user-attachments/assets/cbb16e87-6bdc-40f5-895d-a2cfe3e23cbf" />

---


## Appendix A — Additional Hardening Settings Applied (Not Shown in Main Phases)

The following configurations were applied during Milestone 09 but were not included in the step-by-step documentation sections. These settings further strengthen workstation security and complete the baseline hardening of domain-joined Windows 10 devices.

---

### **A.1 — Guest Account Restrictions**

| Setting | Final Value | Purpose |
|--------|-------------|---------|
| **Accounts: Guest account status** | Disabled | Prevents the built-in Guest account from being used for any authentication or local login. |

---

### **A.2 — Admin Approval Mode for Built-in Administrator**

| Setting | Final Value | Purpose |
|--------|-------------|---------|
| **User Account Control: Admin Approval Mode for the Built-in Administrator account** | Enabled | Forces UAC elevation prompts even when logged in as BUILTIN\Administrator, reducing privilege-misuse risk. |

---

### **A.3 — NTLM Network Restrictions**

| Policy | Final Value | Purpose |
|--------|-------------|---------|
| **Restrict NTLM: Incoming NTLM traffic** | Deny all accounts | Blocks all inbound NTLM authentication attempts. Eliminates NTLM-based lateral movement. |
| **Restrict NTLM: Outgoing NTLM traffic to remote servers** | Deny all | Prevents the workstation from authenticating to servers using NTLM. Forces Kerberos only. |

---

### **A.4 — LDAP Signing Requirements**

| Setting | Final Value | Purpose |
|--------|-------------|---------|
| **Network security: LDAP client signing requirements** | Require signing | Prevents LDAP downgrade attacks and ensures integrity-protected directory queries. |

---

### **A.5 — Windows Defender SmartScreen**

| Setting | Final Value | Purpose |
|--------|-------------|---------|
| **Configure Windows Defender SmartScreen** | Enabled (Warn and prevent bypass) | Provides reputation-based protection for downloaded files and scripts. |

---

### **A.6 — SMB & NTLM Session Security Values (Confirmed but Not Individually Screen-Captured)**

These settings were applied and verified through RSOP and local policy checks:

| Setting | Final Value |
|--------|-------------|
| **Minimum session security for NTLM SSP (clients)** | Require NTLMv2 session security + Require 128-bit encryption |
| **Minimum session security for NTLM SSP (servers)** | Require 128-bit encryption |
| **Microsoft network server: Digitally sign communications (always)** | Enabled |
| **Microsoft network client: Digitally sign communications (if server agrees)** | Enabled |

These enforce integrity-protected SMB traffic and eliminate LM/NTLM downgrade scenarios.

---

## Summary

Milestone 09 successfully hardened LAB-CLIENT01:

- SMB signing enforced  
- NTLM restricted to NTLMv2 with 128-bit encryption  
- Local accounts denied network logon  
- Credential delegation blocked  
- PowerShell logging enabled (script block and module logging)  
- Outbound communication still functional  
- Inbound SMB/ICMP appropriately restricted  

The workstation now behaves like a hardened enterprise endpoint.

---

## What’s Next?

Milestone 10 – Domain Admin & Privileged Access Hardening

Next steps include:

- Reviewing privileged groups
- Hardening administrative account usage
- Reducing lateral movement opportunities
- Continuing toward an enterprise-tier security model

---

