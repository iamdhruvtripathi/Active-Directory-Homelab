# Windows Server Roles (Part 2)
- We are now going to configure the installed roles including the File Server and Print Server

## Building the shared network folder
- Now, we are done with the DHCP configurations and setting up the AD CS, let's take a look at the File server. The File server is activated but we need to build a shared network folder on the DC. This will act as an enterprise style file share that users such as `Jack` can instantly access right from their Windows 11 file explorer. First, I created a folder called `CompanyShare`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/444b4905-4cf3-4a81-968f-84a4d3a790e4" />
</p>

- Then, to start sharing the file, I clicked on `File and Storage Services`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/a948fc06-6047-4ea5-a901-50daa9c2bf3b" />
</p>

- Then, I started filling out the information from the wizard (configuring the `Share location` here)

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/d1b5d149-acde-4f18-8d33-7df61d670192" />
</p>

- As we can see, the share was successfully created. You can also notice the protocol being used is `SMB` which is basically the protocol used to share files and folders over a network. In this case, the server is sharing the folder `CompanyShares` and clients can connect using SMB to `\\DC01\CompanyShares`. The client can read, write, or modify files depending on the permissions granted

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/35235705-a280-443d-bad7-09ae5ad64c84" />
</p>

## Testing if the file share works
- Now, we go into the Windows client and see if the file share is working properly. I opened `File Explorer` and went to `This PC`. I then clicked on `Map Network Drive`. Basically what this does is it assigns a local drive letter (such as `S:`) to a shared folder located on another computer or server. It acts as a permanent shortcut, letting anyone browse and open remote files as if they were stored directly on your own hard drive

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/a0993d47-a2cf-4ff1-9a64-47f030aa1dd5" />
</p>

- I chose the drive letter (`S:`) and the network folder path

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/13cfd236-91d3-43f1-81b8-3302a2a71000" />
</p>

- Boom!, we are successfully able to access the share folder

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/5128afcf-5ce2-4c0e-af56-379715c0a259" />
</p>

## Setting permissions on the folder
- Now, let's set some restrictions on who can access the folder because right now, anyone can do anything. I created a new OU named `Company` and then created a new group called `IT_Staff` inside it and included `Jack` inside

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/d9a7e5ff-b05d-40aa-a409-817ff37e1a89" />
</p>

- I then went to edit the permissions of the shared folder

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/e08b44c7-0fda-4234-bfef-32992f9b724c" />
</p>

- I clicked `Disable inheritance` and then the first option on the popup. What we are doing here is breaking the link between the `C:/` drive and the `CompanyShare` folder. Normally, any permissions applied on the parent folder/drive are passed down to the child folder. Furthermore, the convert option basically means that we are keeping all of the original permissions but can edit them manually

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/350f59ec-2b7e-4e5e-8e2b-0b0c6b05b3d9" />
</p>

- I then added the `IT_Staff` group here

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/7c2083ee-2666-41ba-8756-76ab04c5d474" />
</p>

- Note: I also add to get rid of the two `Users (HOMELAB\Users)` permissions for this to work

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/1a826580-3326-4b75-bfdb-a3875b258668" />
</p>

- Let's see if `Jack` is able to access the shared folder and he is successfully

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/4f4720ba-a3ef-4445-87f6-6f5815371ed8" />
</p>

- But `Harmony` wasn't able to access it

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/f21573b7-1b7c-47f1-80d8-593f7c7f80f7" />
</p>

## Creating and Sharing the Printer
- Now, I am going to work on the Print Server portion of this project. First, we actually need to create a ghost "printer". What I mean by this is I don't actually have a real printer, so we will need to create a fake one just for learning purposes. First let's click on `Print management` and right click and select `Add printer` and then the `Network Printer Installation Wizard` pops up

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/1ab57200-e512-4e0a-bb66-70e0b83310c0" />
</p>

- I then clicked on `Add a new printer using an existing port`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/72a0aa09-2905-40ac-9269-8c0702468e59" />
</p>

- I then clicked on `Install a new driver`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/244bb467-3e06-4e2a-9d2a-65ec46a1b432" />
</p>

- Clicking through the options I arrived here and selected a generic Microsoft printer driver to add

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/a8b0a509-1746-46d6-86bf-7007d5ab3bdf" />
</p>

- Added all the necessary information

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/ca19f608-1a0e-4b89-95df-a52fbc241918" />
</p>

- We have successfully added the printer as seen below

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/d6ea1279-26a7-4f73-ae0e-630f23bb8ef6" />
</p>

## Connecting to the Network Printer from the Windows Client
- Now, we can see how an enterprise user connects to it over the network. Below, we can see the printer being shared successfully. Note that you have to open the run box and type `\\10.10.10.10` to see the shared resources

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/266be552-3c3a-4dbf-964e-0dfd6382bc7f" />
</p>

- I typed a message into `Notepad` and then hit `Ctrl + P` and we can see our printer being listed as one of the options

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/a3105a8f-a7ac-4674-9261-7f89ed3e6c5d" />
</p>

- I clicked `Print` and we can see the print job listed in the queue when we hop over to the DC. Note that it is saying `Error` but that is normal because there is no printer actually plugged into the port on my Proxmox server host
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/6ade6c09-70b4-44ac-88a8-05f9e49ec4f3" />
</p>
