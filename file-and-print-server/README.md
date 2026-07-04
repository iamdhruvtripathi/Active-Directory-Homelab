# Windows Server Roles (Part 2)
- We are now going to configure the installed roles including the File Server and Print Server

## Building the shared network folder
- Now, we are done with the DHCP configurations and setting up the AD CS, let's take a look at the File server. The File server is activated but we need to build a shared network folder on the DC. This will act as an enterprise style file share that users such as `Jack` can instantly access right from their Windows 11 file explorer. First, I created a folder called `CompanyShare`

  <img width="919" height="573" alt="image" src="https://github.com/user-attachments/assets/444b4905-4cf3-4a81-968f-84a4d3a790e4" />

- Then, to start sharing the file, I clicked on `File and Storage Services`

<img width="963" height="601" alt="image" src="https://github.com/user-attachments/assets/fdc9e60f-b243-4762-895f-293509118565" />
 
- Then, I started filling out the information from the wizard (configuring the `Share location` here)

<img width="961" height="601" alt="image" src="https://github.com/user-attachments/assets/d1b5d149-acde-4f18-8d33-7df61d670192" />

- As we can see, the share was successfully created. You can also notice the protocol being used is `SMB` which is basically the protocol used to share files and folders over a network. In this case, the server is sharing the folder `CompanyShares` and clients can connect using SMB to `\\DC01\CompanyShares`. The client can read, write, or modify files depending on the permissions granted

<img width="963" height="604" alt="image" src="https://github.com/user-attachments/assets/35235705-a280-443d-bad7-09ae5ad64c84" />

## Testing if the file share works
- Now, we go into the Windows client and see if the file share is working properly. I opened `File Explorer` and went to `This PC`. I then clicked on `Map Network Drive`. Basically what this does is it assigns a local drive letter (such as `S:`) to a shared folder located on another computer or server. It acts as a permanent shortcut, letting anyone browse and open remote files as if they were stored directly on your own hard drive

<img width="961" height="599" alt="image" src="https://github.com/user-attachments/assets/a0993d47-a2cf-4ff1-9a64-47f030aa1dd5" />

- I chose the drive letter (`S:`) and the network folder path
<img width="965" height="600" alt="image" src="https://github.com/user-attachments/assets/db8baeb5-0ad6-4e5c-b76c-48942b7d3d7f" />

- Boom!, we are successfully able to access the share folder

<img width="961" height="601" alt="image" src="https://github.com/user-attachments/assets/56155012-38e0-4818-9172-e4c6a3045b70" />


