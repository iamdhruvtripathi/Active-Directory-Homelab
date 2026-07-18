# Kali Linux
- Now that we have built our own AD environment, we will now going to pretend to be an attacker and attack our own domain. I will explore four attacks: `AS-REP Roasting`, `Kerberoasting`, `Pass-the-Hash`, and `Golden Tickets`

## What is `AS-REP Roasting`
- Normally the user sends their username and timestamp and this timestamp is encrypted with a key that is derived from the user's password to the KDC (Key Distribution Center) on the DC. The KDC derives the same key from the user's stored password information and decrypts the timestamp and if its valid, the user is authenticated and given a AS-REP which contains a TGT encrypted with a key derived from a special hidden account called krbtgt and a session key encrypted with a key derived from the user's password
- AS-REP roasting, the target account has Kerberos pre-authentication disabled and this skips the user authentication part and just sends the TGT (encrypted) and session key (encrypted). What attackers can do is request that authentication data on behalf of a user, brute force it and recover the user's password by guessing the correct password-derived key that decrypts the session key (the user-encrypted portion). Once it decrypts, the attacker knows that this is the correct user's password

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
