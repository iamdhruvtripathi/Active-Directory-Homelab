# Windows Server Roles (Part 1)
- We are now going to install several Windows roles including the Active Directory Certificate Services (AD CS) & DHCP server

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
