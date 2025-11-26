---

## 1. Creating Security Groups

To implement group-based identity and access control, Global Security Groups (GG-) were created to logically organize users by department. These groups serve as the foundation for role-based access management and will later be used for permissions and access control in future milestones.

**Groups Created (Global Security Type):**

| Department | Group Name | Assigned User |
|------------|------------|---------------|
| Management | GG-Management | John Miller |
| HR | GG-HR | Sarah Davis |
| Sales | GG-Sales | Michael Lee |
| IT Support | GG-IT-Support | Emily Clark |

*Screenshot — Creating a new Global Security Group (_Groups OU_)*  

<img width="433" height="371" alt="Screenshot 2025-11-24 144152" src="https://github.com/user-attachments/assets/9bcae27a-887a-45a7-95b5-fa74d590d97b" />

*Screenshot — Showing the four Global Security Groups created*

<img width="755" height="283" alt="Screenshot 2025-11-24 144344" src="https://github.com/user-attachments/assets/24bcf051-7f9e-4843-bc0b-c0e5b89ce4a9" />

---

## 2. Assigning Users to Security Groups

Each user account was assigned to its correct departmental group using  
**Group Properties → Members → Add** in ADUC.

*Screenshot — Showing GG-Management group membership with John Miller assigned:*

<img width="403" height="466" alt="Screenshot 2025-11-24 144803" src="https://github.com/user-attachments/assets/39b6873b-229c-43d4-bb65-a5f9fc3024f6" />

> _(Note: All other users were assigned to their correct departmental groups using the same process.)_

**Group Purpose Overview**

| Group Name | Purpose |
|------------|---------|
| GG-Management | Elevated access, executive-level resources |
| GG-HR | Access to employee records, HR documents, sensitive files |
| GG-Sales | Client communication, CRM, and sales resources |
| GG-IT-Support | Technical support tools, remote access, troubleshooting |

These Global Security Groups will be used in future milestones when defining  
**Role-Based Access Control (RBAC)** and setting **NTFS permissions**,  
**shared folder security**, and **Group Policy scoping**.

---


## 3. Group Membership Validation (Client-Side)

To confirm that security group assignments were successfully applied, I logged into **LAB-CLIENT01** using a domain user account and ran the following command in Command Prompt:

whoami /groups

This command lists all security groups that the current user is a member of.

### Verified Results:
✔ Output correctly displayed the assigned AD group (e.g., GG-Management for jmiller)  
✔ Confirmed that group assignment from Active Directory propagated to LAB-CLIENT01  
✔ Validated that domain security groups are now active and enforceable on domain-joined machines  

 *Screenshot — Command-line output showing correct group membership (e.g., LAB\GG-Management)*

<img width="1319" height="324" alt="Screenshot 2025-11-24 154141" src="https://github.com/user-attachments/assets/40770b97-6d16-4676-8147-279e39eaee7f" />


---

## 4. Creating Department Shared Folders (LAB-DC01)

On LAB-DC01, I created a central Shared directory with departmental subfolders.

**Location Path:** `C:\Shared\`

**Folders Created:**

| Folder Name | Purpose |
|-------------|---------|
| Management  | Files for Management department |
| HR          | Files for Human Resources |
| Sales       | Files for Sales department |
| IT_Support  | Files for IT Support |

*Screenshot — File Explorer showing all four folders under C:\Shared* 

<img width="799" height="386" alt="Screenshot 2025-11-24 161627" src="https://github.com/user-attachments/assets/9806503d-9456-4f2b-bd13-f7b1d7dfbde3" />

---

## 5. Configuring NTFS Permissions (Security Tab)

To enforce department-based access control, NTFS permissions were applied to each folder.

**Permissions for each folder (example: Sales):**

| Principal       | Permission     | Applies To |
|-----------------|----------------|------------|
| GG-Sales        | Modify         | This folder, subfolders, and files |
| SYSTEM          | Full control   | This folder, subfolders, and files |
| Administrators  | Full control   | This folder, subfolders, and files |

Unnecessary inherited entries (like `Users`) were removed.

*Screenshot — NTFS Security permissions for Sales folder*  

<img width="762" height="516" alt="Screenshot 2025-11-24 170920" src="https://github.com/user-attachments/assets/89139256-119c-413e-b250-07a756266849" />

(Same process for GG-HR → HR, GG-Management → Management, GG-IT-Support → IT-Support)

---

## 6. Validating Access from LAB-CLIENT01

Logged in to LAB-CLIENT01 using test users to verify folder access.

### 🔴 Unauthorized Access (Denied)

Example: Logged in as `jmiller` (Management) and attempted to access `\\LAB-DC01\Shared\Sales` → Access Denied.

*Screenshot — Access denied popup*

<img width="518" height="190" alt="Screenshot 2025-11-24 171228" src="https://github.com/user-attachments/assets/c3351b61-e13f-40b6-ab86-814994e6b3ec" />

---

### 🟢 Authorized Access (Allowed)

Still logged in as `jmiller` (Management), accessed `\\LAB-DC01\Shared\Management` → Access successful.

*Screenshot — Successful access to Management folder*

<img width="732" height="247" alt="Screenshot 2025-11-24 171336" src="https://github.com/user-attachments/assets/7fd93eb2-2d99-4a02-b4dd-93d0adc1cdc1" />

> Access confirmed based on security group membership.

---

## 7. Access Validation Summary

| User | Department | Allowed Folder | Denied Folders | Result |
|------|------------|----------------|----------------|--------|
| jmilller | Management | Management | HR, Sales, IT_Support | ✔️ |
| sdavis   | HR         | HR         | Management, Sales, IT_Support | ✔️ |
| mlee     | Sales      | Sales      | HR, Management, IT_Support | ✔️ |
| eclark   | IT_Support | IT_Support | HR, Sales, Management | ✔️ |

---

##  Milestone 04 Completion Summary

| Completed Task | Status |
|----------------|--------|
| Created department security groups | ✔️ |
| Assigned users to correct groups | ✔️ |
| Created departmental shared folders | ✔️ |
| Applied NTFS access control | ✔️ |
| Validated access restrictions | ✔️ |

---

##  Next Step → Milestone 05: File Sharing & Advanced NTFS Permissions

In the next milestone, we will:

- Configure SMB sharing on LAB-DC01  
- Compare Share vs NTFS permissions  
- Implement least-privilege access  
- Document effective permissions testing  

---

