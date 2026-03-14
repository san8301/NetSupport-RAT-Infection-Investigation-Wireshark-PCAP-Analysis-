# NetSupport RAT Infection Investigation (Wireshark PCAP Analysis)

## Overview
This project documents a network forensic investigation of suspected malware activity within an enterprise environment. The analysis was conducted using a packet capture (PCAP) obtained from Malware Traffic Analysis.

The goal of the investigation was to identify the infected internal host communicating with known NetSupport Manager RAT infrastructure, determine the associated Active Directory user account, and document indicators of compromise.

---

## Scenario

During routine monitoring in a Security Operations Center (SOC), the SIEM generated multiple alerts indicating NetSupport Manager RAT traffic communicating with the external IP address:

45.131.214.85

The activity was observed over TCP port 443 beginning on:

February 28, 2026 at 19:55 UTC

A packet capture of the affected network segment was obtained for investigation.

---

## Network Environment

Network Range: 10.2.28.0/24  
Domain: easyas123.tech  
Active Directory Environment: EASYAS123  
Domain Controller: 10.2.28.2  
Gateway: 10.2.28.1  
Broadcast Address: 10.2.28.255  

---

## Tools Used

- Wireshark – Network traffic analysis
- Malware Traffic Analysis – PCAP dataset

---

# Investigation Process

## 1. Identify Communication With Malicious Infrastructure

The packet capture was filtered for traffic involving the malicious IP address.

Wireshark filter used:

ip.addr == 45.131.214.85

This revealed repeated connections originating from the internal host:

10.2.28.88

The traffic occurred over TCP port 443, consistent with encrypted command-and-control communication.

---

## 2. Identify the Infected Host

Further inspection confirmed that the host at **10.2.28.88** initiated persistent outbound connections to the malicious server.

This behavior is consistent with malware beaconing activity commonly seen in remote access trojan (RAT) infections.

Screenshot:

![Infected Host Identification](infected_host_identification.png) 

---

## 3. Identify the MAC Address

By inspecting Ethernet frame details associated with the infected IP address, the MAC address of the compromised device was identified as:

MAC Address: **00:19:d1:b2:4d:ad**

This information can assist incident responders in identifying the physical device on the network.

Screenshot:

![Infected Host MAC Address](infected_host_mac_address.png) 

---

## 4. Identify the Hostname

Using the following Wireshark filter:

nbns

NetBIOS Name Service traffic was analyzed. This revealed the hostname associated with the infected machine:

Hostname: **DESKTOP-TEYQ2NR**

Screenshot:

![Hostname Discovery](hostname_discover_nbns.png) 

---

## 5. Identify the Active Directory Username

Kerberos authentication traffic was analyzed to determine the user account associated with the infected system.

Wireshark filter used:

kerberos

Inspection of the Kerberos **cname** field revealed the username:

Username: **brolf**

Screenshot:

![Username Identification](kerberos_username_identification.png)

---

## 6. Determine the Full Name Using SAMR Traffic

To identify the full name associated with the user account, a broader search was performed across the packet capture.

Steps used in Wireshark:

1. Removed the display filters.
2. Pressed **Ctrl + F** to open the packet search tool.
3. Selected the following search options:
   - Search In: Packet Details
   - Search Type: String
   - Search Value: Rolf
   - Case Sensitive: Enabled

The search returned a packet with the following characteristics:

Source IP: **10.2.28.2** (Domain Controller)  
Destination IP: **10.2.28.88**  
Protocol: **SAMR**  
Info: **QueryUserInfo response**

Expanding the SAMR protocol fields within the packet revealed the user account information:

Full Name: **Becka Rolf**

Screenshot:

![Full Name Identification](samr_fullname_indentification.png)

---

# Indicators of Compromise (IOCs)

Malicious IP Address: 45.131.214.85  
Internal Infected Host: 10.2.28.88  
Hostname: DESKTOP-TEYQ2NR  
MAC Address: 00:19:d1:b2:4d:ad  
Username: brolf  
Full Name: Becka Rolf  
Malware: NetSupport Manager RAT  
Protocol: TCP  
Port: 443  
First Observed Activity: 2026-02-28 19:55 UTC

---

# Conclusion

Analysis of the packet capture revealed persistent outbound connections from the internal host **10.2.28.88** to known NetSupport Manager RAT infrastructure at **45.131.214.85** over TCP port 443.

By correlating network traffic with authentication protocols and directory service communications, the compromised host and associated user account were identified.

Key findings include:

Infected Host IP: 10.2.28.88  
Hostname: DESKTOP-TEYQ2NR  
MAC Address: 00:19:d1:b2:4d:ad  
Username: brolf  
Full Name: Becka Rolf  

This investigation demonstrates how SOC analysts can combine packet analysis, authentication traffic inspection, and directory service queries to identify compromised hosts and document security incidents.

---

# Skills Demonstrated

- Network traffic analysis
- Packet capture investigation
- Wireshark filtering and packet search techniques
- Malware command-and-control detection
- Active Directory user correlation
- SAMR protocol analysis
- NetBIOS hostname discovery
- Incident documentation and reporting
