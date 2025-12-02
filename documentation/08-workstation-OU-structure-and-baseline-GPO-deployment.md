# 08 – Workstation OU Structure & Baseline GPO Deployment  
**Date:** 12/02/2025  
**Server:** LAB-DC01 (Windows Server 2019)  
**Client:** LAB-CLIENT01 (Windows 10 Pro)

---

# Goal of This Milestone
This milestone establishes a scalable, enterprise-style workstation management structure using Active Directory Organizational Units (OUs) and Group Policy Objects (GPOs).  

It introduces clean workstation separation, standardized baseline settings, and controlled configuration management through Group Policy.

**This milestone includes:**  
- Building a dedicated **OU hierarchy** for domain workstations  
- Creating and linking a **Baseline Workstation Policy** GPO  
- Configuring initial workstation policy settings  
- Forcing and validating GPO delivery on LAB-CLIENT01  
- Confirming correct inheritance, linking, and OU placement  

These steps align with real-world AD/GPO workflows and showcase practical administrative competency.

---

# Phase 1 — OU Structure Configuration

## 1.1 Create a new OU for workstations
Create the **Lab Workstations** OU inside the `_Workstations` OU to establish a proper workstation hierarchy.

*Screenshot: Lab Workstations OU created*  
<img width="276" height="179" alt="image" src="https://github.com/user-attachments/assets/286f3b61-2c1f-4bf7-80b3-2f0b72ea16b9" />

---

## 1.2 Move LAB-CLIENT01 into the Lab Workstations OU
Placing the computer in this OU ensures that the correct GPOs apply only to managed domain workstations.

*Screenshot: LAB-CLIENT01 inside the Lab Workstations OU*  
<img width="1187" height="214" alt="image" src="https://github.com/user-attachments/assets/570a92a2-d6cf-4088-8a75-92ca4afd836f" />

---

# Phase 2 — Create & Link the Baseline Workstation GPO

## 2.1 Create the Baseline Workstation Policy GPO
This GPO will hold workstation-level configuration settings.

*Screenshot: Newly created GPO in Group Policy Management*  
<img width="382" height="173" alt="Screenshot 2025-12-02 153732" src="https://github.com/user-attachments/assets/1467fd29-7aa0-4388-8b9d-33cc775d8220" />

---

## 2.2 Link the GPO to the Lab Workstations OU
Linking ensures policy application is scoped correctly by OU.

*Screenshot: GPO linked under Lab Workstations*  
<img width="1036" height="255" alt="image" src="https://github.com/user-attachments/assets/5e2fa032-dc4e-4f91-8110-3076a4f1adcd" />

---

# Phase 3 — Configure Initial Baseline Settings

This milestone applies two foundational workstation policies: one at the **computer** level and one at the **user** level. These reflect the actual settings configured during GPO creation on your system.

---

## 3.1 Disable Windows Error Reporting (Computer-Side)

### How to navigate to the policy:
1. On LAB-DC01, open **Group Policy Management**.
2. Expand:  
   `Domains → lab.local → _Computers → _Workstations → Lab Workstations`
3. Right-click **Baseline Workstation Policy** → **Edit**.
4. In the left pane, navigate to:

```
Computer Configuration
  → Policies
    → Administrative Templates
      → System
        → Internet Communication Management
          → Internet Communication settings
```

### Setting:
**Turn off Windows Error Reporting** → **Enabled**

*Screenshot: Windows Error Reporting disabled*  
<img width="663" height="461" alt="Screenshot 2025-12-02 165738" src="https://github.com/user-attachments/assets/c794134e-72b1-4e39-90fd-381357c50d7e" />

---

## 3.2 Do not keep a history of recently opened documents (User-Side)

### How to navigate to the policy

1. In the **same** GPO editor window, in the left pane expand:

   ```
   User Configuration
     → Policies
       → Administrative Templates
         → Start Menu and Taskbar
   ```

2. In the right pane, locate:

   **Do not keep a history of recently opened documents**

### Setting

- **Do not keep a history of recently opened documents** → **Enabled**

This prevents Windows from retaining a history of recently opened documents for the user, improving privacy on managed workstations.

*Screenshot: “Do not keep a history of recently opened documents” enabled*  
<img width="674" height="114" alt="Screenshot 2025-12-02 170252" src="https://github.com/user-attachments/assets/d0fa3998-162a-4861-b6f6-5ea5ea88af10" />



---

# Phase 4 — Apply & Validate GPO Delivery

## 4.1 Apply the updated GPO on LAB-CLIENT01

On the workstation (as Administrator):

```
gpupdate /force
```

*Screenshot: gpupdate completed successfully*  
<img width="536" height="113" alt="Screenshot 2025-12-02 155318" src="https://github.com/user-attachments/assets/b559e245-8271-4c7a-9998-75ba0637901e" />

---

## 4.2 Validate policy application using gpresult  
Run:

```
gpresult /r
```

Expected findings:

- **Applied Group Policy Objects:**  
  - Baseline Workstation Policy  
  - Default Domain Policy  

- Confirmed OU Path:  
  `OU=Lab Workstations,OU=_Workstations,OU=_Computers,DC=lab,DC=local`

*Screenshot: gpresult showing Baseline Workstation Policy applied*  
<img width="752" height="1125" alt="Screenshot 2025-12-02 161942" src="https://github.com/user-attachments/assets/7fbccc54-a6b7-4ed4-ba2f-97437b0795a4" />

---

## 4.3 Verify GPO link and inheritance
In Group Policy Management:

`lab.local → _Computers → _Workstations → Lab Workstations`

Confirm:

- GPO link exists  
- Link Enabled = Yes  

*Screenshot: GPO link and inheritance verification*  
<img width="1036" height="255" alt="Screenshot 2025-12-02 164544" src="https://github.com/user-attachments/assets/c483c4ff-dfe4-45cc-ae8b-00bd96a0f123" />

---

# Phase 5 — Troubleshooting Notes
During this milestone, several real-world troubleshooting challenges were resolved:

- Ensuring the workstation OU was correctly placed inside the `_Workstations` hierarchy  
- Identifying accidental OU protection settings preventing movement  
- Ensuring the GPO was linked at the correct OU level  
- Verifying computer-side GPO evaluation  
- Using gpupdate and gpresult to confirm policy flow  
- Understanding propagation timing inherent to Active Directory  

These reflect realistic enterprise GPO diagnosis workflows.

---

# Validation Summary

| Validation Item | Result |
|-----------------|--------|
| LAB-CLIENT01 correctly placed in OU structure | ✔ |
| Baseline Workstation Policy created | ✔ |
| GPO linked to correct OU | ✔ |
| Baseline settings configured | ✔ |
| gpupdate successfully applied settings | ✔ |
| gpresult confirms GPO applied | ✔ |

---

# What’s Next
Proceed to **Milestone 09 – Advanced Workstation GPO Hardening**, which will cover:

- Security option policies  
- Login banners  
- Firewall & Defender management  
- Account lockout standards  
- Windows Update configuration  
- Additional baseline workstation policies  

This milestone builds upon the foundational OU and GPO structure created here.

---


