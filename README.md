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

**Purpose:** Centralized authentication and domain management

* CPU: 2 vCPUs
* RAM: 4 GB
* Storage: 40–50 GB
* Network: Connected to shared virtual network
* Operating System: Windows Server Evaluation

#### VM 2: Domain-Joined Client

**Purpose:** Workstation for user authentication and domain interaction

* CPU: 2 vCPUs
* RAM: 4–6 GB
* Storage: 40–50 GB
* Network: Connected to same network as Domain Controller
* Operating System: Windows 10/11 Enterprise Evaluation

### Network Design

* Shared virtual bridge network within Proxmox
* Static IP addressing for both virtual machines
* DNS services hosted on the Domain Controller
* Internet access enabled for updates and software downloads

### Project Objectives

**Input**

* Windows Server installation
* Windows Client installation
* Active Directory Domain Services deployment
* Domain configuration
* Client domain enrollment

**Output**

* Functional Active Directory domain
* Domain Controller managing centralized authentication
* Domain-joined Windows workstation
* User and group administration capabilities
* Group Policy management
* Foundational enterprise lab environment

### Learning Outcomes

* Deploy and configure Active Directory Domain Services
* Create and manage organizational units (OUs)
* Manage domain users and groups
* Configure Group Policy Objects (GPOs)
* Understand Windows authentication workflows
* Build a foundation for cybersecurity and system administration labs
