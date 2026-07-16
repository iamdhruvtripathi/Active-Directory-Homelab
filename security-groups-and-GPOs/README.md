# Security Groups
- Now that we are finished with the setup of both the DC and Client VM and we also have created 4 users, OUs, it is time to create and learn about security groups

## What are Security Groups?
- Security groups are collections of users and computers to which permissions can be assigned. Instead of assigning permissions to each user individually, permissions are assigned to the security group, and all members of that group inherit those permissions. For example, if a security group is denied access to a particular folder or file and `Alisha` is a member of that group, she will not be able to access it unless I remove her from that specific group

## Making the Security Groups
- To make the group, I right clicked on the `HR` OU > `New` > `Group`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/0e4bdb9c-26e2-4858-accd-46a1173c7622" />
</p>

- I named it `HR_Department` and left the default settings on

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/c0c3a284-8e3f-469a-9796-8bb585a6e94c" />
</p>

- I did the same thing for IT and now, we have two security groups

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/515c82f8-573f-4eb9-958d-4746bf4f0ed6" />
</p>

- We now need to add the users to the groups. I first did it for the `HR department`. I highlighted both `Alisha` and `Bob` to get ready to add them to the group

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/fa3288d5-cf45-4920-bcda-d5ca20cb205a" />
</p>

- I typed `HR_Department` and then clicked `Check Names` and we can see it is underlined. Then, I clicked `OK`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/646999d9-c6c8-440c-9782-ac638fe9a2c0" />
</p>

- We can see it was successful and I repeated the same steps for the `IT Department`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/c29d807c-f4c8-45c1-9253-68db2a686149" />
</p>

## Testing the configurations
- For testing purposes, let's create a shared folder named `IT_Secrets` and edit its permission properties. I right clicked and selected `Properties` > `Sharing` tab > `Advanced sharing` > checked the  `Share this folder` box > `Share permissions` where I removed everyone and only added the `IT Department`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/8d75b8c5-8bf9-44b9-86f8-0e317a0e5a7b" />
</p>

- Then, I logged in as `Jack` and ran `\\10.10.10.10\\IT_Secrets` via the Run box (`Win + R`) and the folder was able to pop up for `Jack`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/e5bbb026-6577-4892-b2fc-0ef1627fbf43" />
</p>

- However, `Alisha` was denied access because she is in the `HR Department` and not the `IT Department` which was assigned access to the shared folder

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/5ce90133-d465-42b3-9700-84065582ce6f" />
</p>

# Group Policy Objects (GPO)
- After finish setting up the security group, lets make some GPOs and assign them to the OUs

## What are Group Policy Objects?
- GPOs (Group Policy Objects) are collections of settings used to manage users and computers in Active Directory. They can be linked to sites, domains, and OUs. For example, you can create a GPO to set a specific desktop wallpaper or configure security settings for everyone in the IT OU

## Making the GPOs
- I first go to `Group Policy Management` > `Forest: homelab.local` > `Domains` > `homelab.local` > `Default Domain Policy` and right-clicked and selected `Edit`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/ba58965e-e0c2-45cb-aec0-5d32cd81a513" />
</p>

- I will now configure password complexity and length. We go all the way down to `Password Policy` to configure the complexity and length

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/db454729-99f9-485c-8447-f8d911b1147f" />
</p>

- I changed the `Minimum password length` from `7` characteres to `12`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/a45c7969-fb57-4e00-a228-915e563a0d4a" />
</p>

- I also set the `Account lockout threshold` to `3` and clicked `OK` with the `Suggested Value Changes` as well

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/37aafd02-f026-4750-ad22-018fd95ee5aa" />
</p>

- Now, usually it takes 90 minutes for the GPOs to background refresh across a network and so we force the rules to be activated immediately via the `gpupdate /force` command. The message below was successful

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/6be6b2f4-6592-4008-a39e-3ea0c6f316c3" />
</p>

## Testing the configurations
- I put on my hacker hat and tried to brute-force the password to get into Alisha's account >:D. I had her username, but was her password?. As you can see, I tried 3+ passwords, and now I am locked out... D:

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/7918d57a-8e01-4b5c-a84b-1f4f1e88a48b" />
</p>

- To troubleshoot this now, I had to back to my DC and unlock that specific account. We can see when I clicked on `Properties` on Alisha, it says the account was locked and I can unlock it by checking the box

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/eb81ed61-0774-4fb8-9eb6-483ebd39b216" />
</p>

- I was successfully able to login to her account

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/84775028-4f29-4879-8cb4-f87c7cb3bb48" />
</p>

## Creating GPOs for IT
- Let's create a new GPO for IT and this one is to disable the command prompt for the standard users so they can not run scripts or probe the network

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/fe55e184-e12e-4915-a4c4-18d4186b2f14" />
</p>

- Now, because this policy restricts user behavior, we will look inside the User Configuration folders. I navigated to the `System` folder and then found the setting for `Prevent access to the command prompt` and toggled it to be `Enabled`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/52a85e49-15a5-4b69-8604-79de088e89e3" />
</p>

- We now test it on `Jack` and poor `Jack` is not able to access the command prompt

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/3c4278b4-f5ff-4b7f-932d-e4253a2c9b9e" />
</p>

## Security Filtering
- Security filtering is an Active Directory feature used to target a Group Policy Object (GPO) to specific users, computers, or security groups. It works by modifying the GPO's permissions so that only members of the selected groups have the authority to read and apply the policy. This allows administrators to narrow down a policy's scope so it does not automatically apply to every object inside an Organizational Unit (OU)

## Creating the Security filter
- Let's say that `Harmony` is a manager and that `Jack` is not. We want `Harmony` to have access to the command prompt but `Jack` should not. For this, I created a new group called `CMD_Blocked_Users`. Then, I went to the properties tab and only added `Jack`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/78702444-90ce-4ff1-9abf-1270694e9b22" />
</p>

- I then added the `CMD_Blocked_Users` GPO underneath the `Security Filtering` tab

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/42d9735a-5249-48d5-ad20-58cb6d06cf93" />
</p>

- We can see here that poor `Jack` is not able to access the command still

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/4ece1b44-3398-4d3d-a3f0-623818662bfd" />
</p>

- However, `Harmony` can :D

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/dd7555b6-250c-463f-8814-615073997100" />
</p>
