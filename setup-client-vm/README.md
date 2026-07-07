# Setup - Client VM
- This part details about creating the isolated switch (`vmbr1`) as mentioned in the main `README.md` and the Client VM
## Creating the Isolated Switch (`vmbr1`)
- Note: The reason why I did this was because so I can create a isolated private network separate from my home network and so there would be no interference from my private network to my home network. These VMs are not able to access my home network or the internet unless I add a router or firewall later

- First, I went to the web console and clicked `Create` > `Linux Bridge`

<p center="align">
  <img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/fa427081-2034-452d-a352-6e0a51cc3882" />
</p>

- Then, I clicked `Apply configuration` and we can see that the `Active` column is set to `Yes` for `vmbr1`

<p center="align">
  <img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/09cacb18-cf84-4de0-a498-4f2bb4461819" />
</p>

- Then, I clicked on `DC01` > `Hardware` and changed the bridge to `vmbr1`

<p center="align">
  <img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/8e778448-cae8-42d4-bb8f-36a7ed3390b5" />
</p>

- As we can see, the packets don't even make it out of the VM's NIC. It is completely isolated as that is the setting we applied

<p center="align">
<img width="960" height="563" alt="image" src="https://github.com/user-attachments/assets/1fc12007-570f-459b-bef0-1865bce9f283" />
</p>

## Setting up the Client VM
- For setting up the Client VM, the steps are very similar, if not, identical to setting up the Domain Controller and so I will skip showing all of the steps

- Update: We are in the process of installing Windows 11 Pro. I chose Windows 11 Pro because Windows 11 Home lacks the business networking features needed for enterprise environments. Most importantly, Home edition cannot join an Active Directory domain

<p center="align">
<img width="960" height="599" alt="image" src="https://github.com/user-attachments/assets/076c62ea-fb21-4d09-bbfd-de942f3b59d8" />
</p>

- Now, I have arrived at this screen where Microsoft wants me to connect to the internet and login in to a Microsoft account but there is a way to bypass this

<p center="align">
<img width="955" height="596" alt="image" src="https://github.com/user-attachments/assets/509100b7-308d-40e2-a4c6-7d286e81a2bd" />
</p>

- We press `Fn + Shift + F10` to open up command prompt and type `OOBE\BYPASSNRO` (this reboots the VM into an offline mode, letting us create a traditional local user account). As we can see, after the VM reboots, we get another option at the bottom

<p center="align">
<img width="1031" height="646" alt="image" src="https://github.com/user-attachments/assets/a49206ed-3963-469f-a918-017696b63f50" />
</p>

- I am waiting...

<p center="align">
<img width="949" height="595" alt="image" src="https://github.com/user-attachments/assets/72a91e06-5618-49cb-919b-960db514dd8d" />
</p>

- We have finally arrived and the installation was successful

<p center="align">
<img width="961" height="596" alt="image" src="https://github.com/user-attachments/assets/f94504a4-cbcb-4ac9-97a3-4c8ec723c3c5" />
</p>

- Next thing before I join it to the domain is to set a static IP and point the DNS to the DC. As you can see below, I set the IP address of the Client VM to `192.168.1.201` and the DNS server to `192.168.1.200`

<p center="align">
<img width="960" height="600" alt="image" src="https://github.com/user-attachments/assets/d1fd04c7-dafa-4523-b015-2aa32335de81" />
</p>

- As a confirmation, I pinged the DC and we can see we received a successful reply :D

<p center="align">
<img width="958" height="597" alt="image" src="https://github.com/user-attachments/assets/acef4098-58cb-483f-b7c7-2e27050155ef" />
</p>

- Now, we join the client VM to the domain `homelab.local`. First, I ran `sysdm.cpl` using `Win + R`. Then, I clicked `Change` at the bottom

<p center="align">
<img width="992" height="616" alt="image" src="https://github.com/user-attachments/assets/aa50101c-d0fc-4a2d-b5cb-1d797a11e061" />
</p>

- I typed in the domain `homelab.local`

<p center="align">
<img width="960" height="599" alt="image" src="https://github.com/user-attachments/assets/6ab034d9-4ed6-4274-8237-587dc0fff0cc" />
</p>

- Entered my username and password

<p center="align">
<img width="960" height="596" alt="image" src="https://github.com/user-attachments/assets/d6570978-71bd-4608-b787-9b0caaf0b17d" />
</p>

- We can see it says a successful welcome message

<p center="align">
<img width="959" height="597" alt="image" src="https://github.com/user-attachments/assets/85c9734f-fa85-4981-8716-8222939b0488" />
</p>

- Now, I restarted to let the changes take effect but once we get to this page, we don't login as the local account, instead I chose the `Other User` option

<p center="align">
<img width="1102" height="690" alt="image" src="https://github.com/user-attachments/assets/3ec2f1ec-4fb5-49e0-bfdb-a3543aa00451" />
</p>

- We can see it says `Sign in to: HOMELAB`

<p center="align">
<img width="1097" height="681" alt="image" src="https://github.com/user-attachments/assets/b35ecd41-5e77-44a8-b96d-bf17ce55359f" />
</p>

- Then, I signed in with the same username and password I created as the admin on the domain controller (created the `Jack` domain user account)

<p center="align">
<img width="1031" height="644" alt="image" src="https://github.com/user-attachments/assets/ee3f37c0-791a-44ea-bd99-1d02d706459c" />
</p>

- We can see I am succesfully able to login as the standard local domain user

<img width="958" height="593" alt="image" src="https://github.com/user-attachments/assets/274d4822-64bc-486b-b296-eac50c84d45f" />
