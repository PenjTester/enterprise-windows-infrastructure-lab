#  Lab Setup — Domain Controller Build (Milestone 1)

**Date:** 11/18/2025  
**Machine:** VirtualBox — Windows Server 2019 (GUI)  
**Host PC:** Ryzen 9 9900X, 32GB RAM, 2TB NVMe SSD  

---

##  Objectives of this Milestone

Set up the **core identity infrastructure** of a virtual enterprise network:

- Install Windows Server 2019  
- Promote it to Domain Controller  
- Create Active Directory Domain Services (AD DS)  
- Configure DNS for the new domain  
- Assign permanent (static) IP address  

---

##  Virtual Machine Configuration (VirtualBox)

OS: Windows Server 2019 (GUI)  
RAM: 2.5 GB  
CPU Cores: 2  
Disk Size: 60 GB  
Network Adapter: NAT  
Guest Additions Installed: Yes  

---

##  Domain Controller Setup

| Component         | Value              |
|------------------|--------------------|
| Computer Name     | LAB-DC01           |
| Domain Name       | lab.local          |
| Administrator Login | LAB\\Administrator |
| DNS Role          | Installed automatically with AD DS |

---

##  Static IP Configuration (Required for Domain Controllers)

| Setting        | Value         |
|---------------|---------------|
| IP Address     | 10.0.2.10     |
| Subnet Mask    | 255.255.255.0 |
| Default Gateway | 10.0.2.2     |
| Preferred DNS  | 127.0.0.1     |
| Alternate DNS  | 8.8.8.8       |

---

##  Connectivity Validation

| Test Command           | Result |
|------------------------|--------|
| ping 10.0.2.10         | Success |
| ping 10.0.2.2          | Success |
| ping 8.8.8.8           | Success |
| ping microsoft.com     | Success (DNS working) |

---

##  Milestone Outcome

By the end of this session, I successfully:

- Built a Windows Server 2019 VM  
- Promoted it to a Domain Controller  
- Created the Active Directory `lab.local` domain  
- Assigned and verified a static IP  
- Verified DNS name resolution and internet connectivity  
- Validated successful domain login using **LAB\\Administrator**  

---

##  Next Steps 

- Create Windows 10 Client VM  
- Join client to `lab.local` domain  
- Create domain users and test authentication  
- Apply Group Policy to enforce restrictions/settings  
- Explore file sharing and permission management  
