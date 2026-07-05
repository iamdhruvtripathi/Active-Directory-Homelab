# Windows Server Roles (Part 1)
- We are now going to install several Windows roles including the Active Directory Certificate Services (AD CS), DHCP server, File Server and Print Server
- Note: For organization, the AD DS and DHCP Server installation and configuration are on this `README.md` and the File Server and Print Server are on this [README.md](https://github.com/iamdhruvtripathi/Active-Directory-Homelab/blob/main/file-and-print-server/README.md)

## Installing the AD CS & DHCP
- First, I clicked on the `Add Roles and Features`

<img width="965" height="602" alt="image" src="https://github.com/user-attachments/assets/343242d0-f9b4-44b1-9b82-45359951d18a" />

- As you can see, I selected the roles I wanted to install

<img width="1042" height="647" alt="image" src="https://github.com/user-attachments/assets/3f071529-b8d1-4289-81c5-7c00691ece19" />

- I clicked through the options leaving them as default
<img width="964" height="603" alt="image" src="https://github.com/user-attachments/assets/58df2921-ee7e-4155-8664-060facbd158d" />

- Same here...
<img width="963" height="602" alt="image" src="https://github.com/user-attachments/assets/6ee73065-f844-4e73-ade3-efe70473142a" />

- And then we click `Install` :D
<img width="963" height="604" alt="image" src="https://github.com/user-attachments/assets/9dac2269-1552-4cd0-9177-ac3c5d76f8fa" />

- Now that the installation succeeded, we have to configure the `AD CS` and the `DHCP` server

<img width="963" height="602" alt="image" src="https://github.com/user-attachments/assets/32a664e7-cde0-4602-9668-621e2be47a50" />

- I successfully installed and configured the `DHCP` server as seen below

<img width="963" height="603" alt="image" src="https://github.com/user-attachments/assets/a6a22c9c-5671-4d66-9979-27ac93930203" />

- Now, I configured the `AD CS` and for this option I clicked `Enterprise CA`. The reason for this is because it seamlessly integrates with `AD DS` and it can look for any users/computers objects and verifies their identity to issue any certificates

<img width="961" height="599" alt="image" src="https://github.com/user-attachments/assets/db8f237b-8ca7-4a83-95ba-536592729a88" />

- I clicked `Create a new private key`

<img width="961" height="599" alt="image" src="https://github.com/user-attachments/assets/2e4c9277-f244-4882-9738-3681321dcca1" />

- After clicking through all the options, I successfully configured the `AD CS`

<img width="961" height="603" alt="image" src="https://github.com/user-attachments/assets/5638244d-a4e0-4ef8-8515-c35b79c88847" />

## Configuring the DHCP Scope
- Now, we have to configure the DHCP Scope (this is just the range of IP addresses the DC will hand out to the virtual machines so they can connect automatically without me having to configure static IPs manually). Now, here we are in the screen presented below

<img width="961" height="600" alt="image" src="https://github.com/user-attachments/assets/99f0db41-fad8-40cc-a3e4-7ad949f77784" />

- I set the range from `192.168.1.50` to `192.168.1.150`. The reason for this range is just to keep the DHCP pool separate from manually assigned addresses causing no IP conflicts

<img width="963" height="600" alt="image" src="https://github.com/user-attachments/assets/515f67e8-dcd2-4fa8-a701-bed8056d452e" />

- Here, I set the DNS server address (which is the DC itself)

<img width="964" height="603" alt="image" src="https://github.com/user-attachments/assets/b4242eca-ad3d-4551-a0a9-7298089c8a2e" />

- We have successfully configured the DHCP scope

<img width="960" height="600" alt="image" src="https://github.com/user-attachments/assets/06b1822f-0987-423f-a636-b8a243e310fe" />

## Testing the DHCP configurations
- Now, it's time to test what we configured by using the Windows client. We can see here that is was the IP address I manually assigned (`192.168.1.201`)

<img width="958" height="597" alt="image" src="https://github.com/user-attachments/assets/86f6487e-063a-47c9-8231-53fe3a31c827" />

- Now, since I hardcoded the IP address, we need to allow the network card to dynamically grab or release IP addresses. I went configure the network connection and clicked on `Obtain an IP address automatically`

<img width="954" height="596" alt="image" src="https://github.com/user-attachments/assets/481b8215-0eb9-4d5b-85e3-c1c7b6745919" />

- We can see it automatically got assigned an IP address (`192.168.1.50`)

<img width="961" height="600" alt="image" src="https://github.com/user-attachments/assets/5324a6e8-0c2f-44ef-9eb3-81c365abf767" />

- If we do `ipconfig /all`, we can see it was assigned by the DHCP server and not manually assigned. You can see I also set the lease for `8` days
<img width="956" height="596" alt="image" src="https://github.com/user-attachments/assets/79e5b283-eb2c-4e1f-b43f-ba47edd4df62" />

## Creating a custom Web Server certificate template
- I am now going to be working on the `AD CS` portion of this project. Since our root CA is already authorized, we are going to create a web server certificate template that can be used to issue digital certificates. First, let's duplicate the standard Windows web server template because it is locked down by default. I right clicked on `Certificate Templates` and selected `Manage`
<img width="962" height="599" alt="image" src="https://github.com/user-attachments/assets/170c6ecc-2267-4afe-b142-76d96fa39043" />

- Then, I started filling out the information as seen below
<img width="964" height="603" alt="image" src="https://github.com/user-attachments/assets/5d82f064-b0e2-4478-a0ab-b6160c22a376" />

- Same here as well. I checked the box because it lets us move the certificate over to Proxmox, Linux VMs, or Splunk later
<img width="961" height="603" alt="image" src="https://github.com/user-attachments/assets/22b2491b-9f45-412e-88ec-8ecfaaef7f64" />

- I made sure the `Authenticated Users` are enrolled
<img width="963" height="600" alt="image" src="https://github.com/user-attachments/assets/898a5cfc-a15a-45c3-a43a-1c43d28c1708" />

- Next, we click on `Certificate template to issue`
<img width="964" height="603" alt="image" src="https://github.com/user-attachments/assets/76c61111-863d-46e7-9d50-21baf9f70285" />

- We see our `Lab Web Server` certificate and click `OK`
<img width="964" height="602" alt="image" src="https://github.com/user-attachments/assets/854a7575-3e0e-4e81-a24f-780e52ea90de" />

## Requesting a Certificate
- Now that we have created a template, it is time to actually request a certificate from the CA. I used the Microsoft Management Console
<img width="957" height="596" alt="image" src="https://github.com/user-attachments/assets/e892612c-2143-4d7a-bb2d-2248a626cf35" />

- I then clicked `Certificates` > `Add >` > `Computer Account`. The reason for this is because the certificate is being issued to the domain controller rather than to my user account. This allows services running on `DC01`, such as a web server like `Apache`, to use the `DC01`'s certificate even when I am not logged in

<img width="958" height="601" alt="image" src="https://github.com/user-attachments/assets/c4647c30-073b-4b69-b7f9-ea2f152d3c58" />

- Then, we go back and select the following options

<img width="956" height="600" alt="image" src="https://github.com/user-attachments/assets/1cbcd4f7-3bf9-4666-bd32-e3380970b1e4" />

- I then clicked on the blue link to define the common name (identity) of this certificate so a browser actually knows what server it belongs to. If we didn't click this option, the CA would sign a blank certificate with no name that the browser wouldn't trust and it would be useless
<img width="961" height="600" alt="image" src="https://github.com/user-attachments/assets/57da9245-77a5-4533-b018-c4122ce2b3f6" />

- Filled in both these values
<img width="963" height="599" alt="image" src="https://github.com/user-attachments/assets/ae5c03f7-a1e9-41f1-afec-670aca247ad5" />

- We can see below we have successfully installed the certificate
<img width="963" height="599" alt="image" src="https://github.com/user-attachments/assets/1b1b4296-63ba-4cbb-83bf-06d75aeea92a" />

- We can see our beautiful certificate sitting here :D
<img width="961" height="598" alt="image" src="https://github.com/user-attachments/assets/22ed34b2-3f79-4c5d-be21-9b83a6a1b8b0" />
