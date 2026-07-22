# Setup - Client VM
- This part details about creating the isolated switch (`vmbr1`) as mentioned in the main `README.md` and the Client VM
## Creating the Isolated Switch (`vmbr1`)  for `DCO1`
- Note: The reason why I did this was because so I can create a isolated private network separate from my home network and so there would be no interference from my private network to my home network. These VMs are not able to access my home network or the internet unless I add a router or firewall later

- First, I went to the web console and clicked `Create` > `Linux Bridge`


  <img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/fa427081-2034-452d-a352-6e0a51cc3882" />


- Then, I clicked `Apply configuration` and we can see that the `Active` column is set to `Yes` for `vmbr1`

  <img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/09cacb18-cf84-4de0-a498-4f2bb4461819" />

- Then, I clicked on `DC01` > `Hardware` and changed the bridge to `vmbr1`

  <img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/8e778448-cae8-42d4-bb8f-36a7ed3390b5" />

- As we can see, the packets don't even make it out of the VM's NIC. It is completely isolated as that is the setting we applied

  <img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/1fc12007-570f-459b-bef0-1865bce9f283" />


## Setting up the Client VM
- For setting up the Client VM, the steps are very similar, if not, identical to setting up the Domain Controller and so I will skip showing all of the steps

- Update: We are in the process of installing Windows 11 Pro. I chose Windows 11 Pro because Windows 11 Home lacks the business networking features needed for enterprise environments. Most importantly, Home edition cannot join an Active Directory domain

   
  <img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/076c62ea-fb21-4d09-bbfd-de942f3b59d8" />
   

- Now, I have arrived at this screen where Microsoft wants me to connect to the internet and login in to a Microsoft account but there is a way to bypass this

   
  <img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/509100b7-308d-40e2-a4c6-7d286e81a2bd" />
   

- We press `Fn + Shift + F10` to open up command prompt and type `OOBE\BYPASSNRO` (this reboots the VM into an offline mode, letting us create a traditional local user account). As we can see, after the VM reboots, we get another option at the bottom

   
  <img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/a49206ed-3963-469f-a918-017696b63f50" />
   

- I am waiting...

   
  <img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/72a91e06-5618-49cb-919b-960db514dd8d" />
   

- We have finally arrived and the installation was successful

   
  <img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/f94504a4-cbcb-4ac9-97a3-4c8ec723c3c5" />


- First thing to do was to move this Windows VM 11 Client to `vmbr1`

  <img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/6e6ab4a1-2646-4c3f-9426-ee5fb155ba16" />

- Next thing before I join it to the domain is to set a static IP and point the DNS to the DC. As you can see below, I set the IP address of the Client VM to `10.10.10.11` and the DNS server to `10.10.10.10`

   
  <img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/96135446-c14a-45b0-bc87-04399f82fc2a" />

   

- As a confirmation, I pinged the DC and we can see we received a successful reply :D

   
  <img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/0759dde4-dbe0-470e-8a92-7deaa11e10cc" />

   

- Now, we join the client VM to the domain `homelab.local`. First, I ran `sysdm.cpl` using `Win + R`. Then, I clicked `Change` at the bottom

   
  <img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/aa50101c-d0fc-4a2d-b5cb-1d797a11e061" />
   

- I typed in the domain `homelab.local`

   
  <img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/6ab034d9-4ed6-4274-8237-587dc0fff0cc" />
   

- Entered my username and password

   
  <img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/d6570978-71bd-4608-b787-9b0caaf0b17d" />
   

- We can see it says a successful welcome message

   
  <img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/85c9734f-fa85-4981-8716-8222939b0488" />
   

- Now, I restarted to let the changes take effect but once we get to this page, we don't login as the local account, instead I chose the `Other User` option

   
  <img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/3ec2f1ec-4fb5-49e0-bfdb-a3543aa00451" />
   

- We can see it says `Sign in to: HOMELAB`

   
  <img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/b35ecd41-5e77-44a8-b96d-bf17ce55359f" />
   

- Then, I signed in with the same username and password I created as the admin on the domain controller (created the `Jack` domain user account)

   
  <img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/ee3f37c0-791a-44ea-bd99-1d02d706459c" />
   

- We can see I am succesfully able to login as the standard local domain user

  <img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/274d4822-64bc-486b-b296-eac50c84d45f" />
