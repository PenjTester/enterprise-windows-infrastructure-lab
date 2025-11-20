# 02 – Windows 10 Client: Domain Join and Identity Confirmation

**Date:** 11/19/2025  
**VM Created:** LAB-CLIENT01 (Windows 10 Pro)  
**Host PC Specs:** Ryzen 9 9900X, 32 GB DDR5 RAM, 2TB NVMe SSD  

---

##  Goal of This Milestone
Successfully configure a Windows 10 client to:

- Use correct internal networking (lab-network)
- Use DNS from the Domain Controller (LAB-DC01)
- Resolve the domain name (lab.local) via DNS
- Join Active Directory using LAB\Administrator
- Confirm domain membership and authentication

---

##  1. Virtual Machine Setup (VirtualBox)

| Setting | Value |
|---------|-------|
| VM Name | LAB-CLIENT01 |
| OS Type | Windows 10 Pro (64-bit) |
| RAM | 3 GB |
| CPU Cores | 2 vCPU |
| Disk | 70 GB (VDI) |
| Network Adapter | Internal Network (“lab-network”) |
| Adapter Type | Intel PRO/1000 MT Desktop (82540EM) |

<img width="854" height="530" alt="Screenshot 2025-11-19 140604" src="https://github.com/user-attachments/assets/33ac2682-4e12-4b6c-82a7-6d290d68270f" />

---

##  2. Windows Installation Completed

Windows 10 installed successfully. Reached default desktop and performed basic system setup.

<img width="1627" height="1065" alt="Screenshot 2025-11-19 143238" src="https://github.com/user-attachments/assets/960b30f5-4fed-466f-97eb-1353b615ef7f" />

---

##  3. Installed VirtualBox Guest Additions

Guest Additions installed to enable:

- Better mouse/keyboard control
- Higher resolution/full-screen mode
- Clipboard between host and VM


---

##  4. Configure Static IP and DNS

To enable domain communication, I manually set:

| Setting | Value |
|---------|--------|
| IP Address | 10.0.2.11 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 10.0.2.2 |
| Preferred DNS | 10.0.2.10 (DC) |
| Alternate DNS | not set |

<img width="399" height="457" alt="image" src="https://github.com/user-attachments/assets/7ed2b49c-d14d-4c0f-b125-0589f15a09c5" />

---

##  5. Network Testing (Connectivity + DNS Resolution)

Commands executed from Command Prompt:

    ping 10.0.2.10  
    ping lab.local  
    nslookup lab.local  

**Expected Results:**
- Domain Controller replies to ping
- `lab.local` resolves to 10.0.2.10
- nslookup confirms DNS is resolving via DC

<img width="535" height="568" alt="Screenshot 2025-11-19 184716" src="https://github.com/user-attachments/assets/da15c449-ef04-4540-a08f-7fade5693711" />

---

##  6. Joining Domain: lab.local

Windows Settings path used:

**System → About → Advanced system settings → Change**

Entered this domain name:

    lab.local

When prompted for credentials, I used:

    LAB\Administrator

Received confirmation:

<img width="261" height="148" alt="Screenshot 2025-11-19 154434" src="https://github.com/user-attachments/assets/865bffca-74ff-4968-847a-6e0af5b01fe1" />

---

##  7. Restart and Confirm Domain Login

On reboot, the login screen now displayed domain authentication options:

- LAB\Administrator  
- Other User (LAB domain context)

<img width="1623" height="1150" alt="image" src="https://github.com/user-attachments/assets/2eb8e8ce-a3fc-4436-83f1-0eb0ad00b6e8" />

After logging in using domain credentials, I confirmed domain identity via:

**Control Panel → System Properties → Computer Name tab**

<img width="410" height="464" alt="Screenshot 2025-11-19 155248" src="https://github.com/user-attachments/assets/943d7a31-0b08-40b4-a3e5-85048bb20584" />

---

##  Milestone Completion Summary

| Verification Check | Status |
|--------------------|--------|
| Ping Domain Controller success | ✔️ |
| DNS resolves lab.local correctly | ✔️ |
| Domain join completed | ✔️ |
| Login using LAB\Administrator | ✔️ |
| System now part of lab.local domain | ✔️ |

---

## 🧾 Outcome

LAB-CLIENT01 is now successfully joined to the lab.local domain and fully authenticated through Active Directory using DNS name resolution. The client and the domain controller can now communicate using domain-based identity.

---

##  Next Steps (Milestone 3)

| Task | Purpose |
|------|---------|
| Create domain users in Active Directory | Realistic identity management |
| Log in from LAB-CLIENT01 as new AD users | Confirm authentication and trust |
| Explore Group Policy (GPO) basics | Begin domain control concepts |
| Add documentation to GitHub | Continue building portfolio evidence |

---
