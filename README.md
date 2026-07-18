# Active-Directory Homelab

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/c3755cce-da09-45a2-b1d3-9e2a2f2c7992" />
</p>

Virtualized Windows domain environment that provides hands-on experience with Active Directory administration, domain services, authentication, Group Policy, and enterprise security operations using Proxmox virtualization
## Project Background

Active Directory (AD) is a centralized identity and access management solution used to manage users, computers, groups, authentication, authorization, and security policies within a Windows environment. Because Active Directory remains one of the most commonly targeted technologies during cyberattacks, understanding its architecture and administration is a valuable skill for system administrators, cybersecurity analysts, and penetration testers.

This project was developed to provide a hands-on Active Directory lab environment using Proxmox Virtual Environment (VE). By leveraging Microsoft's free evaluation operating systems and Proxmox virtualization, users can build a realistic Windows domain in a safe, isolated homelab environment without requiring expensive hardware or software licenses.

The lab focuses on foundational Active Directory concepts including domain controllers, domain-joined workstations, DNS, Group Policy Objects (GPOs), user and group administration, authentication, and Windows domain management. The environment is designed to support practical learning and experimentation with Active Directory administration while serving as a foundation for future cybersecurity, system administration, blue team, and red team projects

## Network Architecture
- TBH

## Project Specifications

### Infrastructure Outline

**Hypervisor Platform**

* Proxmox VE
* 32 GB RAM

**Primary Management Device**
* macOS (MacBook Pro) accessing the environment via Proxmox Web UI

### Operating Systems

**Domain Controller**

* Windows Server 2025 Evaluation
* Active Directory Domain Services (AD DS) role installed

**Client Workstation**

* Windows 11 Pro Enterprise Evaluation
* Joined to the Active Directory domain

**SIEM**

* Ubuntu Linux running Splunk Enterprise Server

### Virtual Machine Configuration

#### VM 1: Domain Controller
* CPU: 2 vCPUs
* RAM: 5 GB
* Storage: 32 GB
* Network:
  * NIC 1: Connected to `vmbr1` (Isolated network)
* Operating System: Windows Server Evaluation
* Log agents: Sysmon (SwiftOnSecurity config) & Splunk Universal Forwarder

#### VM 2: Domain-Joined Client
* CPU: 2 vCPUs
* RAM: 5 GB
* Storage: 32 GB
* Network:
  * NIC 1: Connected to `vmbr1` (Isolated network)
* Operating System: Windows 11 Pro Enterprise Evaluation
* Log agents: Sysmon (SwiftOnSecurity config) & Splunk Universal Forwarder

#### VM 3: Splunk Enterprise Server
* Operating System: Ubuntu Linux
* Network (Dual-NIC Architecture):
  * NIC 1 (`ens18`): Connected to `vmbr0` (Management network) obtaining an IP address via DHCP to communicate with the physical Macbook Pro
  * NIC 2 (`ens19`): Connected to `vmbr1` (Isolated network) with a hardcoded static IP

* Role: Listens on TCP port `9997` for incoming sandbox telemetry and hosts the Splunk Web UI on port `8000`

### Network Design

* `vmbr0` (Management Network): Used for Proxmox management and to access the Splunk Web UI from the MacBook Pro
* `vmbr1` (Isolated Lab Network): Private virtual bridge with no physical uplink that hosts all Active Directory traffic between the Domain Controller, Windows 11 client, and Splunk server
* IP Addressing: The Domain Controller and the Splunk server's `ens19` interface use static IP addresses on `vmbr1`. The Windows 11 client receives its IP address from the Domain Controller via DHCP, while the Splunk server's `ens18` interface obtains an IP address from the home network via DHCP
* DNS & Authentication: The Domain Controller provides Active Directory, DNS, and DHCP services for all domain-joined systems
* Centralized Logging: Sysmon and Splunk Universal Forwarder send logs from the Domain Controller and Windows 11 client to the Splunk Enterprise server over the isolated network
