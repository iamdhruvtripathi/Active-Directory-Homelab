# Active-Directory Homelab
Virtualized Windows domain environment that provides hands-on experience with Active Directory administration, domain services, authentication, Group Policy, and enterprise security operations using Proxmox virtualization
## Project Background

Active Directory (AD) is a centralized identity and access management solution used to manage users, computers, groups, authentication, authorization, and security policies within a Windows environment. Because Active Directory remains one of the most commonly targeted technologies during cyberattacks, understanding its architecture and administration is a valuable skill for system administrators, cybersecurity analysts, and penetration testers.

This project was developed to provide a hands-on Active Directory lab environment using Proxmox Virtual Environment (VE). By leveraging Microsoft's free evaluation operating systems and Proxmox virtualization, users can build a realistic Windows domain in a safe, isolated homelab environment without requiring expensive hardware or software licenses.

The lab focuses on foundational Active Directory concepts including domain controllers, domain-joined workstations, DNS, Group Policy Objects (GPOs), user and group administration, authentication, and Windows domain management. The environment is designed to support practical learning and experimentation with Active Directory administration while serving as a foundation for future cybersecurity, system administration, blue team, and red team projects

## Project Specifications

### Infrastructure Requirements

**Hypervisor Platform**

* Proxmox VE
* 32 GB RAM
* Internet connectivity for updates and software installation

### Operating Systems

**Domain Controller**

* Windows Server 2022 Evaluation or Windows Server 2025 Evaluation
* Active Directory Domain Services (AD DS) role installed

**Client Workstation**

* Windows 10 Enterprise Evaluation or Windows 11 Enterprise Evaluation
* Joined to the Active Directory domain

### Virtual Machine Configuration

#### VM 1: Domain Controller
* CPU: 2 vCPUs
* RAM: 4 GB
* Storage: 40–50 GB
* Network: Connected to `vmbr1` (Fully Isolated Network)
* Operating System: Windows Server Evaluation

#### VM 2: Domain-Joined Client
* CPU: 2 vCPUs
* RAM: 4–6 GB
* Storage: 40–50 GB
* Network: Connected to `vmbr1` (Fully Isolated Network)
* Operating System: Windows 10/11 Enterprise Evaluation

### Network Design

* Isolated Virtual Bridge (`vmbr1`): A dedicated virtual switch inside Proxmox with no physical port attachment, isolating all AD traffic, DNS, and future simulation/security tools
* Static IP Addressing: Both VMs will use static IPs within a dedicated private subnet assigned to the `vmbr1` interfaces
* DNS Services: Primary DNS for all domain assets is hosted on the Domain Controller via `vmbr1`
* Controlled Internet Access: Internet connectivity for updates and software downloads is restricted to a secondary interface (`vmbr0`) on the Domain Controller, ensuring the client remains fully isolated
