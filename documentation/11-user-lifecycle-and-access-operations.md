# 11 – User Lifecycle and Access Operations

**Date:** 2025-12-09  
**Server:** LAB-DC01 (Windows Server 2019)  
**Client:** LAB-CLIENT01 (Windows 10 Enterprise, domain-joined)  

---

## 11.1 Milestone Overview

In this milestone, I moved from “building the environment” to modeling **day-to-day user administration workflows**:

- Introduced an explicit **Help Desk Tier 1** role.
- Delegated **limited, task-focused permissions** over end-user accounts.
- Verified that Help Desk Tier 1 can **reset passwords** and perform basic account maintenance.
- Confirmed that Help Desk Tier 1 **cannot create/delete accounts or escalate privileges**.
- Re-applied workstation hardening so the environment remains secure and realistic.

The goal is to match what an entry-level help desk / junior sysadmin would actually do in a real Active Directory environment, and to show that I can design, implement, and validate that model in a controlled way.

---

## 11.2 Starting Point and OU Structure

By the time I started Milestone 11:

- The domain **lab.local** already existed.
- The OU structure under **lab.local** included:

  - `_Computers`
  - `_Groups`
  - `_IT`
  - `_Admins`
  - `_Users`
    - `_HR`
    - `_IT_Support`
    - `_Management`
    - `_Sales`
    - `_HelpDesk` (created at the beginning of this milestone)

- Standard user accounts for different departments already lived under these OUs
  (for example, **Michael Lee** in `_Sales`).
- Workstations were hardened by GPO (see Milestones 9 and 10), including:
  - Domain admins and privileged accounts **cannot log on locally** to workstations.
  - Local accounts are denied network access.
  - NTLM and SMB have stricter session and authentication policies.

Milestone 11 adds a realistic **Help Desk Tier 1** function on top of that foundation without weakening the security model long-term.

---

## 11.3 Creating the Help Desk OU and Tier 1 User

### 11.3A Help Desk OU

To keep Help Desk staff separated from normal end-users, I created a dedicated OU:

- Under **lab.local → `_Users`**, I created:
  - **`_HelpDesk`**

Rationale:

- Keeps Help Desk staff logically grouped in one place.
- Makes it easier to audit which accounts are operational vs. regular end-users.
- Keeps future delegation and higher-tier help desk roles cleaner and more maintainable.

*Screenshot: ADUC showing `_Users` with `_HR`, `_IT_Support`, `_Management`, `_Sales`, and the new `_HelpDesk` OU.*  
<img width="795" height="431" alt="Screenshot 2025-12-09 142830" src="https://github.com/user-attachments/assets/4b91de8f-f3ca-49d6-96af-b5b1963f4790" />

---

### 11.3B Creating the Tier 1 Help Desk User

Inside **`_HelpDesk`**, I created a new user to represent a Tier 1 help desk technician:

- **Name:** Alex Turner  
- **Logon name:** `aturner`  
- **UPN:** `aturner@lab.local`  
- **Display name:** `Alex Turner`  
- **Department / Title:** Help Desk Tier 1 (for clarity inside the consoles)

Key points:

- The account is a normal domain user, **not** a domain admin or local admin.
- It lives in `_HelpDesk`, not in the default `Users` container.
- It will gain operational permissions only via delegated rights and group membership, not via built-in admin groups.

*Screenshot: Alex Turner user object located under `_Users\_HelpDesk` in ADUC.*  
<img width="434" height="375" alt="Screenshot 2025-12-09 143007" src="https://github.com/user-attachments/assets/e277dbba-a7d1-4da6-8098-2edf379c6c13" />

---

## 11.4 HelpDesk_Tier1 Role Group and Membership

### 11.4A Creating the HelpDesk_Tier1 Group

Rather than assigning rights directly to a user, I followed **role-based access control (RBAC)** and created a security group for the role itself.

Under **lab.local → `_Groups`**, I created:

- **Group name:** `HelpDesk_Tier1`  
- **Group scope:** Global  
- **Group type:** Security

This group represents the **Help Desk Tier 1 role**. Any user placed in this group should inherit the same delegated capabilities.

*Screenshot: `HelpDesk_Tier1` security group in the `_Groups` OU.*  
<img width="695" height="502" alt="Screenshot 2025-12-09 145139" src="https://github.com/user-attachments/assets/b3be1b6d-4a9c-469a-b53b-3431ea4f2434" />

---

### 11.4B Adding Alex Turner to the Role Group

I then added **Alex Turner (ATurner)** as a member of the `HelpDesk_Tier1` group:

- On the group’s Members tab, Alex appears as a member.
- On Alex’s Member Of tab, `HelpDesk_Tier1` is listed.

This confirms that:

- The **role** is defined by the group.
- The **user** (ATurner) simply occupies that role.
- Additional Help Desk staff in the future can be added to `HelpDesk_Tier1` without redoing delegation.

*Screenshot: HelpDesk_Tier1 group properties showing Alex Turner as a member.*  
<img width="402" height="452" alt="Screenshot 2025-12-09 145754" src="https://github.com/user-attachments/assets/6586e563-6023-4b23-a826-0928eb3b20b2" />

---

## 11.5 Designing a Realistic Delegation Model

### 11.5A Deciding What Help Desk Tier 1 Should Be Allowed to Do

Before touching ACLs or wizards, I defined what a realistic Tier 1 Help Desk should be able to do.

**Tier 1 Help Desk – Allowed:**

- Reset user passwords.
- Force password change at next logon.
- Unlock locked-out accounts.
- Read basic user information.
- Update some non-sensitive properties (for example, phone number or office location).

**Tier 1 Help Desk – Not Allowed:**

- Create new user accounts.
- Delete user accounts.
- Modify group memberships or security groups.
- Change privileged/admin accounts.
- Log on interactively to servers as an administrator.

This reflects a real-world pattern: Tier 1 handles **operational support**, not provisioning or high-impact security tasks.

---

### 11.5B Where to Delegate: Choosing the `_Users` OU

All non-admin human user accounts live under **`_Users`** and its child OUs:

- `_HR`
- `_IT_Support`
- `_Management`
- `_Sales`
- `_HelpDesk`

So I delegated control at the **`_Users` OU** level. That means:

- `HelpDesk_Tier1` can operate on user accounts in all sub-OUs (HR, Sales, etc.).
- Admin and server-related accounts that live outside `_Users` (for example, in `_IT` or `_Admins`) are not affected.
- Delegation is centralized and easier to reason about instead of being scattered across many smaller OUs.

---

## 11.6 Delegating Password and Account Maintenance on `_Users`

### 11.6A Delegation of Control Wizard: Core Tasks

On **LAB-DC01**, in **Active Directory Users and Computers (ADUC)**:

- I right-clicked the **`_Users`** OU and selected **Delegate Control…** to open the Delegation of Control Wizard.
- I added the group **`HelpDesk_Tier1`** as the security principal to delegate to.

On the “Tasks to Delegate” page, I chose the core built-in tasks:

- Reset user passwords and force password change at next logon  
- Read all user information  

This combination gives `HelpDesk_Tier1` the essential operational abilities:

- Fix password-related issues without involving higher-tier admins.
- View user attributes (names, departments, phones, etc.) for troubleshooting.

*Screenshot: Delegation of Control Wizard on `_Users` selecting HelpDesk_Tier1 and various permissions.*  
<img width="497" height="387" alt="Screenshot 2025-12-09 150338" src="https://github.com/user-attachments/assets/9c38fb8d-e95c-4f98-8f36-1db004ce3983" />  

<img width="495" height="390" alt="Screenshot 2025-12-09 151108" src="https://github.com/user-attachments/assets/3c013dec-c707-4a87-aeed-88c239b0eddc" />

---

### 11.6B Adding Property-Specific Write Access for User Attributes

The built-in delegation tasks do not provide a simple “Write all user information” option without opening the door to unwanted access. To allow Help Desk Tier 1 to update some **non-sensitive user properties** (phone, office, etc.) without granting full control, I used a **custom task**:

- Ran the Delegation of Control Wizard again on `_Users`.
- Selected “Create a custom task to delegate”.
- Chose “Only the following objects in this folder” and selected **User objects**.
- On the permissions page, I selected specific property-related rights, such as:
  - Telephone number
  - Office location
  - Other basic contact fields as appropriate

The intent is that Help Desk Tier 1 can help keep directory information accurate without being able to alter group membership, security identifiers, or other sensitive settings.

*Screenshot: `_Users` OU Advanced Security dialog showing a HelpDesk_Tier1 ACE with property-specific write permissions on user objects.*  
<img width="1222" height="737" alt="Screenshot 2025-12-09 153044" src="https://github.com/user-attachments/assets/3d87baf9-1609-427c-9073-d3dec46c965d" />

---

## 11.7 Ensuring Help Desk Cannot Create or Delete Accounts

### 11.7A Identifying and Removing Unwanted Create/Delete Rights

During delegation, an ACE (Access Control Entry) appeared that allowed `HelpDesk_Tier1` to:

- Create user objects  
- Delete user objects  

This is **too much power** for a Tier 1 role. It effectively turns them into an account provisioning team, which was not the goal.

Using the **Advanced Security** settings on the `_Users` OU:

- I opened `_Users` → Properties → Security → Advanced.
- Located the ACE for **HelpDesk_Tier1**.
- Edited that ACE and **removed** any rights that allowed:
  - Create user objects
  - Delete user objects

I retained:

- Password-related rights.
- Property-specific write privileges on non-sensitive user attributes.
- The necessary read access.

Result:

- `HelpDesk_Tier1` can **maintain** existing user accounts (passwords, selected attributes).
- `HelpDesk_Tier1` cannot **create** or **delete** user accounts.
- Onboarding and offboarding can be reserved for a more senior role or separate provisioning process.

*Screenshot: Advanced Security settings for `_Users` showing HelpDesk_Tier1 ACE without create/delete user object rights.*  
<img width="1114" height="732" alt="image" src="https://github.com/user-attachments/assets/274041c1-6663-49a7-9772-6ec52db8fe07" />

---

## 11.8 Installing RSAT on LAB-CLIENT01

### 11.8A Why RSAT on a Workstation?

Real help desk staff typically work from **client machines**, not by logging on interactively to domain controllers. To simulate that:

- I installed **Remote Server Administration Tools (RSAT)** on **LAB-CLIENT01**.
- The goal was to manage Active Directory from the workstation, the way a real help desk tech would.

### 11.8B Handling the Network Constraint

The lab network is normally isolated via an **Internal Network (lab-network)** in VirtualBox. To install RSAT (which relies on Windows Update / online features):

- I temporarily switched the LAB-CLIENT01 NIC from:
  - Internal Network (lab-network)  
  to  
  - NAT (to reach the internet).
- Installed the necessary RSAT components from **Settings → Apps → Optional features**:
  - RSAT: Active Directory Domain Services and Lightweight Directory Services Tools  
  - RSAT: Group Policy Management Tools  
  - RSAT: Server Manager  
- Once installation succeeded, I switched the NIC back to:
  - Internal Network → lab-network

This preserved the isolated lab design while still enabling realistic tooling on the client.

*Screenshot: LAB-CLIENT01 Start Menu showing ADUC and other RSAT tools.*  
<img width="1488" height="936" alt="Screenshot 2025-12-10 143152" src="https://github.com/user-attachments/assets/a406e698-cb66-4373-869b-d21f7d1d616a" />  
<img width="1486" height="941" alt="Screenshot 2025-12-10 143205" src="https://github.com/user-attachments/assets/73018d6f-74eb-4446-9f69-f4ad109b4edd" />


---

## 11.9 Running ADUC on LAB-CLIENT01 as HelpDesk_Tier1

Because HelpDesk_Tier1 (LAB\ATurner) is an allowed workstation logon account—and because Milestones 9 and 10 explicitly blocked only privileged identities (Domain Admins, lab-admin, Administrator)—ATurner can authenticate interactively on LAB-CLIENT01 without violating any security boundaries.

This provides a realistic model of Tier 1 operations in many organizations: Help Desk staff work from managed workstations where they can run administrative tools, but do **not** possess domain-wide or workstation-privileged logon rights.

---

### 11.9A Why Log On Directly Instead of Using `runas`?

Originally, a `runas` pattern was considered for safety (logging on interactively as a low-privilege user, then launching tools as HelpDesk_Tier1).

However, in this lab:

- **HelpDesk_Tier1 is *not* a privileged account** (it has scoped delegation only within `_Users`).
- **HelpDesk_Tier1 is permitted to log on to workstations** (it is not included in any “Deny log on locally” policy).
- **Interactive logon as ATurner is realistic for Tier 1 personnel**, and does not introduce elevated risk.

Therefore, the simpler and more accurate workflow for this environment is:

> Log in directly as LAB\ATurner and launch ADUC normally.

---

### 11.9B Launching ADUC as HelpDesk_Tier1 on LAB-CLIENT01

On **LAB-CLIENT01**, the following process was used:

1. Logged into Windows with the delegated Help Desk account:  
   **LAB\ATurner**
2. Opened the Start Menu and launched:  
   **Active Directory Users and Computers (dsa.msc)**  
   (installed earlier through RSAT)
3. ADUC opened successfully, running under the full security context of HelpDesk_Tier1.
4. All operations performed inside ADUC—such as password resets—were executed with the delegated rights configured in `_Users`.

**Result:**

- ADUC ran without elevation, proving that **delegated permission—not privileged role membership—grants the abilities used in this milestone**.
- HelpDesk_Tier1 could perform allowed tasks (password reset) and was blocked from disallowed tasks (user deletion), demonstrating correct delegation boundaries.

*Screenshot: ADUC running on LAB-CLIENT01 as LAB\aturner*
<img width="1675" height="1136" alt="Screenshot 2025-12-10 145912" src="https://github.com/user-attachments/assets/e2c2f6b1-ab95-4a44-a7da-025f0a604464" />



---

## 11.10 Validating Delegated Help Desk Capabilities

### 11.10A Resetting a User Password (Expected: Allowed)

To test the password reset delegation, I used an existing user:

- **Michael Lee** in the `_Sales` OU.

Inside the ADUC console running as HelpDesk_Tier1:

- Navigated to `_Users\_Sales`.
- Right-clicked **Michael Lee** and chose Reset Password.
- Set a new temporary password and checked “User must change password at next logon”.

Result:

- The operation **succeeded** without an Access Denied error.

This confirmed that:

- The delegated rights on `_Users` are applied and working.
- HelpDesk_Tier1 can perform password resets across departmental OUs under `_Users`.

*Screenshot: Password reset dialog for Michael Lee under HelpDesk_Tier1 context.*  
<img width="369" height="177" alt="Screenshot 2025-12-09 173030" src="https://github.com/user-attachments/assets/2a22a726-bcc1-4d5e-b78b-bf4945286179" />

---

### 11.10B Attempting to Delete a User (Expected: Blocked)

To ensure the delegation did **not** accidentally grant destructive powers:

- Still in ADUC running as HelpDesk_Tier1, I right-clicked **Michael Lee** and selected Delete.
- Confirmed the deletion prompt.

Result:

- Active Directory returned an error similar to:

  - “You do not have sufficient privileges to delete Michael Lee or this object is protected from accidental deletion.”

This demonstrates that:

- The removal of create/delete rights worked as intended.
- Tier 1 Help Desk is intentionally **blocked** from user deletion (and, by design, from account creation).
- The role aligns with a “support and maintenance” posture, not lifecycle provisioning.

*Screenshot: Insufficient privileges error when HelpDesk_Tier1 attempts to delete Michael Lee.*  
<img width="424" height="165" alt="Screenshot 2025-12-09 173124" src="https://github.com/user-attachments/assets/43d7938f-462c-44ee-bbd7-8a84a6e59baa" />

---

## 11.11 Verification of Restored Workstation Hardening

After completing Help Desk Tier 1 testing, the final step of Milestone 11 was to **re-harden LAB-CLIENT01** so that it again matches the security posture from Milestones 9 and 10.  
Because local security policy is enforced via Group Policy, the cleanest way to verify this is from **LAB-DC01** using the **Group Policy Management Editor (GPMC)**.

The goal of this section is to show that:

- Workstations in the **Lab Workstations** OU are once again protected by the hardened GPOs.
- The key user-rights assignments that prevent privileged and local accounts from logging on to workstations are **back in place**.

---

### 11.11A Confirm the Workstation Hardening GPO Is Linked

On **LAB-DC01**:

1. Open **Group Policy Management**.
2. In the left pane, expand:

   - `Forest: lab.local`
   - `Domains`
   - `lab.local`
   - `_Computers`
   - `_Workstations`
   - `Lab Workstations`

3. In the right pane, confirm that the workstation-hardening GPOs are linked, for example:

   - `Baseline Workstation Policy`
   - `Advanced Workstation Hardening`
   - `Privileged Access – Block Domain Admins On Workstations`

This confirms that LAB-CLIENT01 (which lives in **Lab Workstations**) is still in scope for the hardening policies.

---

### 11.11B Verify User Rights Assignments in the Hardening GPO

Still on **LAB-DC01**, in **Group Policy Management**:

1. Right-click **`Privileged Access – Block Domain Admins On Workstations`** (or whatever name you used for the privileged-access GPO) and choose  
   **Edit…**
2. In the GPO editor, navigate to:

   - `Computer Configuration`
     - `Policies`
       - `Windows Settings`
         - `Security Settings`
           - `Local Policies`
             - `User Rights Assignment`

3. Confirm that the following settings are configured as expected:

   - **Deny log on locally**
     - Includes: `Domain Admins`, `lab-admin`  
       (and optionally `Administrator`, if you chose to add it in Milestone 10)
   - **Deny log on through Remote Desktop Services**
     - Includes: `Domain Admins`, `lab-admin`  
       (and optionally `Administrator`, if configured)
   - **Deny access to this computer from the network**
     - Includes: `Local account`, `Local account and member of Administrators group`

These entries confirm that:

- **Domain admins and the dedicated admin account (lab-admin)** are blocked from interactive and RDP logon on workstations.
- **Local Administrator–type accounts** cannot be used for network access to the workstation (e.g., SMB/administrative shares).

*Screenshot: Re-hardened User Rights Assignments*  
<img width="1928" height="1185" alt="Screenshot 2025-12-10 152952" src="https://github.com/user-attachments/assets/9a65a73a-0c0d-4002-9194-42ba6e3bc441" />


This screenshot visually documents that the deny policies were restored after testing and that the workstation is once again enforcing the hardened baseline.

---

## Milestone 11 Completion Summary

Milestone 11 implemented a realistic **Help Desk Tier 1** workflow while keeping the environment aligned with least-privilege principles and the hardened workstation baseline from earlier milestones.

By the end of this milestone, the lab demonstrates that:

- A dedicated **HelpDesk_Tier1** group exists under the `_Groups` structure.
- A Help Desk user (e.g., **Alex Turner – A.Turner**) resides in the `_Helpdesk` OU under `_Users` and is a member of **HelpDesk_Tier1`.
- Delegation on the `_Users` OU grants HelpDesk_Tier1 the ability to:
  - Reset user passwords.
  - Unlock user accounts.
  - Modify non-privileged user attributes (where configured).
- Help Desk **cannot**:
  - Delete user accounts.
  - Create new user accounts.
  - Modify or elevate privileged identities (such as `lab-admin` or members of `Domain Admins`).
- LAB-CLIENT01 remains **fully re-hardened**:
  - Domain admins and admin-only accounts are denied local and RDP logon.
  - Local Administrator–type accounts cannot be used for network access.
  - Help Desk cannot read or change local security policy directly; their view is intentionally limited.

In short, Milestone 11 demonstrates the ability to:

- Design a Help Desk OU and group structure.
- Delegate **just enough** permissions for Tier 1 account-level support.
- Verify that those permissions work from an actual workstation.
- Restore and confirm workstation hardening so that no lingering exceptions remain.
