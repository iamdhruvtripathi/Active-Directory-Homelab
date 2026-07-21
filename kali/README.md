# Kali Linux
- Now that we have built our own AD environment, we will now going to pretend to be an attacker and attack our own domain. I will explore four attacks: `AS-REP Roasting`, `Kerberoasting`, `Pass-the-Hash`, and `Golden Tickets`

## What is `AS-REP Roasting`
- Normally, when a user authenticates with Kerberos, the client sends an `AS-REQ` containing the client's username and pre-authentication data. This pre-authentication data is typically an encrypted timestamp (`PA-ENC-TIMESTAMP`), which is encrypted using a key derived from the user's password. The Key Distribution Center (KDC) uses the same password-derived key to decrypt the timestamp and verify that the client knows the correct password. If the timestamp is successfully decrypted and valid, the KDC returns an `AS-REP` containing a Ticket Granting Ticket (TGT), which is encrypted with the `krbtgt` account's key, and a copy of the session key encrypted with the user's password-derived key

- In an `AS-REP` roasting attack, the target account has Kerberos pre-authentication disabled. Because the KDC no longer requires the client to prove knowledge of the user's password before issuing a ticket, anyone can request an `AS-REP` for that account. The `AS-REP` still contains the TGT and the session key encrypted with the user's password-derived key. An attacker can capture this response and perform an offline brute force or dictionary attack against the encrypted session key. For each password guess, the attacker derives the corresponding key and attempts to decrypt the encrypted session key. If the decryption succeeds, the password guess is correct

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
 
## What is `Kerberoasting`
- Kerberoasting is a post-exploitation attack against Kerberos authentication in Active Directory that targets domain service accounts

- The attack begins when an attacker has access to any authenticated domain user account. Using that account, they send a `TGS-REQ` (Ticket Granting Service Request) to the Key Distribution Center (KDC) for a service associated with a specific Service Principal Name (SPN). If the request is valid, the KDC returns a `TGS-REP` (Ticket Granting Service Reply) containing a service ticket

- The service ticket is encrypted using the service account's key, which is derived from the service account's password. Although the attacker cannot directly decrypt the ticket, they can extract the encrypted service ticket and perform offline password cracking. For each password guess, they derive the corresponding Kerberos key and attempt to decrypt the ticket. If decryption succeeds, the guessed password (or its equivalent key) is correct, allowing the attacker to recover the service account's credentials
