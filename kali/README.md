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

- Now, two things to make this go smoothly. I will set `Alisha`'s password to `Summer2026` and check `Do not require Kerberos pre-authentication` but first I had to change the password settings so I could actually set her password to that value and then do `gpupdate /force` to make sure it applied everyone in the domain 

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/782b6402-66d5-4635-a7bd-064d81f37e9f" />
</p>

- Now, I changed her password to `Summer2026`

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
- BOOM!, we get the result we wanted. Now that we have her encrypted data (`AS-REP hash`), what we can do is use `John the Ripper` for offline password cracking to get her real password (which should be `Summer2026`). The idea here is that `John the Ripper` tries different passwords to get the correct password derived key and if we are able to decrypt the session key and we get valid data in the `AS-REP` hash it means we have correctly guessed the user's password

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/bf5d363a-6c07-49c9-a304-2c7f3c53eae2" />
</p>

- Now, I requested the data again and saved it into a file called `hash.txt`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/d202dec8-cbac-4558-b1ea-3076fe5859a5" />
</p>

- I made some changes to the `.txt` file so when I feed it, `John` can actually understand it

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/ac06ddbd-b0f1-4fa9-b0dd-279b6e2dc167" />
</p>
