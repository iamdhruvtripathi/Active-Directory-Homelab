# Kali Linux
- Now that we have built our own AD environment, we will assume the role of an attacker against our own domain. We will begin with `Nmap` for `Reconnaissance` before demonstrating the `AS-REP Roasting`, `Golden Tickets` and `Password Spraying` attacks
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

## What is `Nmap`
- Nmap (Network Mapper) is a network reconnaissance and enumeration tool used to discover hosts, identify open ports, detect running services, and gather information about devices on a network. It is commonly used by system administrators for network management and by security professionals during security assessments and penetration testing

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/2efee528-2ccf-4d88-a50f-d0138ec2c6ba" />
</p>

- These open ports and services tell us a lot about the DC and each can be a door for the attacker per se. For example port `53` where `DNS` is running which is expected because this is on the DC. `Kereberos-sec` is running on port `88` and that confirms this is a domain controller. Port `135` has `msrpc` which is a windows remote procedure calls used for lots of admin activity. Ports `139` and `445` are for `NetBIOS` and `SMB` respectively and this is for file sharing. Ports `389` and `636` are for `LDAP` and `LDAPS` respectively and this is the AD query port. Port `464` is the kerberos password change port. Port `593` is RPC over HTTP aka remote management. Ports `3268` and `3269` are global catalog LDAP which is a domain wide AP search port and finally we have port `5985` which is `WSMan` which is `WinRM`

- We can see when I ran with the `-sV` option, it confirms a lot of what we said before
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/e8b648c4-4481-4c90-8a21-3e93a8374e87" />
</p>

## What is `Password Spraying`
Password spraying is a cyberattack in which an attacker tries a small number of common passwords (such as `"Spring2026!"` or `"Password123"`) across many different user accounts instead of attempting many passwords on a single account. This helps avoid account lockouts while increasing the chances of finding accounts that use weak or reused passwords

- In Kali, I first created a userlist containing the passwords for the domains accounts. For learning purposes, one of them I already know which is `welcome` for `Alisha`'s account
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/08e2d872-c8de-47eb-ac43-7ed801a71c90" />
</p>

- Then, I starting spraying the passwords
```
kerbrute passwordspray -d homelab.local --dc 10.10.10.10 users.txt <password>
```
- I tested it with two passwords, `Password01` and `welcome1` and we can see that while the first password did not work, the second one did work for `Alisha`'s account only which is expected
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/16752e67-6ba0-421d-b1e4-8f9abe6eb482" />
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

- Now, as a detective we also need to see what happened. If we take a look into Splunk searching for the Event ID `4768`, we can see indeed it was our Kali linux machine (IP address `10.10.10.20`) that requested the TGT. The account name was indeed `Alisha`. Furthermore, we can see `Pre_Authentication_Type` is set to `0` which means that Kerberos pre-authentication was explicitly bypassed or not required. Standard Kerberos authentication uses type `2` (`PA-ENC-TIMESTAMP`). The `Ticket_Encryption_Type` is set to `0x17` which corresponds to `RC4-HMAC`. This specific hashing structure is easier to crack offline compared to AES-128 (`0x11`) or AES-256 (`0x12`)
  
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/f55d592f-4fab-4ffe-a225-07d93a36b746" />
</p>

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

- We can clearly see that my Kali Linux VM (`10.10.10.20`) had `Alisha`'s credentials and used them to log into the VM via RDP, making it appear as though `Alisha` had logged in herself. This represents inital access through the use of valid credentials over RDP
 
## What is a `Golden Ticket`

- A Golden Ticket attack is a post exploitation attack against Kerberos authentication in Active Directory that allows an attacker to forge a valid Ticket Granting Ticket (TGT) and impersonate any user in the domain

- The attack begins after an attacker has compromised a Domain Controller (or otherwise obtained the password hash of the `KRBTGT` account). Since every Kerberos TGT is signed using the `KRBTGT` account's secret key, possession of this hash allows the attacker to create their own forged `TGT` with arbitrary user identities, group memberships (such as Domain Admins), and expiration times

- Instead of requesting an initial TGT from the Key Distribution Center (KDC), the attacker injects the forged TGT into their session and presents it to the KDC when requesting service tickets (`TGS-REQ`). Because the forged TGT is correctly signed with the `KRBTGT` key, the KDC considers it legitimate and issues valid service tickets (`TGS-REP`) for requested services. This enables the attacker to authenticate as virtually any user and gain persistent, domain-wide access until the `KRBTGT` account password is reset twice

### Targeting the DC as Administrator
- Now, let's assume we already have the Domain Administrator credentials, we can run the below command and this pulls every account hash from the DC's database including `krbtgt`
```
impacket-secretsdump homelab.local/administrator:AdminPassword@10.10.10.10
```
- We were successfully able to dump the `krbtgt`'s hash as highlighted in blue. We specifically need the AES key

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/b7438446-7a82-47e2-9840-cfb412ec2698" />
</p>

- I also needed the domain SID and I ran Powershell on the DC to get it. We need this because a forged ticket has to claim membership in a specific domain

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/a7505858-09fa-4964-91cf-477019b7e606" />
</p>

- An example is `S-1-5-21-1234567890-1234567890-1234567890` but there is another number attached after the third `1234567890` in the above SID which we can drop because that states the account's individual RID which is part of the domain SID itself. We can now forge the ticket

```
impacket-ticketer -aesKey <aesKey> -domain-sid <domain-sid> -domain homelab.local -user-id 500 -groups 512,513,518,519,520 administrator
```
- The ticket is forged and this creates a file called `administrator.ccache`. Above, we knew the `aesKey` from dumping which signs the fake ticket so the KDC accepts it as a legitimate ticket, `-user-id 500` tells us that the RID is 500 which is always the built in Administrator so this ticket has Administrator level rights and `administrator` which is a real AD account
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/8d83056b-3f9c-4e68-b0ad-7a99c21d062c" />
</p>

- We can load the forged ticket into our session where this tells Kereberos aware tools to use this ticket file for authentication instead of asking for a password
```
export KRB5CCNAME=administrator.ccache
```
- and running it with gives us the shell
```
impacket-smbexec -k -no-pass homelab.local/administrator@dc01.homelab.local
```
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/158feae8-7319-44df-9b2b-4ab2782b27c5" />
</p>

- Above, the golden ticket forges a Kerberos Ticket Granting Ticket (`TGT`) that allows an attacker to authenticate as a privileged user, such as a Domain Administrator. We used the tool `smbexec` where when it is executed, it presents the forged `TGT` to the Key Distribution Center (KDC), which issues a legitimate service ticket (`TGS`) for the target SMB (`CIFS`) service because the forged TGT appears valid. The tool then uses this TGS to authenticate to the SMB service on the target system. Since the impersonated account has administrative privileges, `smbexec` leverages built-in Windows administrative features, such as creating and starting a temporary service, to execute commands remotely and provide an interactive shell. Therefore, the Golden Ticket enables trusted authentication, while the shell is obtained through legitimate Windows remote administration mechanisms
