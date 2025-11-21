# 03 – Active Directory Identity Management

**Date Started:** 11/21/2025  
**Server:** LAB-DC01 (Windows Server 2019)  
**Client:** LAB-CLIENT01 (Windows 10 Pro)  

---

##  Goal of This Milestone

Design and implement a realistic Active Directory identity structure to support:

- Organizational Units (OUs) for users, computers, and admin accounts  
- Proper OU nesting and hierarchy  
- Future Group Policy targeting (GPO)  
- Preparation for domain user creation and group-based access control  

---

## 1️. Accessing Active Directory Users and Computers (ADUC)

Opened **Active Directory Users and Computers (ADUC)** on LAB-DC01 to begin structuring the domain.

 *Screenshot — ADUC open, showing lab.local before any OU organization*

<img width="825" height="181" alt="Screenshot 2025-11-21 125849" src="https://github.com/user-attachments/assets/b569d69c-6b49-418e-9977-440432daedb4" />


---

## 2️. Creating Organizational Units (All OUs Created — Flat Structure)

All required OUs were created at this stage, but were still at the same level (not yet nested into their parent OUs).

| OU Name Created |
|------------------|
| _IT_Admins |
| _Users |
| _Computers |
| _Groups |
| _Management |
| _HR |
| _Sales |
| _IT_Support |
| _Workstations |
| _Servers |

 *Screenshot — ADUC displaying all OUs created under lab.local, before nesting*

<img width="352" height="407" alt="Screenshot 2025-11-21 131434" src="https://github.com/user-attachments/assets/23edfaa7-326f-4cd5-86bc-7156f8cec3c7" />


---

## 3️. OU Hierarchy (After Nesting)

After temporarily disabling **Protect object from accidental deletion**, OUs were properly nested into the intended structure:

```
lab.local
│
├── _IT_Admins
│
├── _Users
│   ├── _Management
│   ├── _HR
│   ├── _Sales
│   └── _IT_Support
│
├── _Computers
│   ├── _Workstations
│   └── _Servers
│
└── _Groups
```

Protection was re-enabled once final placement was complete.

 *Screenshot — Final OU hierarchy correctly nested under lab.local*

<img width="350" height="234" alt="image" src="https://github.com/user-attachments/assets/2d97e102-16c9-46e5-85f2-cc9337f666dd" />


---

##  Note on Object Protection

Some OUs could not be moved initially due to **Protect object from accidental deletion** being enabled.  
Protection was temporarily disabled to allow nesting and re-enabled after structure was finalized.

---


## 4️. Domain User Account Creation

With the OU structure established, four domain user accounts were created in their appropriate departmental OUs. Each account follows a standard naming convention of **first initial + last name**, and was configured with:

- ✔️ User must change password at next login  
- ❌ Account is disabled (unchecked) 
- ❌ User cannot change password (unchecked)  
- ❌ Password never expires (unchecked)  

| Full Name     | Username | OU Location                         |
|---------------|----------|-------------------------------------|
| John Miller   | jmiller  | _Users → Management                 |
| Sarah Davis   | sdavis   | _Users → HR                         |
| Michael Lee   | mlee     | _Users → Sales                      |
| Emily Clark   | eclark   | _Users → IT_Support                 |

 *Screenshot — Example of “New Object – User” window showing account details (jmiller)*  

<img width="435" height="369" alt="Screenshot 2025-11-21 140253" src="https://github.com/user-attachments/assets/0e6c46ab-35c7-4ec8-a2f5-d0937a8203b6" />


 *Screenshot — Password configuration screen with “User must change password at next login” enabled*

<img width="434" height="375" alt="Screenshot 2025-11-21 140356" src="https://github.com/user-attachments/assets/52916d6f-2bb3-4b4b-b521-236b01a7c5ba" />


 *Screenshot — ADUC showing John Miller correctly placed in the Management OU*

<img width="745" height="243" alt="image" src="https://github.com/user-attachments/assets/23c52168-47aa-4898-a03b-33ee4d742722" />

> **Note:** All other user accounts (sdavis, mlee, eclark) were also verified to be correctly located in their respective departmental OUs, matching the intended AD structure.


---

## 5. Domain Logon Testing (LAB-CLIENT01)

To verify that newly created Active Directory users can authenticate against the domain, I logged into **LAB-CLIENT01** using a standard domain user account.

**Actions Performed:**
1. Booted LAB-CLIENT01 and selected **Other User** at the login screen.
2. Entered domain credentials for:  
   ```
   jmiller
   ```
3. Received expected message: *“The user’s password must be changed before signing in.”*  
   (This confirms the AD account policy is active and functioning.)
4. Successfully changed the password and logged into Windows using domain credentials.

 *Screenshot — showing login page for jmiller within the LAB domain*

<img width="1616" height="1128" alt="Screenshot 2025-11-21 144810" src="https://github.com/user-attachments/assets/5566c9cd-c8fd-4977-84dc-d131f01e4aab" />


---

## 6. Identity Confirmation (Client-Side Verification)

Once logged into LAB-CLIENT01 as **jmiller**, I confirmed domain-aware authentication using command-line identity checks.

**Commands executed from Command Prompt:**

```
whoami
echo %userdomain%
echo %username%
nslookup lab.local
systeminfo | findstr /B /C:"Domain"
```

**Verified Results:**
- `whoami` returned **lab\jmiller** confirming domain identity.
- `%userdomain%` showed **LAB**
- `%username%` showed **jmiller**
- `nslookup lab.local` successfully resolved via the domain controller (10.0.2.10)
- `systeminfo` confirmed the device is joined to **lab.local**

---


 *Screenshot — LAB-CLIENT01 Command Prompt showing identity validation using whoami and echo commands*  

<img width="444" height="148" alt="Screenshot 2025-11-21 150341" src="https://github.com/user-attachments/assets/2202ece9-e09a-42f4-9d4b-7ecca02aeaad" />

Confirms domain login as `lab\jmiller` and verifies user/domain context through `%userdomain%` and `%username%`.


---

 *Screenshot — LAB-CLIENT01 verifying domain membership using `systeminfo /findstr /B /C:"Domain"`* 

<img width="483" height="54" alt="Screenshot 2025-11-21 150801" src="https://github.com/user-attachments/assets/6ab49276-6010-4d61-963a-a0c6d8f069c7" />

Shows confirmation that the machine is joined to **lab.local** domain.


---

>  *The first-time password change prompt and “Preparing Windows” profile screens were completed successfully but not captured. Future user logins can include screenshots if needed for documentation completeness.*

---

### Status: ✔ Domain authentication successfully validated

| Verification Check | Result |
|--------------------|--------|
| User logs in with LAB domain credentials | ✔ |
| Password change policy enforced at first login | ✔ |
| DNS resolves domain correctly | ✔ |
| systeminfo confirms domain join | ✔ |
| Identity commands confirm AD-based login | ✔ |

---

**Next Step →**
Proceed to **Group Management and Access Control**  
We will create security groups in AD, assign users to their role-based OUs, and validate centralized management.


