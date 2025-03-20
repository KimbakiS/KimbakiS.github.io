---
layout: default
---

# Cybersecurity Home Lab - Detection and Monitoring

A cybersecurity home lab featuring pfSense for network management, Active Directory with Windows Server, Security Onion for traffic monitoring, and pentesting tools for analysis and exploitation.


---


## Network Diagram

![Cybersecurity Home Lab](/images/Cybersecurity Home Lab.PNG)


---


## Project Components


- **PfSense Server:** Multi-interface router/firewall that manages network traffic and subnet isolation for enhanced security.
  
- **Windows Server Domain Controller:** Centralized management for the Active Directory environment, including user authentication, authorization, and directory services.
  
- *Workstations (Windows 11 Clients):** Two client machines running Windows 10, configured as part of the Active Directory domain for testing and monitoring purposes.
  
- **Security Onion Server:** A network security monitoring tool that captures and analyzes traffic (via a span port) for intrusion detection, log management, and real-time monitoring.
  
- **Analyst VM (Ubuntu):** A Linux-based virtual machine used for accessing the Security Onion web portal and conducting deeper analysis on network traffic and security logs.
  
- **Pentesting VM (Kali Linux):** A penetration testing environment used for manual and automated vulnerability scanning, reconnaissance, exploitation, and gaining access to the pfSense portal for testing security defenses.


---


## Pfsense Configuration

### Interface Assignment:
  1. WAN: **em0 — `192.168.106.130/24` (auto)**
  2. LAN: **em1 — `192.168.1.1/24` — enable DHCP**
  3. OP1: **em2 — `192.168.2.1`**
  4. OP2: **em3 — `192.168.3.1`**
  5. OP3: **em4 — no IP (this will be a span port)**
  6. OP4: **em5 — `192.168.4.1`**

### Web Portal Configuration:
- Primary DNS: `8.8.8.8`   Secondary DNS: `4.4.4.4`
- Interface Names: LAN=`Kali`   OPT1=`VictimNetwork`   OPT2=`SecOnion`   OPT3=`SpanPort`
- Add `SPANPORT` as the span port for `VictimNetwork` so the latter's traffic will pass through to `SecOnon`
- Firewall: accept traffic on all interfaces and all ports -- _intentional security misconfiguration_


---


## Security Onion Configuration

### Set Up
- **Hardware:**   Disk = 200GB   RAM = 8GB   Adapters = Vmnet4 (connects to Pfsense), Vmnet5 (connects to span port), NAT (general internet connection)
- Regular NAT adapter set as  the management NIC -- `ens160`
- Vmnet5 (span port interface) set as the monitoring port
- Static DHCP: `192.168.106.10/24`
- Domain Name Pfsense's LAN network: `home.arpa`
- Set up credentials for Admin:   Email = `admin@middleearth.com`
- Create a firewall rule to allow Analyst VM to accesss Security Onion's web portal at `192.168.106.131`

### Testing & Troubleshooting
- Check if traffic is being captured from the Windows Server on the span port and sent to the monitoring interface `ens224` (Vmnet5) -- ``sudo tcpdump -i ens224``
  
- Legacy command to add an anlyst machine with `sudo so-allow` was not longer valid, so found an alternative: `sudo so-firewall includehost analyst 192.168.106.133`


---


## Analyst VM -- Ubuntu

### Tools


---


## Pentester VM -- Kali Linux

### Purposes

  1. Accesses the Pfsense web portal at `http://192.168.1.1`
  2. Conducts attack simulations on the target network to test defenses

### Troubleshooting

- Had an issue with updating the machine because of a missing key; retrieved and added the key from Kali's site via the command: `wget -q -O -https://archive.kali.org/archive-key.asc | apt-key add`


--- 


## Victim Network -- Active Directory Environment

### Windows Domain Controller

#### Set Up

- PC Name: `FRODO-DC`
- Add Active Directory Domain Services role and promote the server to a domain controller
- Add new forest with domain name `MIDDLEEARTH.local`
- Add Active Directory Certificate Services role and configure to add a CA
- Add two new sers to the environment: `MERRY` and `PIPPIN`

#### (Mis)Configurations

- Set the certificate validity period to 99 years -- _intentional security misconfiguration_
- Turn off all aspects of Windows Firewall, including Real-Time Protection -- _intentional misconfiguration_
- Network Adapter (static): IP = `192.168.2.10`   Mask = `255.255.255.0`   Gateway (Pfsense) = `192.168.2.1`

### Windows 11 Workstations

#### Set Up

- PC Names: `MERRY-WS` and `PIPPIN-WS`
- Network Adapter:   IP = `192.168.2.21`,`192.168.2.31`   Gateway = `192.168.2.1`   DNS Server (DC) = `192.168.2.10`
- Join AD domain `MIDDLEEARTH.local` -- _need AD administrator credentials_
