<div align="center">

# Active Directory Homelab

**A hands-on enterprise Windows environment for AD administration, detection engineering and adversary simulation**

<img src="https://img.shields.io/badge/Proxmox_VE-E57000?style=for-the-badge&logo=proxmox&logoColor=white" alt="Proxmox"/>
<img src="https://img.shields.io/badge/Windows_Server-0078D4?style=for-the-badge&logo=windows&logoColor=white" alt="Windows Server"/>
<img src="https://img.shields.io/badge/Splunk-000000?style=for-the-badge&logo=splunk&logoColor=white" alt="Splunk"/>
<img src="https://img.shields.io/badge/Sysmon-5C2D91?style=for-the-badge&logo=windowsterminal&logoColor=white" alt="Sysmon"/>
<img src="https://img.shields.io/badge/Kali_Linux-557C94?style=for-the-badge&logo=kalilinux&logoColor=white" alt="Kali Linux"/>
<img src="https://img.shields.io/badge/PowerShell-5391FE?style=for-the-badge&logo=powershell&logoColor=white" alt="PowerShell"/>

<br/><br/>

<img width="90%" alt="Active Directory Homelab banner" src="https://github.com/user-attachments/assets/c3755cce-da09-45a2-b1d3-9e2a2f2c7992" />

</div>

---

## 📖 Overview

This lab simulates a small enterprise Windows domain on Proxmox VE. It covers Active Directory deployment, DNS, DHCP, Group Policy, certificate services, centralized logging with Splunk, endpoint monitoring with Sysmon, and controlled adversary simulations from Kali Linux

| 🔵 Blue Team | 🔴 Red Team |
|---|---|
| AD DS, DNS, DHCP, AD CS, GPOs | Password spraying, AS-REP roasting, Kerberoasting |
| Sysmon + Splunk telemetry pipeline | Lateral movement over RDP and SMB |
| Event log analysis and detection lessons | Credential dumping and Golden Ticket persistence |

---

## 🧭 Table of Contents

- [🎯 Project Background](#-project-background)
- [🗺️ Network Architecture](#️-network-architecture)
- [📁 Repository Structure](#-repository-structure)
- [🖥️ Infrastructure](#️-infrastructure)
- [🌐 Network Design](#-network-design)
- [🔴 Red Team Exercises](#-red-team-exercises)
- [🛡️ Defensive Lessons Learned](#️-defensive-lessons-learned)

---

## 🎯 Project Background

Active Directory (AD) is Microsoft's centralized identity and access management solution for users, computers, groups, authentication, authorization and security policy in a Windows environment. Because it remains one of the most commonly targeted technologies in real-world attacks, understanding its architecture and administration is a valuable skill for sysadmins, SOC analysts and penetration testers.

Using Microsoft's free evaluation operating systems and Proxmox virtualization, anyone can build a realistic Windows domain in a safe, isolated homelab without expensive hardware or licenses.

Beyond administration, the environment is built for adversary simulation and log analysis: offensive techniques are performed safely while the resulting security events are analyzed through Sysmon and Splunk.

---

## 🗺️ Network Architecture

<p align="center">
  <img width="90%" alt="Network architecture diagram" src="https://github.com/user-attachments/assets/38975126-8050-4303-8ae6-668eeff7178d" />
</p>

---

## 📁 Repository Structure

| | Folder | Contents |
|:---:|---|---|
| 🏛️ | [`setup-domain-controller`](./setup-domain-controller) | DC installation and AD DS promotion |
| 💻 | [`setup-client-vm`](./setup-client-vm) | Windows 11 domain join and configuration |
| 📜 | [`ad-cs-and-dhcp`](./ad-cs-and-dhcp) | Certificate Services and DHCP configuration |
| 🌐 | [`dns`](./dns) | DNS setup and zone configuration |
| 🖨️ | [`file-and-print-server`](./file-and-print-server) | File sharing and print services |
| 👥 | [`security-groups-and-GPOs`](./security-groups-and-GPOs) | Group Policy and security group configuration |
| ⚙️ | [`powershell-automation`](./powershell-automation) | Automation scripts for AD administration |
| 📋 | [`windows-event-logs`](./windows-event-logs) | Event log configuration and monitoring |
| 🔍 | [`sysmon`](./sysmon) | Sysmon install with the SwiftOnSecurity config |
| 📊 | [`splunk`](./splunk) | Splunk Universal Forwarder and Enterprise setup |
| 🐉 | [`kali`](./kali) | Red team attack walkthroughs |

---

## 🖥️ Infrastructure

| Component | Details |
|---|---|
| Hypervisor | Proxmox VE · 32 GB RAM |
| Management device | MacBook Pro (macOS) via the Proxmox Web UI |

> [!NOTE]
> Most of this lab was built on Windows Server 2025 Evaluation. For the `kali` attack demos, the Domain Controller was switched to Windows Server 2022 Evaluation, because Kerberoasting worked there but could not be reproduced on 2025. The AD functionality used here is substantially the same in both versions, so the lab's objectives are unaffected. If rebuilding from scratch, I'd use Server 2022 throughout.

### Operating Systems

| System | Operating System |
|---|---|
| 🏛️ **Domain Controller** | Windows Server 2022 Evaluation |
| 💻 **Client Workstation** | Windows 11 Pro Enterprise Evaluation |
| 📊 **SIEM** | Ubuntu Linux |
| 🐉 **Attacker Machine** | Kali Linux |

### Virtual Machines

| | 🏛️ Domain Controller | 💻 Domain-Joined Client | 📊 Splunk Server | 🐉 Kali Linux |
|---|---|---|---|---|
| **CPU / RAM / Disk** | 2 vCPU · 5 GB · 32 GB | 2 vCPU · 5 GB · 32 GB | — | — |
| **Network** | `vmbr1` | `vmbr1` | `ens18` → `vmbr0` (DHCP)<br/>`ens19` → `vmbr1` (static) | `vmbr1` |
| **OS** | Windows Server Evaluation | Windows 11 Pro Enterprise Eval | Ubuntu Linux | Kali Linux |
| **Role** | AD DS, AD CS, DNS, DHCP, File & Print | Domain workstation | Receives telemetry on TCP `9997`, Web UI on `8000` | Internal attacker |
| **Agents / Tools** | Sysmon + Splunk UF | Sysmon + Splunk UF | Splunk Enterprise | impacket, netexec, smbexec, kerbrute, john, nmap |

---

## 🌐 Network Design

| Network | Purpose |
|---|---|
| **`vmbr0`** · Management | Proxmox management and Splunk Web UI access from the MacBook Pro |
| **`vmbr1`** · Isolated lab | Private bridge with no physical uplink, carrying all AD traffic between the DC, client, Splunk and Kali |

- **IP addressing:** the DC and Splunk's `ens19` use static IPs on `vmbr1`. The Windows 11 client gets its IP from the DC over DHCP, and Splunk's `ens18` gets one from the home router on `vmbr0`.
- **DNS & authentication:** the DC provides AD, DNS, DHCP, File & Print and AD CS for all domain-joined systems.
- **Centralized logging:** Sysmon and the Splunk Universal Forwarder ship logs from the DC and client to Splunk Enterprise over the isolated network.

---

## 🔴 Red Team Exercises

> [!IMPORTANT]
> All attacks were run from Kali against this isolated, self-owned lab for educational and defensive purposes only.

### 🔑 Credential Attacks

| Attack | Tool | Description |
|---|---|---|
| **Password Spraying** | `kerbrute` | Tested common passwords across multiple domain accounts |
| **AS-REP Roasting** | `impacket-GetNPUsers` + John | Retrieved and cracked an AS-REP hash from an account without Kerberos pre-auth |
| **Kerberoasting** | Impacket + John | Retrieved and cracked a service ticket for an account with an SPN |

### 🚪 Initial Access & Lateral Movement

| Technique | Tool | Description |
|---|---|---|
| **Valid Accounts / RDP** | `xfreerdp` | Used recovered credentials to RDP into a workstation |
| **SMB Authentication** | `NetExec` / Impacket | Used recovered credentials to authenticate to the DC over SMB |
| **SMB Share Enumeration** | `NetExec` | Enumerated accessible SMB shares and permissions |

### 👑 Privilege Escalation & Persistence

| Attack | Tool | Description |
|---|---|---|
| **Credential Dumping** | `impacket-secretsdump` | Extracted domain credential hashes, including `krbtgt` |
| **Golden Ticket** | `impacket-ticketer` | Forged a Kerberos TGT using the `krbtgt` key |
| **Remote Command Execution** | `impacket-smbexec` | Used the forged ticket to run commands remotely on the DC |

---

## 🛡️ Defensive Lessons Learned

| Attack | Key Telemetry | Defensive Lesson |
|---|---|---|
| **Password Spraying** | `4771` `4768` | Enforce strong passwords and monitor repeated authentication failures |
| **AS-REP Roasting** | `4768` | Enable Kerberos pre-auth and monitor `Pre_Authentication_Type = 0` |
| **Kerberoasting** | `4769` | Prefer AES, use strong service passwords, monitor RC4 (`0x17`) TGS requests |
| **Valid Accounts / RDP** | `4624` | Restrict RDP and monitor unexpected Logon Type `10` |
| **SMB Enumeration** | `4624` `5140` | Restrict SMB access and monitor access to sensitive shares |
| **Credential Dumping** | Privileged activity | Protect Domain Controllers and restrict privileged access |
| **Golden Ticket** | `4769` `4672` `7045` | Monitor anomalous Kerberos activity; reset `krbtgt` twice after compromise |
| **Remote Command Execution** | `7045` `4672` | Investigate unexpected service creation and privileged remote execution |

---

<div align="center">

⭐ **If this lab helped you, consider starring the repo!** ⭐

</div>
