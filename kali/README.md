# Kali Linux
- Now that we have built our own AD environment, we will assume the role of an attacker against our own domain. I began with `AS-REP Roasting`, `Kerberoasting`, `Password Spraying` and `Golden Tickets` attacks

### Setting up Kali Linux
- We are now going to setup Kali Linux and grab the lightweight Kali Linux Installer ISO from the official Kali website. For the sake of brevity, I am not going to show the process of allocating RAM, CPU, etc. to this VM as I have already done that with the other 3 VMs I setup and the VM Configuration can be read on the main README of this project

- Here I am on the first screen and I proceeded through the installation process
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/f4eb33be-748b-47fe-85f1-ad933e7404fa" />
</p>

- Since this VM is on `vmbr1`, which is an isolated sandbox, I will have to manually configure the IP address and so I set this one to `10.10.10.20`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/781c3d3e-b654-49ef-8ddc-aa9147032edc" />
</p>

- For DNS, it is also `10.10.10.10`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/b9a813fc-9235-44c4-8619-bd538a443257" />
</p>

- The Domain name is listed below

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/89dc108b-b0c0-4397-8a6e-3b2078fbf807" />
</p>

- I went through all the options and finally arrived here

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/7fde5f49-5571-4192-9763-24a182e43ffd" />
</p>

- Note that when I assigned the IP address to the Kali VM during installation, it did not set it for some reason and so I had to run these two commands to set the IP address again

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/f4ae88f9-89ba-4739-893f-550df67f48bf" />
</p>

- As we can see, it can talk to the DC correctly :D

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/862a7db6-c6e3-43b8-a7fe-38df2a4726bc" />
</p>

## What is `AS-REP Roasting`
- Normally, when a user authenticates with Kerberos, the client sends an `AS-REQ` containing the client's username and pre-authentication data. This pre-authentication data is typically an encrypted timestamp (`PA-ENC-TIMESTAMP`), which is encrypted using a key derived from the user's password. The Key Distribution Center (KDC) uses the same password-derived key to decrypt the timestamp and verify that the client knows the correct password. If the timestamp is successfully decrypted and valid, the KDC returns an `AS-REP` containing a Ticket Granting Ticket (TGT), which is encrypted with the `krbtgt` account's key, and a copy of the session key encrypted with the user's password-derived key

- In an `AS-REP` roasting attack, the target account has Kerberos pre-authentication disabled. Because the KDC no longer requires the client to prove knowledge of the user's password before issuing a ticket, anyone can request an `AS-REP` for that account. The `AS-REP` still contains the TGT and the session key encrypted with the user's password-derived key. An attacker can capture this response and perform an offline brute force or dictionary attack against the encrypted session key. For each password guess, the attacker derives the corresponding key and attempts to decrypt the encrypted session key. If the decryption succeeds, the password guess is correct

- Now, two things to make this go smoothly. I will set `Alisha`'s password to `welcome1` and check `Do not require Kerberos pre-authentication` but first I had to change the password settings so I could actually set her password to that value and then do `gpupdate /force` to make sure it applied everyone in the domain 

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/782b6402-66d5-4635-a7bd-064d81f37e9f" />
</p>

- Now, I changed her password to `welcome1`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/32a47e5e-65dd-4fc4-af99-599d61039bbb" />
</p>

- and I checked the box

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/4794de3a-f8e8-4554-9c6b-33f59488b303" />
</p>

- Now, it is time for the fun part and I will be running Impacket's `GetNPUsers` tool to request the ticket hash
```
impacket-GetNPUsers homelab.local/alisha -dc-ip 10.10.10.10 -no-pass
```
- BOOM!, we get the result we wanted. Now that we have her encrypted data (`AS-REP hash`), what we can do is use `John the Ripper` for offline password cracking to get her real password (which should be `welcome1`). The idea here is that `John the Ripper` tries different passwords to get the correct password derived key and if we are able to decrypt the session key and we get valid data in the `AS-REP` hash it means we have correctly guessed the user's password. Below, I saved the output into a file named `hash.txt`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/cd114155-4715-40b7-9da8-72302b21bf84" />
</p>

- I made some changes to the `.txt` file so when I feed it, `John` can actually understand it

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/1a7a19e1-9ca0-4d38-9c3a-a76fd34f686e" />
</p>

- `John` ended up cracking the password in less than 1 second

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/b46f479f-3933-4689-9939-81b9549f465b" />
</p>

- Now, as a detective we also need to see what happened. If we take a look into Splunk searching for the Event ID `4768`, we can see indeed it was our Kali linux machine (IP address `10.10.10.20`) that requested the TGT. The account name was indeed `Alisha`. Furthermore, we can see `Pre_Authentication_Type` is set to `0` which means that Kerberos pre-authentication was not required. Standard Kerberos authentication uses type `2` (`PA-ENC-TIMESTAMP`). The `Ticket_Encryption_Type` is set to `0x17` which corresponds to `RC4-HMAC`. This specific hashing structure is easier to crack offline compared to AES-128 (`0x11`) or AES-256 (`0x12`)
  
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/f55d592f-4fab-4ffe-a225-07d93a36b746" />
</p>

## Defense Best Practices for `AS-REP Roasting`
- To defend against AS-REP Roasting, ensure Kerberos pre-authentication is enabled for all accounts unless there is a documented exception. Enforce strong, unique passwords so that even if an AS-REP is captured, offline cracking is difficult. Prefer AES encryption over legacy RC4 where possible and regularly audit accounts for weak Kerberos configurations. Finally, monitor Event ID `4768` for unusual requests, especially those with `Pre_Authentication_Type = 0`, and investigate suspicious source IPs

## RDP
- Now before I move onto the next topic, as an attacker I now know `Alisha`'s username and password, so we can login remotely into her account using RDP but first I needed to enable `Remote Desktop` on the computer
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/8baa088d-5c62-4d83-b796-7a880b00d939" />
</p>

- Also, I had to run the command below and then restart the computer
```
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
```

- I also tested to see if the port was listening over the network from Kali before I ran `xfreerdp` and it said it was open
```
nc -zvw 3 10.10.10.50 3389
```

- Lastly, one of the most important steps was giving all domain users permission to login via RDP, otherwise, the connection would fail
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/25c35de5-e234-4bd4-81d0-276b6bbc8909" />
</p>

- Now, we can remotely access her computer via RDP from Kali and BOOM!, we are in
```
xfreerdp /v:10.10.10.10 /d:homelab.local /u:alisha /p:"welcome1" /cert:ignore /dynamic-resolution
```
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/0826fec8-37f2-42b2-be9b-211f24c509f7" />
</p>

- In Splunk, I made a neat table that gave us the information
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/1ac0a50d-808a-41eb-9525-a0babc2a260a" />
</p>

- We can clearly see that my Kali Linux VM (`10.10.10.20`) had Alisha's credentials and used them to log into the VM via RDP, making it appear as though Alisha had logged in herself. The corresponding Windows logon event shows Logon Type `10` (RemoteInteractive), which indicates that the account authenticated through a remote interactive session such as RDP. This represents initial access through the use of valid credentials over RDP

## What is Kerberoasting?
- Normally, when a user or service authenticates to a Kerberos-protected service, the client first obtains a Ticket Granting Ticket (TGT) from the Key Distribution Center (KDC). When the client wants to access a particular service, it sends a `TGS-REQ` to the KDC requesting a service ticket for that service's Service Principal Name (SPN). The KDC returns a `TGS-REP` containing a service ticket. This ticket includes information encrypted with the password-derived key of the account associated with the SPN, allowing the service to verify and authenticate the request

- In a Kerberoasting attack, an attacker who has a valid domain account requests service tickets for accounts that have registered SPNs. The KDC normally allows authenticated users to request these tickets as part of normal Kerberos functionality. The attacker can capture the returned service ticket and extract the encrypted portion associated with the service account. Because this portion is encrypted using a key derived from the service account's password, the attacker can perform an offline brute-force or dictionary attack against it. For each password guess, the attacker derives the corresponding key and attempts to verify or decrypt the captured ticket data. If the guess produces the expected result, the service account's password has been recovered

- First thing to do was create a target user and assign an SPN using PowerShell
```
New-ADUser -Name "svc_sql" -AccountPassword(ConvertTo-SecureString "password123" -AsPlainText -Force) -Enabled $true
```
```
Set-ADUser -Identity "svc_sql" -ServicePrincipalNames @{Add="MSSQLSvc/db01.homelab.local:1433"}
```
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/f7ea35c7-0c21-479d-a5b9-620a33effa3f" />
</p>

- Now what we can do is use Impacket to request the service ticket and retrieve the Kerberoastable hash and we can also use the user that we already have (`Alisha`'s account)
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/0066dd91-c8bd-4c10-881b-0636e07f70a8" />
</p>

- When we feed this into John, it instantly cracks the password, giving us the credentials for the account associated with the SPN

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/042219dc-2684-4d6a-855a-8eeffe9ba548" />
</p>

- Now, in Splunk, we can detect this attack. The first part is detecting the Initial Ticket Request in Splunk
```
index=main EventCode=4769
```
- We can see the evidence of Kerberoasting here captured in this log
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/2d6ab10d-4c18-4625-bbba-645ccc71e7f5" />
</p>

- Looking at the Splunk log, we can start piecing together what happened. Searching for Event ID `4769`, we can see that the account `Alisha` requested a TGS ticket for the `svc_sql` service account from the Kali Linux machine (`10.10.10.20`). The `Ticket_Encryption_Type` is `0x17`, which corresponds to RC4-HMAC and is commonly associated with Kerberoasting because it can be cracked offline more easily than AES-128 (`0x11`) or AES-256 (`0x12`). The `Failure_Code` is `0x0`, confirming that the ticket request was successful

- We can turn this into a neat table as well
```
index=main EventCode=4769 Service_Name!="*$" Ticket_Encryption_Type="0x17" Failure_Code="0x0"
| eval src_ip = replace(Client_Address, "::ffff:", "")
| stats count earliest(_time) as first_seen latest(_time) as last_seen values(Service_Name) as targets by Account_Name, src_ip, Ticket_Encryption_Type
| eval Encryption = "RC4-HMAC (Downgrade/Roast)"
| convert ctime(first_seen) ctime(last_seen)
| table first_seen, last_seen, Account_Name, src_ip, targets, Encryption, count
```
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/f1130516-e116-4ac5-82e7-7cea747e6693" />
</p>

- Now, an attacker isn't going to stop after cracking the password. The next step would most likely be to use the recovered credentials to authenticate against `DC01`, in this case over SMB. As we can see below, the authentication was successful

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/560f2dc6-0bbc-4aa6-b0cc-08262b33fa65" />
</p>

- To confirm this activity from the defender's perspective, we can then turn to Splunk and look for the corresponding successful logon event. By searching for Event ID `4624`, we can identify the `svc_sql` account authenticating over the network from the Kali machine
```
index=main EventCode=4624 (Target_User_Name="svc_sql" OR Account_Name="svc_sql")
| eval src_ip = replace(IpAddress, "::ffff:", "")
| eval LogonTypeDesc = case(
    Logon_Type==2, "Interactive (Console)",
    Logon_Type==3, "Network (SMB/RPC)",
    Logon_Type==10, "Remote Interactive (RDP)",
    1=1, "Other (".Logon_Type.")"
  )
| table _time, Target_User_Name, src_ip, Workstation_Name, Logon_Type, LogonTypeDesc, Authentication_Package, Process_Name
```
- This query searches for successful `4624` logons involving the `svc_sql` account. It cleans up the source IP, identifies the logon type (with Type `3` being a network/SMB logon), and displays the key details such as the source IP, workstation, authentication method, and process used

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/b0c49aa7-856c-48b0-bb66-a29d337a3f5a" />
</p>

- With the `svc_sql` credentials now confirmed, the next step is to see what resources the account can access over SMB. From the attacker’s perspective, this means enumerating the available network shares and their permissions. We can then switch back to Splunk to detect this activity and identify which shares the compromised account accessed

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/87871497-f372-410c-9f14-6b47bf61c419" />
</p>

- The NetExec output shows that `svc_sql` successfully authenticated to DC01 and enumerated its SMB shares. The account does not have administrative access to `ADMIN$` or `C$`, but it does have `READ` access to `NETLOGON` and `SYSVOL`, giving the attacker an opportunity to inspect domain scripts and configuration files for exposed credentials or other sensitive information

- When NetExec runs the `--shares` command, it checks each SMB share to determine what permissions `svc_sql` has. In this case, NetExec tested the `SYSVOL` share and confirmed that the account has `READ` access. We can verify this activity in Splunk by looking at Event ID `5140`, which shows that the SYSVOL share was accessed during the permission check

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/601afdab-9e91-4d23-b17c-e8ef927562d1" />
</p>

- The same process occurs for the `NETLOGON` and `IPC$` shares. NetExec tests each share individually to determine what access `svc_sql` has, and Windows logs these checks as Event ID `5140` in Splunk. In our logs, `NETLOGON` and `IPC$` were also accessed during the enumeration, confirming that NetExec checked the permissions of each available SMB share

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/8bf09407-4961-452a-bf5b-2f8310c0e71b" />
</p>

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/79cf4041-42bf-4a5f-bbd1-2b77db9394ea" />
</p>

## Defense Best Practices for `Kerberoasting`

- To defend against Kerberoasting, use gMSAs where possible so service account passwords are automatically managed and rotated. Disable RC4-HMAC and prefer AES-128/AES-256 for Kerberos authentication. For legacy service accounts, use long, complex passwords and apply least privilege to limit what the account can access. Finally, monitor Event ID `4769` for unusual TGS requests, especially requests for non-machine accounts using RC4 (`0x17`), and correlate them with Event ID `4624` and `5140` to detect the use of compromised credentials

## What is `Password Spraying`
Password spraying is a cyberattack in which an attacker tries a small number of common passwords (such as `"Spring2026!"` or `"Password123"`) across many different user accounts instead of attempting many passwords on a single account. This helps avoid account lockouts while increasing the chances of finding accounts that use weak or reused passwords

- In Kali, I first created a userlist containing the passwords for the domains accounts. For learning purposes, one of them I already know which is `welcome1` for `Alisha`'s account
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/08e2d872-c8de-47eb-ac43-7ed801a71c90" />
</p>

- I needed to install `Go` for this
```
sudo apt update
sudo apt install git golang -y
```
- Then I installed `kerbrute`
```
git clone https://github.com/ropnop/kerbrute.git
cd kerbrute
go build -o kerbrute
./kerbrute -h
```

- Then, I starting spraying the passwords
```
./kerbrute passwordspray -d homelab.local --dc 10.10.10.10 users.txt <password>
```
- I tested it with two passwords, `Password01` and `welcome1` and we can see that while the first password did not work, the second one did work for `Alisha`'s account only which is expected
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/16752e67-6ba0-421d-b1e4-8f9abe6eb482" />
</p>

- When `alisha@homelab.local:welcome1` succeeded, we could observe a successful Kerberos authentication in Splunk through Event ID `4768`. Because the event originated from the Kali VM and occurred during the password-spraying activity, it provides evidence that the credential guess was successful

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/41294894-e93c-45c5-bd33-46ce7ee2a260" />
</p>

- When the password spraying attempt failed against the `Administrator` account, we can detect it in Splunk using Event ID `4771`. The `Failure_Code` of `0x18` indicates that the password was incorrect, while the `Client_Address` of `10.10.10.20` identifies the Kali machine as the source of the failed authentication attempt

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/b0a0a0a3-87af-45df-9cb3-f1f42b3751b1" />
</p>


## Defense Best Practices for `Password Spraying` attacks

- Enforce strong, unique passwords and block common passwords such as `welcome1`. Use MFA where possible to prevent stolen credentials from being used. Configure account lockout/smart lockout policies to limit repeated attempts and restrict access to Domain Controllers through network segmentation. Finally, monitor Event ID `4771` for repeated `0x18` failures, especially when one source IP targets multiple accounts, and correlate these with successful `4768` or `4624` events

## What is a `Golden Ticket`

- A Golden Ticket attack is a post exploitation attack against Kerberos authentication in Active Directory that allows an attacker to forge a valid Ticket Granting Ticket (TGT) and impersonate any user in the domain

- The attack begins after an attacker has compromised a Domain Controller (or otherwise obtained the password hash of the `KRBTGT` account). Since every Kerberos TGT is signed using the `KRBTGT` account's secret key, possession of this hash allows the attacker to create their own forged `TGT` with arbitrary user identities, group memberships (such as Domain Admins), and expiration times

- Instead of requesting an initial TGT from the Key Distribution Center (KDC), the attacker injects the forged TGT into their session and presents it to the KDC when requesting service tickets (`TGS-REQ`). Because the forged TGT is correctly signed with the `KRBTGT` key, the KDC considers it legitimate and issues valid service tickets (`TGS-REP`) for requested services. This enables the attacker to authenticate as virtually any user and gain persistent, domain-wide access until the `KRBTGT` account password is reset twice

### Targeting the DC as Administrator
- Now, let's assume we already have the Domain Administrator credentials, we can run the below command and this pulls every account hash from the DC's database including `krbtgt`
```
impacket-secretsdump homelab.local/administrator:AdminPassword@10.10.10.10
```
- We were successfully able to retrieve the `krbtgt` account's key material, as highlighted in blue. For this demonstration, I will use the `KRBTGT AES` key with Impacket's ticketer to forge the TGT

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/b7438446-7a82-47e2-9840-cfb412ec2698" />
</p>

- I also needed the domain SID and I ran Powershell on the DC to get it. We need this because a forged ticket has to claim membership in a specific domain

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/a7505858-09fa-4964-91cf-477019b7e606" />
</p>

- An example is `S-1-5-21-1234567890-1234567890-1234567890` and now we can now forge the ticket

```
impacket-ticketer -aesKey <aesKey> -domain-sid <domain-sid> -domain homelab.local -user-id 500 -groups 512,513,518,519,520 administrator
```
- The ticket is forged and this creates a file called `administrator.ccache`. Above, we knew the `aesKey` from dumping which signs the fake ticket so the KDC accepts it as a legitimate ticket, `-user-id 500` tells us that the RID is 500 which is always the built in Administrator so this ticket has Administrator level rights and `administrator` which is a real AD account
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/27fb61d6-7b71-414f-917b-42d0e11dbf60" />
</p>

- We can load the forged ticket into our session where this tells Kerberos aware tools to use this ticket file for authentication instead of asking for a password
```
export KRB5CCNAME=administrator.ccache
```
- and running it with gives us the shell
```
impacket-smbexec -k -no-pass homelab.local/administrator@dc01.homelab.local
```
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/906c8119-dd61-4e74-8749-7e3f472d26ab" />
</p>

- Now, we can detect this attack by looking for Event ID `4769` (TGS requests) without a preceding Event ID `4768` (TGT request). In normal Active Directory operations, a user must first request a TGT (Event ID `4768`) before requesting a Service Ticket (TGS). However, with a Golden Ticket, the attacker presents the forged TGT directly to the Domain Controller, bypassing the normal TGT request process. This creates an orphan TGS per se, where an Event ID `4769` appears without a corresponding Event ID `4768`

```
index=main (EventCode=4768 OR EventCode=4769)
| eval user = lower(coalesce(Target_User_Name, Account_Name))
| eval user = replace(user, "@.*", "")
| eval src_ip = coalesce(Client_Address, IpAddress, Source_Address)
| eval src_ip = replace(src_ip, "::ffff:", "")
| stats 
    count(eval(EventCode=4768)) as tgt_requests, 
    count(eval(EventCode=4769)) as tgs_requests, 
    values(Service_Name) as services_accessed 
    by user, src_ip
| where tgt_requests == 0 AND tgs_requests > 0
| table user, src_ip, tgt_requests, tgs_requests, services_accessed
```

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/a4f75f08-0007-41ee-8108-dd51bbbcdd33" />
</p>

- This flags any account that is requesting service access across the network whose initial TGT was never issued by the Domain Controller

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/6bc01f4c-dd78-4a1e-934e-48f8fc38d12d" />
</p>

- While the orphan TGS pattern is a useful indicator, we can also look at what happened after the forged ticket was used. Event ID `7045` shows that temporary services with random names, such as `kOpQmldp` and `oqwvcyTL`, were created on DC01. This is consistent with how Impacket's smbexec works when running commands remotely. The event also shows commands such as `whoami` and `dir` being written to a temporary batch file, executed, and their output redirected to a temporary file under the `C$` share before the batch file is deleted. This gives us strong evidence that smbexec was used to remotely execute commands after the forged ticket was accepted

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/45dc63e3-6769-4ff0-abf7-5c5e11182e80" />
</p>

- We can also correlate this activity with Event ID `4672`, which shows that the forged Administrator account (RID 500) was given special administrative privileges on DC01 showing that the same privileged session was being used to execute commands on the Domain Controller

## Defense Best Practices for `Golden Ticket` attacks
- To defend against Golden Ticket attacks, reset the krbtgt password twice with enough time for the changes to replicate across the domain. Protect Domain Controllers in a secure Tier 0 environment and use dedicated PAWs to reduce the risk of stolen credentials. Keep Domain Controllers fully patched with the latest Kerberos security updates, and place high-privilege admins in the Protected Users group for stronger protection. Finally, monitor Windows security events such as `4769`, `4624`, and `4672` for unusual ticket requests, network logons, and unexpected administrative privileges
