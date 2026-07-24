# Active-Directory Homelab

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/c3755cce-da09-45a2-b1d3-9e2a2f2c7992" />
</p>

A hands-on Active Directory homelab designed to simulate a small enterprise Windows environment. Built on Proxmox VE, the lab covers Active Directory deployment, DNS, DHCP, Group Policy, certificate services, centralized logging with Splunk, endpoint monitoring with Sysmon, and controlled adversary simulations using Kali Linux

## Project Background

Active Directory (AD) is a centralized identity and access management solution used to manage users, computers, groups, authentication, authorization, and security policies within a Windows environment. Because Active Directory remains one of the most commonly targeted technologies during cyberattacks, understanding its architecture and administration is a valuable skill for system administrators, cybersecurity analysts, and penetration testers

This project was developed to provide a hands-on Active Directory lab environment using Proxmox Virtual Environment (VE). By leveraging Microsoft's free evaluation operating systems and Proxmox virtualization, anyone can build a realistic Windows domain in a safe, isolated homelab environment without requiring expensive hardware or software licenses

The lab provides hands-on experience with enterprise Active Directory operations, including identity management, authentication, authorization, DNS, DHCP, Group Policy, certificate services, and security configuration. Beyond administration, the environment is designed for adversary simulation and log analysis, allowing offensive techniques to be performed safely while analyzing security events through Sysmon and Splunk

## Network Architecture
- TBD

## Project Specifications

### Infrastructure Outline

**Hypervisor Platform**

* Proxmox VE
* 32 GB RAM

**Primary Management Device**
* macOS (MacBook Pro) accessing the environment via Proxmox Web UI

### Operating Systems

| System | Operating System 
|--------|-------------------|
| **Domain Controller** | Windows Server 2025 Evaluation
| **Client Workstation** | Windows 11 Pro Enterprise Evaluation
| **SIEM** | Ubuntu Linux
| **Attacker Machine** | Kali Linux


### Virtual Machine Configuration

#### VM 1: Domain Controller
* CPU: 2 vCPUs
* RAM: 5 GB
* Storage: 32 GB
* Network:
  * NIC 1: Connected to `vmbr1` (Isolated network)
* Operating System: Windows Server Evaluation
* Roles: Active Directory Domain Service, Active Directory Certificate Service, DNS, DHCP, File Server, Printer Server
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
  * NIC 1 (`ens18`): Connected to `vmbr0` (Management network) obtaining an IP address via DHCP (home router) to communicate with the Macbook Pro
  * NIC 2 (`ens19`): Connected to `vmbr1` (Isolated network) with a hardcoded static IP
* Role: Listens on TCP port `9997` for incoming sandbox telemetry and hosts the Splunk Web UI on port `8000`

#### VM 4: Kali Linux (Attacker Machine)
* Network:
  * NIC 1 connected to vmbr1 (Isolated network)
* Operating System: Kali Linux
* Role: Simulates an attacker on the internal network for red team exercises
* Key Tools: impacket, netexec, smbexec, kerbrute, john, nmap

### Network Design

* `vmbr0` (Management Network): Used for Proxmox management and to access the Splunk Web UI from the MacBook Pro
* `vmbr1` (Isolated Lab Network): Private virtual bridge with no physical uplink that hosts all Active Directory traffic between the Domain Controller, Windows 11 client, Splunk server and Kali
* IP Addressing: The Domain Controller and the Splunk server's `ens19` interface use static IP addresses on `vmbr1`. The Windows 11 client receives its IP address from the Domain Controller via DHCP, while the Splunk server's `ens18` interface obtains an IP address from the home network via DHCP on `vmbr0`
* DNS & Authentication: The Domain Controller provides Active Directory, DNS, DHCP, File & Printer, and AD CS services for all domain-joined systems
* Centralized Logging: Sysmon and Splunk Universal Forwarder send logs from the Domain Controller and Windows 11 client to the Splunk Enterprise server over the isolated network

# Red Team Exercises

The following attacks were simulated against the lab environment from the Kali Linux attacker machine. All exercises were performed in an isolated, controlled environment for educational purposes

## Recon

| Attack | Tool | Description |
|--------|------|-------------|
| Port Scanning | `nmap` | Identified open ports and running services on the Domain Controller (DC) without credentials |
| Password Spraying | `kerbrute` | Sprayed one password across all domain users to identify weak credentials without triggering account lockouts |

## Credential Attacks

| Attack | Tool | Description |
|--------|------|-------------|
| AS-REP Roasting | `impacket-GetNPUsers` | Extracted crackable hashes from accounts with Kerberos pre-authentication disabled |
| Offline Hash Cracking | `john` | Cracked extracted Kerberos hashes using the `rockyou.txt` wordlist |

## Privilege Escalation & Persistence

| Attack | Tool | Description |
|--------|------|-------------|
| Credential Dumping | `impacket-secretsdump` | Dumped all domain account hashes, including the `krbtgt` account, from the Domain Controller |
| Golden Ticket | `impacket-ticketer` | Forged a Kerberos TGT signed with the `krbtgt` AES256 key to impersonate Administrator indefinitely |

## Defensive Lessons Learned

| Attack | Defensive Lesson |
|--------|------------------|
| **Port Scanning** | Close unnecessary ports and limit exposed services with firewalls and network segmentation |
| **Password Spraying** | Use strong passwords, enable MFA, and monitor failed logins (Event ID `4625`) |
| **AS-REP Roasting** | Enable Kerberos pre-authentication for all accounts |
| **Offline Hash Cracking** | Use long, random passwords, especially for service accounts, to make cracking impractical |
| **Credential Dumping (DCSync)** | Restrict replication permissions and limit Domain Admin privileges |
| **Golden Ticket** | Rotate the krbtgt password twice after a domain compromise and monitor Kerberos ticket activity (Event IDs `4768` and `4769`) |

## Repository Structure

| Folder | Contents |
|--------|----------|
| `setup-domain-controller` | DC installation and AD DS promotion steps |
| `setup-client-vm` | Windows 11 domain join and configuration |
| `ad-cs-and-dhcp` | Certificate Services and DHCP configuration |
| `dns` | DNS setup and zone configuration |
| `file-and-print-server` | File sharing and print services |
| `security-groups-and-GPOs` | Group Policy and security group configuration |
| `powershell-automation` | Automation scripts for AD administration |
| `windows-event-logs` | Event log configuration and monitoring |
| `sysmon` | Sysmon installation and SwiftOnSecurity configuration |
| `splunk` | Splunk Universal Forwarder and Enterprise setup |
| `kali` | Red team attack walkthroughs |
