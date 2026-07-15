# Windows Server Roles (Part 1)
- We are now going to install several Windows roles including the Active Directory Certificate Services (AD CS), DHCP server, File Server and Print Server
- Note: For organization, the AD DS and DHCP Server installation and configuration are on this `README.md` and the File Server and Print Server are on this [README.md](https://github.com/iamdhruvtripathi/Active-Directory-Homelab/blob/main/file-and-print-server/README.md)

## Installing the AD CS & DHCP
- First, I clicked on the `Add Roles and Features`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/343242d0-f9b4-44b1-9b82-45359951d18a" />
</p>

- As you can see, I selected the roles I wanted to install

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/3f071529-b8d1-4289-81c5-7c00691ece19" />
</p>

- I clicked through the options leaving them as default
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/58df2921-ee7e-4155-8664-060facbd158d" />
</p>

- Same here...
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/6ee73065-f844-4e73-ade3-efe70473142a" />
</p>

- And then we click `Install` :D
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/9dac2269-1552-4cd0-9177-ac3c5d76f8fa" />
</p>

- Now that the installation succeeded, we have to configure the `AD CS` and the `DHCP` server

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/32a664e7-cde0-4602-9668-621e2be47a50" />
</p>

- I successfully installed and configured the `DHCP` server as seen below

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/a6a22c9c-5671-4d66-9979-27ac93930203" />
</p>

- Now, I configured the `AD CS` and for this option I clicked `Enterprise CA`. The reason for this is because it seamlessly integrates with `AD DS` and it can look for any users/computers objects and verifies their identity to issue any certificates

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/db8f237b-8ca7-4a83-95ba-536592729a88" />
</p>

- I clicked `Create a new private key`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/2e4c9277-f244-4882-9738-3681321dcca1" />
</p>

- After clicking through all the options, I successfully configured the `AD CS`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/5638244d-a4e0-4ef8-8515-c35b79c88847" />
</p>

## Configuring the DHCP Scope
- Now, we have to configure the DHCP Scope (this is just the range of IP addresses the DC will hand out to the virtual machines so they can connect automatically without me having to configure static IPs manually). Now, here we are in the screen presented below

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/99f0db41-fad8-40cc-a3e4-7ad949f77784" />
</p>

- I set the range from `10.10.10.50` to `10.10.10.100`. The reason for this range is just to keep the DHCP pool separate from manually assigned addresses causing no IP conflicts

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/da1f9a6d-f70f-4e73-b325-e252684117cc" />
</p>

- Here, I set the DNS server address (which is the DC itself)

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/2b544b54-4147-4470-805e-30dd3995a4dc" />
</p>

- We have successfully configured the DHCP scope

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/18f49038-556a-4d68-a4e3-1e1516ab2427" />
</p>

## Testing the DHCP configurations
- Now, it's time to test what we configured by using the Windows client. We can see here that is was the IP address I manually assigned (`10.10.10.11`)

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/895c830d-927b-4f44-a096-33f6d762dfeb" />
</p>

- Now, since I hardcoded the IP address, we need to allow the network card to dynamically grab or release IP addresses. I went configure the network connection and clicked on `Obtain an IP address automatically`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/06c60503-e23d-4299-bb35-1b66095b0c1a" />
</p>

- We can see it automatically got assigned an IP address (`10.10.10.50`)

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/0ff9dd8c-97ad-4d78-9b00-0e6d33bf54ee" />
</p>

- If we do `ipconfig /all`, we can see it was assigned by the DHCP server and not manually assigned. You can see I also set the lease for `8` days

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/455fa71a-881b-40c1-adfb-c03bd3e86821" />
</p>

## Creating a custom Web Server certificate template
- I am now going to be working on the `AD CS` portion of this project. Since our root CA is already authorized, we are going to create a web server certificate template that can be used to issue digital certificates. First, let's duplicate the standard Windows web server template because it is locked down by default. I right clicked on `Certificate Templates` and selected `Manage`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/170c6ecc-2267-4afe-b142-76d96fa39043" />
</p>

- Then, I started filling out the information as seen below

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/5d82f064-b0e2-4478-a0ab-b6160c22a376" />
</p>

- Same here as well. I checked the box because it lets us move the certificate over to Proxmox, Linux VMs, or Splunk later

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/22b2491b-9f45-412e-88ec-8ecfaaef7f64" />
</p>

- I made sure the `Authenticated Users` are enrolled

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/898a5cfc-a15a-45c3-a43a-1c43d28c1708" />
</p>

- Next, we click on `Certificate template to issue`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/76c61111-863d-46e7-9d50-21baf9f70285" />
</p>

- We see our `Lab Web Server` certificate and click `OK`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/854a7575-3e0e-4e81-a24f-780e52ea90de" />
</p>

## Requesting a Certificate
- Now that we have created a template, it is time to actually request a certificate from the CA. I used the Microsoft Management Console

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/e892612c-2143-4d7a-bb2d-2248a626cf35" />
</p>

- I then clicked `Certificates` > `Add >` > `Computer Account`. The reason for this is because the certificate is being issued to the domain controller rather than to my user account. This allows services running on `DC01`, such as a web server like `Apache`, to use the `DC01`'s certificate even when I am not logged in

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/c4647c30-073b-4b69-b7f9-ea2f152d3c58" />
</p>

- Then, we go back and select the following options

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/1cbcd4f7-3bf9-4666-bd32-e3380970b1e4" />
</p>

- I then clicked on the blue link to define the common name (identity) of this certificate so a browser actually knows what server it belongs to. If we didn't click this option, the CA would sign a blank certificate with no name that the browser wouldn't trust and it would be useless

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/57da9245-77a5-4533-b018-c4122ce2b3f6" />
</p>

- Filled in both these values

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/b10a07a7-345d-4324-bb00-467a184b4fb2" />
</p>

- We can see below we have successfully installed the certificate

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/1b1b4296-63ba-4cbb-83bf-06d75aeea92a" />
</p>

- We can see our beautiful certificate sitting here :D

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/499c9c10-086a-43f2-ba26-02bb3c69b891" />
</p>
