# 10 – Privilege Access Management and Administrative Account Hardening

## Overview

This milestone enforces enterprise-style privileged access boundaries in the `lab.local` domain.

**Goals:**

- Normal domain users (e.g., `jmiller`, `sdavis`, `mlee`, `eclark`) can log on to `LAB-CLIENT01` and work normally.
- Privileged accounts (`lab-admin`, `Domain Admins`, `Administrator`) are blocked from logging on to `LAB-CLIENT01` (locally and via RDP).
- Local Administrator–type accounts cannot be used for network logon.
- All controls are implemented via a GPO linked to the workstation OU.

This demonstrates real systems/network administrator hardening practices, not just default AD setup.

---

## 10.1 – Verify Administrative OU Structure and Privileged Account

On `LAB-DC01`, open **Active Directory Users and Computers** and confirm:

- The following OUs exist at the domain root:
  - `_IT_Admins`
  - `_Users`
  - `_Groups`
  - `_Computers`

- Under `_IT_Admins`, confirm your dedicated privileged account exists:
  - `lab-admin`

> **Screenshot Placeholder M10-01:** ADUC top-level OU structure showing `_IT_Admins`, `_Users`, `_Groups`, `_Computers`.  
> **Screenshot Placeholder M10-02:** ADUC focused on `_IT_Admins` with `lab-admin` visible.

---

## 10.2 – Confirm Workstation OU Placement

Still in ADUC, confirm the workstation is in the correct OU path:

- Expand:
  - `lab.local`
    - `_Computers`
      - `_Workstations`
        - `Lab Workstations`

Verify that:

- `LAB-CLIENT01` is located under `Lab Workstations`.

> **Screenshot Placeholder M10-03:** ADUC showing `LAB-CLIENT01` inside `_Computers → _Workstations → Lab Workstations`.

This OU is the scope where workstation-targeted hardening GPOs will apply.

---

## 10.3 – Create the Workstation Privileged Access GPO

On `LAB-DC01`:

1. Open **Group Policy Management**.
2. In the left pane, navigate to:
   - `Forest: lab.local`
     - `Domains`
       - `lab.local`
         - `_Computers`
           - `_Workstations`
             - `Lab Workstations`
3. Right-click `Lab Workstations` and select:
   - “Create a GPO in this domain, and Link it here…”
4. Name the new GPO (linked to `Lab Workstations`):

   `Privileged Access – Block Domain Admin Logon`

> **Screenshot Placeholder M10-04:** GPMC showing the GPO “Privileged Access – Block Domain Admin Logon” linked to `Lab Workstations`.

---

## 10.4 – Deny Local Logon for Privileged Accounts

Edit the new GPO:

1. In the GPO editor, navigate to:
   - Computer Configuration  
     → Policies  
     → Windows Settings  
     → Security Settings  
     → Local Policies  
     → User Rights Assignment

2. In the right pane, locate and open:
   - “Deny log on locally”

3. Enable and configure “Deny log on locally”:
   - Select “Define these policy settings”.
   - Add:
     - `Domain Admins`
     - `lab-admin`
     - `Administrator` (optional but recommended)

4. Apply and close the dialog.

> **Screenshot Placeholder M10-05:** GPO editor showing “Deny log on locally” populated with `Domain Admins`, `lab-admin`, and (optionally) `Administrator`.

This prevents privileged accounts from logging on interactively at the workstation console.

---

## 10.5 – Deny Remote Desktop Logon for Privileged Accounts

In the same GPO (`Privileged Access – Block Domain Admin Logon`):

1. Still under:
   - Computer Configuration  
     → Policies  
     → Windows Settings  
     → Security Settings  
     → Local Policies  
     → User Rights Assignment

2. Locate and open:
   - “Deny log on through Remote Desktop Services”

3. Configure “Deny log on through Remote Desktop Services”:
   - Select “Define these policy settings”.
   - Add:
     - `Domain Admins`
     - `lab-admin`
     - `Administrator` (optional but recommended)

> **Screenshot Placeholder M10-06:** GPO editor showing “Deny log on through Remote Desktop Services” with the same privileged identities listed.

This prevents privileged accounts from using RDP to log into workstations.

---

## 10.6 – Block Local Administrator from Network Access

Still in the same GPO and same node (**User Rights Assignment**):

1. Locate and open:
   - “Deny access to this computer from the network”

2. Configure “Deny access to this computer from the network”:
   - Select “Define these policy settings”.
   - Add:
     - `Local account`
     - `Local account and member of Administrators group`

> **Screenshot Placeholder M10-07:** GPO editor showing “Deny access to this computer from the network” populated with `Local account` and `Local account and member of Administrators group`.

This reduces lateral movement risk by preventing local Administrator–type accounts from being used for network logon.

---

## 10.7 – Apply the GPO and Refresh Policies

On `LAB-DC01`:

1. Open an elevated command prompt.
2. Run:
   - `gpupdate /force`

On `LAB-CLIENT01`:

1. Restart the workstation to ensure computer-side policy is fully applied.

No screenshots are strictly required here, but you can capture the gpupdate output if desired.

---

## 10.8 – Validate Normal User Functionality

After the reboot of `LAB-CLIENT01`:

1. At the logon screen, log in as a **normal domain user**, for example:
   - `jmiller`
2. Confirm that:
   - The desktop loads normally.
   - Network access is functional (e.g., open Command Prompt and ping `LAB-DC01` and `lab.local`).
   - Access to the user’s departmental share still works via the UNC path you configured in earlier milestones (for example, the department-specific share for jmiller).

> **Screenshot Placeholder M10-08:** `LAB-CLIENT01` logged in as `jmiller`, with a command prompt showing successful pings to `LAB-DC01` and `lab.local`.  
> **Screenshot Placeholder M10-09:** File Explorer on `LAB-CLIENT01` showing successful access to the appropriate departmental share for that user.

This confirms that standard user workflows are not broken by the new restrictions.

---

## 10.9 – Validate That Privileged Accounts Are Blocked on Workstations

On `LAB-CLIENT01`, at the Windows logon screen:

1. Click “Other user”.
2. Attempt to log on using:
   - `lab-admin`
3. Enter the correct password.
4. Observe the logon result.

Expected behavior:

- The logon attempt is denied with a message such as:
  - “The sign-in method you’re trying to use isn’t allowed.”
  - Or equivalent text indicating that the account is not permitted to sign in here.

Optionally, repeat the test with:

- `Administrator` (domain or local, depending on configuration)
- Any other privileged identity added to the deny lists above.

> **Screenshot Placeholder M10-10:** `LAB-CLIENT01` showing the logon failure screen for `lab-admin`, with a message indicating that the sign-in is not allowed.

Because the restriction is enforced locally via “Deny log on locally” and “Deny log on through Remote Desktop Services”, the workstation can block these logons before any full domain authentication occurs. As a result, the domain controller may not log a 4625 failed logon event for these specific denied workstation logons. This is expected given this hardening approach; behavioral evidence at the client (failed logon with “not allowed” message) is the primary validation.

---

## 10.10 – Milestone Completion Summary

At the end of Milestone 10:

- `_IT_Admins` exists and contains your dedicated privileged admin account (`lab-admin`).
- `LAB-CLIENT01` resides in `_Computers → _Workstations → Lab Workstations`, where workstation-hardening GPOs are scoped.
- GPO `Privileged Access – Block Domain Admin Logon` is linked to `Lab Workstations` and configured to:
  - Deny local logon for:
    - `Domain Admins`
    - `lab-admin`
    - `Administrator` (if configured)
  - Deny Remote Desktop logon for the same privileged identities.
  - Deny network access from:
    - `Local account`
    - `Local account and member of Administrators group`
- Normal domain users (e.g., `jmiller`) can still log in to `LAB-CLIENT01`, access the network, and reach their departmental shares.
- Privileged accounts are prevented from interactive use on workstations, aligning the lab with real-world privileged access management practices.

This milestone completes the first step of privileged access hardening and prepares the environment for subsequent server and domain controller–focused hardening in future milestones.
