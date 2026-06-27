# Setup
- This part details about downloading and uploading the ISO, creating the Domain Controller and the Client VM

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

<img width="873" height="546" alt="image" src="https://github.com/user-attachments/assets/c307b470-4cb5-4142-aa97-3a057104f088" />

- We are finally here and I went through the options

<img width="1056" height="548" alt="image" src="https://github.com/user-attachments/assets/da81b6d7-2cf9-4b5a-a1dd-1a61a0fc8d59" />

- I selected the highlighted option

<img width="877" height="547" alt="image" src="https://github.com/user-attachments/assets/8d4623ca-9eb2-45aa-9eb5-f1415ce92d2e" />

- I selected `Windows Server 2025 Standard Evaluation (Desktop Experience)`

<img width="959" height="545" alt="image" src="https://github.com/user-attachments/assets/2c4b54c4-0ab3-44d2-ac01-1424e033c8e9" />

- I selected the highlighted option

<img width="871" height="548" alt="image" src="https://github.com/user-attachments/assets/fca9279e-eb84-430f-a5d6-1374dac8b205" />

- Click install

<img width="868" height="541" alt="image" src="https://github.com/user-attachments/assets/20fe2713-39a2-4dbf-a455-37bc2244da3e" />

- The Windows OS has finished installing, we are now at this screen

<img width="876" height="547" alt="image" src="https://github.com/user-attachments/assets/5de12488-1290-420b-a38c-ffda51e8dff1" />

- We are now at the Server Manager page after everything has finished installing and setting up

<img width="919" height="572" alt="image" src="https://github.com/user-attachments/assets/d2ad176d-92cf-4984-8354-804556534e5c" />

- However, before installing Active Directory, I first need to make sure the Domain Controller (DC) has a static IP address before promoting it. I opened `powershell` and typed `ipconfig /all`. We can see that the IP address assigned to my VM was via DHCP and there is a lease set which means it will expire tomorrow and can have a different IP address and so it is vital to set it to a static IP address so the Client VM can join it and not have trouble finding it at a later time

<img width="1128" height="658" alt="image" src="https://github.com/user-attachments/assets/71986c96-94a4-46b0-87de-aa073ae3a1ed" />

- I successfully changed the IP address
```
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.1.200 -PrefixLength 24 -DefaultGateway 192.168.1.254
```
<img width="1103" height="587" alt="image" src="https://github.com/user-attachments/assets/b24efc8a-7cf9-4488-af77-3114d4545275" />

- Then, I configured the DNS server to point to the Domain Controller (`192.168.1.100` on the DC, and the DC's static IP on client machines). When the server is promoted to a Domain Controller, it also becomes a DNS server. Active Directory Domain Services (AD DS) registers SRV records in DNS, which advertise services such as LDAP and Kerberos. The Domain Controller queries its own DNS to locate these services, while client computers query the DC's DNS server for the SRV records to discover the Domain Controller before joining the domain or authenticating
```
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.1.100
```

<img width="908" height="537" alt="image" src="https://github.com/user-attachments/assets/dd20fa11-eca1-470f-91ae-01bc4b5ba0ae" />

- Both the IP address and DNS server have been configured

<img width="1020" height="601" alt="image" src="https://github.com/user-attachments/assets/6781f681-b07c-4b9c-9f95-2bdf67b309ed" />

- Then, I renamed the server's name to `DC01`
```
Rename-Computer -NewName "DC01" -Restart
```
## Installing Active Directory Domain Service (AD DS)

- I first clicked on `Manage` > `Add roles and features`

<img width="911" height="572" alt="image" src="https://github.com/user-attachments/assets/69871347-74d6-41f6-a824-f9d5ac418880" />

- I went with the default option

<img width="912" height="569" alt="image" src="https://github.com/user-attachments/assets/93fce793-785f-4001-a472-c4ec07f5dcd9" />

- Our server is already highlighted :D

<img width="911" height="569" alt="image" src="https://github.com/user-attachments/assets/06382122-c156-4f72-8a4e-e19f50865aba" />

- From the server roles, I selected AD DS

<img width="1043" height="646" alt="image" src="https://github.com/user-attachments/assets/2124c8f2-6ca5-40e8-9cd9-22f5c3a36fe5" />

- Click install

<img width="1818" height="1136" alt="image" src="https://github.com/user-attachments/assets/89179d2d-64f9-4449-9f60-a3838ba16694" />

- The installation has succeeded and now we can click the `Promote this server to a domain controller` link

<img width="963" height="602" alt="image" src="https://github.com/user-attachments/assets/1a9526b9-0afb-4ef6-bd97-dbe868d2f9e8" />

- Then, I created a new forest and named it `homelab.local`

<img width="912" height="567" alt="image" src="https://github.com/user-attachments/assets/4a598e64-b8dd-4db4-a5af-ecb3854fe66b" />

- We click continue since the box is greyed out anyways

<img width="907" height="568" alt="image" src="https://github.com/user-attachments/assets/6f41915e-f520-45a1-8f49-a56baad3b738" />

- The NetBIOS domain name is already filled for us

<img width="912" height="568" alt="image" src="https://github.com/user-attachments/assets/09a2b1dc-e2af-4453-85c1-26ceb10cb70e" />

- We see where our database logs are stored

<img width="908" height="571" alt="image" src="https://github.com/user-attachments/assets/e697a0bc-98ae-4801-9112-58bca7e9cd64" />

- Since all prerequisites pass, we click `Install`

<img width="908" height="567" alt="image" src="https://github.com/user-attachments/assets/387ab34b-4bcb-4c62-a971-e69bea7ec680" />

- Now, it has finished and logging in looks a bit different

<img width="909" height="569" alt="image" src="https://github.com/user-attachments/assets/8fa94dc2-e9be-4257-816f-2ec0a2611e12" />

- Once we login, we can see the `AD DS`, `DNS` and `File and Storage Services` on the left side

<img width="916" height="576" alt="image" src="https://github.com/user-attachments/assets/761eba0a-bc3a-43c5-9038-de755aa68359" />

- Next, I clicked on `Active Directory Users and Computers`

<img width="909" height="572" alt="image" src="https://github.com/user-attachments/assets/535d1572-6591-4207-bfa1-4671138b3661" />

- We are now creating a new user account

<img width="919" height="571" alt="image" src="https://github.com/user-attachments/assets/571c52a2-6702-4c4e-9d63-d670d9f6583e" />

- I put the name `Jack` for the first name and logon name `jack`

<img width="918" height="575" alt="image" src="https://github.com/user-attachments/assets/08102008-dfd2-4af1-8dd9-54f705cfaff0" />

- For test purposes, I unchecked the `User must change password at next logon` and checked the box `Password never expires`

<img width="909" height="570" alt="image" src="https://github.com/user-attachments/assets/e9a2b651-f400-4b6e-b699-cfea46fe0c4e" />

- We have our domain user account ready

<img width="906" height="565" alt="image" src="https://github.com/user-attachments/assets/7b3d5077-a157-45d1-8846-f03da100cf9a" />

- We can see `Jack` in the Users OU (Organizational Unit which is basically a folder)

<img width="941" height="585" alt="image" src="https://github.com/user-attachments/assets/e6f2e41c-8ac2-4bf1-92a4-9df1f1d845e8" />
