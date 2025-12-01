# 07 – DNS Management & Domain Service Reliability


Date Started: 11/30/2025   
Server: LAB-DC01 (Windows Server 2019)   
Client: LAB-CLIENT01 (Windows 10 Pro)  

---

#  Goal of This Milestone

The purpose of Milestone 07 is to validate and strengthen DNS reliability inside the Active Directory domain, ensuring:

- Correct forward lookup zones  
- Correct reverse lookup zones (PTR)  
- Healthy SRV records for AD services  
- Successful DHCP → DNS dynamic updates  
- Forward and reverse resolution  
- Secure dynamic updates  
- Forwarders removed for isolated lab networks  
- DNS scavenging for long-term cleanliness  

DNS is one of the most critical services in Active Directory. When DNS is healthy, AD is healthy.

---

#  Background

Active Directory depends entirely on DNS. If DNS breaks, everything breaks:
- Domain join  
- Kerberos authentication  
- Group Policy  
- SYSVOL replication  
- LDAP queries  
- DHCP integration  

Because earlier milestones established a healthy foundation, DNS was already functioning correctly.  
This milestone focused on **validating** DNS health and **cleaning up configuration**, not repairing damage.

---

# 1. Verify Forward Lookup Zones (LAB-DC01)

We inspected forward lookup zones on the domain controller. Expected zones:

- lab.local  
- _msdcs.lab.local  

These contain:
- Host A records  
- SOA/NS records  
- SRV records  
- Domain controller locator records  

### *Screenshot – DNS Manager showing Forward Lookup Zones*  
<img width="838" height="264" alt="Screenshot 2025-11-30 125817" src="https://github.com/user-attachments/assets/8f0a4407-80f5-4b16-86c5-8cc9a3eed2e2" />

### *Screenshot – lab.local zone contents*  
<img width="1019" height="367" alt="Screenshot 2025-11-30 130112" src="https://github.com/user-attachments/assets/28cd5856-c3ef-407b-9f3e-cce3a95ea577" />

---
 
# 2. Verify Reverse Lookup Zone (PTR Zone)

A reverse lookup zone existed from earlier milestones:

2.0.10.in-addr.arpa

It was configured for:
- Active Directory Integration  
- Secure-only dynamic updates  

PTR records confirmed:
- LAB-DC01 → 10.0.2.10  
- LAB-CLIENT01 → 10.0.2.100 (DHCP)  
- A stale PTR from early testing (10.0.2.11), safe to ignore  

### *Screenshot – Reverse Lookup Zone contents*  
<img width="1015" height="307" alt="Screenshot 2025-11-30 131403" src="https://github.com/user-attachments/assets/1622fb82-e445-42f7-b99f-247866d5446e" />

---

# 3. Validate SRV Records for AD Services

Under _msdcs.lab.local, the folders:

- dc  
- domains  
- gc  
- pdc  

contained correct SRV records for:
- LDAP  
- Kerberos  
- PDC Emulator  
- Global Catalog  

### *Screenshot – SRV records under _msdcs.lab.local → dc*  
<img width="1144" height="419" alt="image" src="https://github.com/user-attachments/assets/9e75778a-0cca-4ab2-8ba9-3ec349ca8f28" />

---

# 4. DNS Cleanup, Forwarders, and Scavenging

## 4.1 – Forwarder Removal

DNS previously listed an unreachable forwarder:

192.168.1.1

Since the lab uses an isolated VirtualBox Internal Network, this forwarder caused delays and unnecessary resolution attempts.  
We removed it.

### *Screenshot – Forwarders tab after cleanup* 
<img width="396" height="463" alt="Screenshot 2025-11-30 133942" src="https://github.com/user-attachments/assets/a789cf60-e40a-4169-91bf-7484b9c417b4" />

---

## 4.2 – Root Hints Validation

Root hints were confirmed present and left unchanged.

---

## 4.3 – DNS Aging & Scavenging (Enabled)

Scavenging was enabled to demonstrate DNS hygiene:

- No-refresh: 7 days  
- Refresh: 7 days  

Applied to existing AD-integrated zones.

### *Screenshot – Aging & Scavenging configuration*  
<img width="380" height="383" alt="Screenshot 2025-11-30 134356" src="https://github.com/user-attachments/assets/9d928672-8b46-4159-9a62-a1f350e10130" />

---

# 5. DNS Functional Testing (Completed Earlier)

These tests were completed during the workflow:

### Forward Lookup  
nslookup lab-dc01.lab.local  
→ Resolved 10.0.2.10

<img width="478" height="282" alt="Screenshot 2025-11-30 132541" src="https://github.com/user-attachments/assets/f76b6bc1-c440-400d-9c3a-c76c576dad85" />

---

### Reverse Lookup  
nslookup 10.0.2.10  
→ Returned LAB-DC01.lab.local

<img width="417" height="122" alt="Screenshot 2025-11-30 132659" src="https://github.com/user-attachments/assets/55bc4c34-c105-4910-9cc4-a9e9dd44cfad" />

---

### DHCP Dynamic Updates  
ipconfig /release  
ipconfig /renew  
→ Client reacquired 10.0.2.100 and PTR was updated

<img width="636" height="410" alt="Screenshot 2025-11-30 133321" src="https://github.com/user-attachments/assets/d47e3ccc-5509-455e-9ecd-6077bca37287" />

---

### DNS Cache Validation  
ipconfig /displaydns  
→ Displayed A and SRV records confirming DNS health

<img width="864" height="391" alt="Screenshot 2025-11-30 133201" src="https://github.com/user-attachments/assets/4279e2d2-121c-4681-b4b2-fb3f6a552455" />

---

#  FINAL RESULTS – MILESTONE 07 COMPLETE

DNS is now:

- Fully AD-integrated  
- Secure with dynamic updates  
- Clean and predictable  
- Correct in forward & reverse zones  
- Integrated with DHCP  
- Loaded with correct SRV records  
- Free of unreachable forwarders  
- Configured with scavenging  
- Able to resolve forward & reverse lookups reliably  

This milestone confirms that the foundational name services for the entire domain are functioning at an enterprise level.

---

#  NEXT

## Milestone 08 – Group Policy Management & Enterprise Configuration

Next milestone will include:

- Creating and linking GPOs  
- Understanding GPO inheritance and precedence  
- Configuring security policies  
- Testing user and computer GPOs  
- Enterprise-style configuration management  

---
