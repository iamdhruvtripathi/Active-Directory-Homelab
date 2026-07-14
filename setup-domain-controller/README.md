# Setup - Domain Controller VM
- This part details about downloading and uploading the ISO, and creating the Domain Controller

## Downloading the Windows Server 2025 ISO
- I went to the official Microsoft website to grab this ISO

  <img width="1512" height="826" alt="image" src="https://github.com/user-attachments/assets/894106f6-5e12-480a-8874-edfe36fd8dda" />

## Downloading the Windows 11 ISO
- I did the same thing for downloading Windows 11 (Client VM)

  <img width="1512" height="740" alt="image" src="https://github.com/user-attachments/assets/32cadf29-33ac-433b-9ed5-ea7b70fb6dc6" />

## Creating the Domain Controller VM
- After downloading and uploading the ISOs, I started creating the VM and started filling out all the information needed. I named it `DC01`

  <img width="1512" height="683" alt="image" src="https://github.com/user-attachments/assets/4c495da4-bc34-4610-b25d-ec997db9da3f" />

- Then, I selected the ISO that I downloaded and the type to `Microsoft Windows`

  <img width="1512" height="689" alt="image" src="https://github.com/user-attachments/assets/35f880a7-422a-47ed-8791-10fcb0fa94e9" />

- All default options are checked (I had to check the box for `Qemu agent`)
  <img width="1512" height="693" alt="image" src="https://github.com/user-attachments/assets/752b2b87-46d6-4a65-8202-42eb062365c5" />

- This option is default
  <img width="1512" height="702" alt="image" src="https://github.com/user-attachments/assets/fd897be9-c3a9-4bfa-a96d-6645e4976e9c" />

- Set the CPU to 2 cores and Memory to ~5 GB

  <img width="1512" height="690" alt="image" src="https://github.com/user-attachments/assets/1361f522-7ee1-424b-bc0a-a9595ad74407" />

- Network is temporarily set to `vmbr0`

  <img width="1512" height="688" alt="image" src="https://github.com/user-attachments/assets/2d8b2c6a-b58e-4889-b597-a59d2a9b8531" />

## Creating the Client VM
- This process below follows the same thing as I did above but for the Client VM and so for brevity, this was the final step. The only change was naming this one to `WKSTN01`
  <img width="1512" height="690" alt="image" src="https://github.com/user-attachments/assets/022db911-c04f-4e5d-bb2a-dfea6b2009bb" />

## Windows Installation (Domain Controller VM)
- Below are the steps for the standard Windows installation. I hit `Enter` for this
<p align="center">
  <img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/c307b470-4cb5-4142-aa97-3a057104f088" />
</p>

- We are finally here and I went through the options

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/da81b6d7-2cf9-4b5a-a1dd-1a61a0fc8d59" />
</p>

- I selected the highlighted option

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/8d4623ca-9eb2-45aa-9eb5-f1415ce92d2e" />
</p>

- I selected `Windows Server 2025 Standard Evaluation (Desktop Experience)`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/2c4b54c4-0ab3-44d2-ac01-1424e033c8e9" />
</p>

- I selected the highlighted option

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/fca9279e-eb84-430f-a5d6-1374dac8b205" />
</p>

- Click install

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/20fe2713-39a2-4dbf-a455-37bc2244da3e" />
</p>

- The Windows OS has finished installing, we are now at this screen

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/5de12488-1290-420b-a38c-ffda51e8dff1" />
</p>

- We are now at the Server Manager page after everything has finished installing and setting up

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/d2ad176d-92cf-4984-8354-804556534e5c" />
</p>

- However, before installing Active Directory, I first need to make sure the Domain Controller (DC) has a static IP address before promoting it. I opened `powershell` and typed `ipconfig /all`. We can see that the IP address assigned to my VM was via DHCP and there is a lease set which means it will expire tomorrow and can have a different IP address and so it is vital to set it to a static IP address so the Client VM can join it and not have trouble finding it at a later time

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/71986c96-94a4-46b0-87de-aa073ae3a1ed" />
</p>

- I successfully changed the IP address via the run box (`Win + R`)

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/b47a4dfa-6ff0-48db-bc7b-d9fded11de93" />
</p>

- I also configured the DNS server to point to the Domain Controller (`10.10.10.10` on the DC, and the DC's static IP on client machines). When the server is promoted to a Domain Controller, it also becomes a DNS server. Active Directory Domain Services (AD DS) registers SRV records in DNS, which advertise services such as LDAP and Kerberos. The Domain Controller queries its own DNS to locate these services, while client computers query the DC's DNS server for the SRV records to discover the Domain Controller before joining the domain or authenticating

- Then, I renamed the server's name to `DC01`
```
Rename-Computer -NewName "DC01" -Restart
```
## Installing Active Directory Domain Service (AD DS)

- I first clicked on `Manage` > `Add roles and features`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/69871347-74d6-41f6-a824-f9d5ac418880" />
</p>

- I went with the default option

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/93fce793-785f-4001-a472-c4ec07f5dcd9" />
</p>

- Our server is already highlighted :D

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/bc29052b-4b2d-48eb-8944-258071cb6a41" />
</p>

- From the server roles, I selected AD DS

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/2124c8f2-6ca5-40e8-9cd9-22f5c3a36fe5" />
</p>

- Click install

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/89179d2d-64f9-4449-9f60-a3838ba16694" />
</p>

- The installation has succeeded and now we can click the `Promote this server to a domain controller` link

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/1a9526b9-0afb-4ef6-bd97-dbe868d2f9e8" />
</p>

- Then, I created a new forest and named it `homelab.local`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/4a598e64-b8dd-4db4-a5af-ecb3854fe66b" />
</p>

- We click continue since the box is greyed out anyways

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/6f41915e-f520-45a1-8f49-a56baad3b738" />
</p>

- The NetBIOS domain name is already filled for us

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/09a2b1dc-e2af-4453-85c1-26ceb10cb70e" />
</p>

- We see where our database logs are stored

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/e697a0bc-98ae-4801-9112-58bca7e9cd64" />
</p>

- Since all prerequisites pass, we click `Install`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/387ab34b-4bcb-4c62-a971-e69bea7ec680" />
</p>

- Now, it has finished and logging in looks a bit different

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/8fa94dc2-e9be-4257-816f-2ec0a2611e12" />
</p>

- Once we login, we can see the `AD DS`, `DNS` and `File and Storage Services` on the left side

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/761eba0a-bc3a-43c5-9038-de755aa68359" />
</p>

- Next, I clicked on `Active Directory Users and Computers`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/535d1572-6591-4207-bfa1-4671138b3661" />
</p>

- We are now creating a new user account

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/571c52a2-6702-4c4e-9d63-d670d9f6583e" />
</p>

- I put the name `Jack` for the first name and logon name `jack`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/08102008-dfd2-4af1-8dd9-54f705cfaff0" />
</p>

- For test purposes, I unchecked the `User must change password at next logon` and checked the box `Password never expires`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/e9a2b651-f400-4b6e-b699-cfea46fe0c4e" />
</p>

- We have our domain user account ready

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/7b3d5077-a157-45d1-8846-f03da100cf9a" />
</p>

- We can see `Jack` in the Users OU (Organizational Unit which is basically a folder)

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/e6f2e41c-8ac2-4bf1-92a4-9df1f1d845e8" />
</p>
