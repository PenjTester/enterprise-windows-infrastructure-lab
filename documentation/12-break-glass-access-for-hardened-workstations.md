# Milestone 12 — Privileged Break-Glass Access for Hardened Workstations

## 12.1 Overview and Purpose

Milestone 12 introduces a controlled **break-glass administrative access model** for hardened workstations in the `lab.local` domain.  
Previous milestones enforced strict workstation hardening and tier separation, intentionally preventing privileged identities (such as `lab-admin` and Domain Admins) from logging on to member workstations like **LAB-CLIENT01**.

This created a realistic operational problem:

> **How can an administrator troubleshoot Group Policy, validate hardening, or perform emergency workstation recovery without weakening the security model?**

To solve this, a new workstation-scoped privileged account named **AuditAdmin** is created and deployed. The account provides:

- Local administrative rights on LAB-CLIENT01  
- No domain-wide privileged group membership  
- Full visibility into *computer-side* Group Policy (`gpresult /r`)  
- The ability to perform local troubleshooting and recovery without bypassing Domain Admin restrictions

This milestone models how real enterprises implement break-glass access for Tier 2 workstations while preserving least privilege.

---

## 12.2 Final State Architecture

By the end of Milestone 12:

- **AuditAdmin** exists under:  
  `lab.local → _IT_Admins`

- AuditAdmin:
  - Is *not* a Domain Admin  
  - Is *not* a member of any high-privilege domain group besides the default `Domain Users`  
  - Is a **local administrator** on LAB-CLIENT01 (and any future workstations in the Lab Workstations OU) via Group Policy Preferences  
  - Is explicitly allowed to log on locally (and via RDP if required) by the workstation GPOs  
  - Can run `gpupdate`, `gpresult /r`, Event Viewer, and other administrative tools on the workstation

- Existing hardening controls remain in place:
  - Domain Admins and `lab-admin` are still denied logon to LAB-CLIENT01  
  - HelpDesk users keep only Tier 1 capabilities  
  - No Domain Controller or Tier 0 configuration is relaxed

This preserves the tiering model:

- **Tier 0:** Domain Controllers  
- **Tier 1:** Domain-level administration  
- **Tier 2:** Workstations, hardened and isolated  
- **Break-Glass Admin:** `AuditAdmin`, a constrained local admin used only for workstation troubleshooting

---

## 12.3 Configuration Steps Performed

### 12.3A — Create the AuditAdmin Account

A new controlled identity was created using **Active Directory Users and Computers** on LAB-DC01:

- OU: `_IT_Admins`
- First name: `Audit`
- Last name: `Admin`
- User logon name: `auditadmin@lab.local`
- Password: strong, complex password with **Password never expires** enabled
- Initial group membership:
  - `Domain Users` (default only)

This ensured the account started with no domain-wide privileged rights.

**Screenshot Required — AuditAdmin group membership**

- Tool: Active Directory Users and Computers (LAB-DC01)  
- Path: `lab.local → _IT_Admins`  
- Action: Right-click **Audit Admin** → **Properties** → **Member Of** tab  
- Expected: Only `Domain Users` is listed (no Domain Admins, no Enterprise Admins, etc.)

---

### 12.3B — Add AuditAdmin to Local Administrators on LAB-CLIENT01 via GPP

To provide workstation-only administrative capability, AuditAdmin needed to be a **local administrator** on LAB-CLIENT01. This was implemented centrally using **Group Policy Preferences** in the existing **Advanced Workstation Hardening** GPO linked to the **Lab Workstations** OU.

**GPO:** `Advanced Workstation Hardening`  
**Scope:** `lab.local → _Computers → _Workstations → Lab Workstations`

**Configuration path:**

- `Computer Configuration → Preferences → Control Panel Settings → Local Users and Groups`

**Steps performed:**

1. In the GPO editor, under **Local Users and Groups**, a new **Local Group** item was created.
2. **Action:** `Update`
3. **Group name:** `Administrators (built-in)` (selected from the drop-down)
4. The options **Delete all member users** and **Delete all member groups** were left **unchecked** to avoid removing existing members.
5. Under **Members**, `LAB\auditadmin` was added:
   - Click **Add**
   - Enter `LAB\auditadmin`
   - Click **Check Names** to resolve to the domain account
   - Click **OK**

Result: Every workstation within the **Lab Workstations** OU automatically includes `LAB\auditadmin` as a member of the local **Administrators** group.

**Screenshot Required — Local Administrators membership on LAB-CLIENT01**

1. Log on to LAB-CLIENT01 as `LAB\auditadmin`.  
2. Open an elevated Command Prompt (Run as administrator).  
3. Run: `net localgroup administrators`  
4. Capture the command output showing `LAB\auditadmin` listed as a member of the local Administrators group.

---

### 12.3C — Allow AuditAdmin to Log On Locally (User Rights Assignment)

Previously, workstation hardening GPOs restricted local and RDP logon paths to enforce least privilege. For AuditAdmin to function as a break-glass account, it needed explicit rights to log on to LAB-CLIENT01.

Changes were made in the **Advanced Workstation Hardening** GPO:

**Path in GPO:**

- `Computer Configuration → Policies → Windows Settings → Security Settings → Local Policies → User Rights Assignment`

**Final configured entries:**

- **Allow log on locally**
  - `Administrators`
  - `LAB\auditadmin`

- **Allow log on through Remote Desktop Services** (if RDP access is desired)
  - `Administrators`
  - `LAB\auditadmin`

These settings ensure:

- Any local Administrators (including AuditAdmin, once added by GPP) can log on interactively.
- Domain Admins and `lab-admin` remain blocked by separate **Deny** policies defined in the **Privileged Access – Block Domain Admins On Workstations** GPO.

---

### 12.3D — Preserve Domain Admin Deny Controls

The existing **Privileged Access – Block Domain Admins On Workstations** GPO remains unchanged. Its key User Rights Assignment settings are:

- **Deny log on locally**
  - `Domain Admins`
  - `LAB\lab-admin`

- **Deny log on through Remote Desktop Services**
  - `Domain Admins`
  - `LAB\lab-admin`

This preserves the security design from earlier milestones:

- Domain Admin–level accounts cannot be used casually on workstations.  
- `lab-admin` remains a domain administrative identity restricted to server/DC contexts.  
- AuditAdmin becomes the *only* workstation-level troubleshooting identity with local admin access, and it does **not** hold Domain Admin membership.

---

### 12.3E — Validate Effective Policy on LAB-CLIENT01

After the configuration changes, the final step was to confirm that LAB-CLIENT01 was:

- Still receiving all expected workstation GPOs, and  
- Actually treating AuditAdmin as a local administrator with visibility into computer-side policy.

**Validation steps:**

1. Log on to **LAB-CLIENT01** as `LAB\auditadmin`.
2. Open an elevated Command Prompt (Run as administrator).
3. Run `gpupdate /force` and wait for both Computer and User policy updates to complete.
4. Run `gpresult /r` to view Resultant Set of Policy.

In the **COMPUTER SETTINGS** section of the `gpresult /r` output, the **Applied Group Policy Objects** list confirmed:

- `Advanced Workstation Hardening`
- `Baseline Workstation Policy`
- `Privileged Access – Block Domain Admins On Workstations`
- `Default Domain Policy`

This verifies that:

- The workstation is still fully hardened.  
- The break-glass account sees the same hardened policy stack as any other user.  
- All configuration is being delivered via GPO, not local one-off changes.

**Screenshot Required — gpresult /r (Computer Settings)**

1. On LAB-CLIENT01 as `LAB\auditadmin`, in an elevated Command Prompt, run: `gpresult /r`  
2. Scroll to the **COMPUTER SETTINGS** section.  
3. Capture the portion that shows **Applied Group Policy Objects** listing the workstation GPOs above.

---

## 12.4 Security Rationale

The design implemented in this milestone is intentionally conservative:

- **No Domain Admin elevation:**  
  AuditAdmin is not a member of `Domain Admins`, `Enterprise Admins`, or other domain-privileged groups. Its administrative scope is local to LAB-CLIENT01 (and any other workstations in the Lab Workstations OU).

- **Tier separation preserved:**  
  Domain-level administrative identities remain blocked from logging on to workstations, as enforced by the Privileged Access GPO. The introduction of AuditAdmin does not weaken this boundary.

- **Quick revocation:**  
  Because local admin membership is delivered via Group Policy Preferences, disabling or deleting the AuditAdmin account—or removing the Local Group preference item—will automatically remove its local admin rights on the next policy refresh.

- **Operational utility without excessive privilege:**  
  The account exists specifically to:
  - Run `gpresult /r` for computer policy troubleshooting.  
  - Inspect event logs and local configuration.  
  - Support patching, monitoring, and backup/recovery tests in later milestones.

This models a real-world break-glass scenario where a workstation-tier admin identity is available for emergency and diagnostic tasks without exposing the environment to full domain admin risk.

---

## 12.5 Operational Use Cases for AuditAdmin

Going forward, AuditAdmin will be used in scenarios such as:

- **Patch Management (Milestone 13):**
  - Verifying that Windows Update or third-party patching GPOs apply correctly.
  - Checking local update logs and scheduled tasks.

- **Monitoring & Event Review (Milestone 14):**
  - Inspecting Security, System, and Application logs on LAB-CLIENT01.
  - Confirming audit policy behavior configured in previous GPOs.

- **Backup & Recovery (Milestone 15):**
  - Testing restoration scenarios that require administrative access on the workstation.
  - Validating that recovery operations do not need Domain Admin credentials.

- **General GPO Troubleshooting:**
  - Running `gpupdate`, `gpresult`, and RSOP queries when HelpDesk Tier 1 encounters issues they cannot resolve with their limited permissions.

In all of these cases, the use of AuditAdmin is deliberate and controlled, consistent with a break-glass or Tier 2 troubleshooting account.

---

## 12.6 Summary of Milestone 12 Accomplishments

Milestone 12 completed the following:

- Designed and implemented a **break-glass workstation admin account** (`AuditAdmin`) located in the `_IT_Admins` OU.
- Ensured AuditAdmin has **no domain-wide privileged membership**, only local administrative rights delivered via Group Policy Preferences.
- Updated User Rights Assignment so AuditAdmin can log on locally (and via RDP if chosen) without weakening the existing “block Domain Admins on workstations” controls.
- Verified, using `gpresult /r`, that LAB-CLIENT01 continues to receive:
  - `Advanced Workstation Hardening`
  - `Baseline Workstation Policy`
  - `Privileged Access – Block Domain Admins On Workstations`
  - `Default Domain Policy`
- Established AuditAdmin as the standard account for workstation-level troubleshooting, patch validation, and future monitoring/backup milestones.

**Milestone 12 — Privileged Break-Glass Access for Hardened Workstations — is complete and ready for portfolio publication.**
