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
