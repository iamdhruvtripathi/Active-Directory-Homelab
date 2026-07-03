# Security Groups
- Now that we are finished with the setup of both the DC and Client VM and we also have created 4 users, OUs, it is time to create and learn about security groups

## What are Security Groups?
- Security groups are collections of users and computers to which permissions can be assigned. Instead of assigning permissions to each user individually, permissions are assigned to the security group, and all members of that group inherit those permissions. For example, if a security group is denied access to a particular folder or file and `Alisha` is a member of that group, she will not be able to access it unless I remove her from that specific group

## Making the Security Groups
- To make the group, I right clicked on the `HR` OU > `New` > `Group`
<img width="961" height="601" alt="image" src="https://github.com/user-attachments/assets/0e4bdb9c-26e2-4858-accd-46a1173c7622" />

- I named it `HR_Department` and left the default settings on

<img width="961" height="600" alt="image" src="https://github.com/user-attachments/assets/c0c3a284-8e3f-469a-9796-8bb585a6e94c" />

- I did the same thing for IT and now, we have two security groups

<img width="965" height="599" alt="image" src="https://github.com/user-attachments/assets/515c82f8-573f-4eb9-958d-4746bf4f0ed6" />

- We now need to add the users to the groups. I first did it for the `HR department`. I highlighted both `Alisha` and `Bob` to get ready to add them to the group

<img width="960" height="596" alt="image" src="https://github.com/user-attachments/assets/fa3288d5-cf45-4920-bcda-d5ca20cb205a" />

- I typed `HR_Department` and then clicked `Check Names` and we can see it is underlined. Then, I clicked `OK`

<img width="957" height="597" alt="image" src="https://github.com/user-attachments/assets/646999d9-c6c8-440c-9782-ac638fe9a2c0" />

- We can see it was successful and I repeated the same steps for the `IT Department`

<img width="961" height="599" alt="image" src="https://github.com/user-attachments/assets/c29d807c-f4c8-45c1-9253-68db2a686149" />

## Testing the configurations
- For testing purposes, let's create a shared folder named `IT_Secrets` and edit its permission properties. I right clicked and selected `Properties` > `Sharing` tab > `Advanced sharing` > checked the  `Share this folder` box > `Share permissions` where I removed everyone and only added the `IT Department`

<img width="962" height="601" alt="image" src="https://github.com/user-attachments/assets/8d75b8c5-8bf9-44b9-86f8-0e317a0e5a7b" />

- Then, I logged in as `Jack` and ran `192.168.1.200\\IT_Secrets` via the Run box (`Win + R`) and the folder was able to pop up for `Jack`

 <img width="960" height="596" alt="image" src="https://github.com/user-attachments/assets/d77b1a10-ad67-45c9-9012-cf7fa399d005" />

- However, `Alisha` was denied access because she is in the `HR Department` and not the `IT Department` which was assigned access to the shared folder
<img width="949" height="601" alt="image" src="https://github.com/user-attachments/assets/2a2a54bc-c3e5-415a-bb80-0e47aa09aa70" />

# Group Policy Objects (GPO)
- After finish setting up the security group, lets make some GPOs and assign them to the OUs

## What are Group Policy Objects?
- GPOs (Group Policy Objects) are collections of settings used to manage users and computers in Active Directory. They can be linked to sites, domains, and OUs. For example, you can create a GPO to set a specific desktop wallpaper or configure security settings for everyone in the IT OU

## Making the GPOs
- I first go to `Group Policy Management` > `Forest: homelab.local` > `Domains` > `homelab.local` > `Default Domain Policy` and right-clicked and selected `Edit`

<img width="961" height="600" alt="image" src="https://github.com/user-attachments/assets/ba58965e-e0c2-45cb-aec0-5d32cd81a513" />

- I will now configure password complexity and length. We go all the way down to `Password Policy` to configure the complexity and length

<img width="964" height="600" alt="image" src="https://github.com/user-attachments/assets/db454729-99f9-485c-8447-f8d911b1147f" />

- I changed the `Minimum password length` from `7` characteres to `12`

<img width="1052" height="653" alt="image" src="https://github.com/user-attachments/assets/a45c7969-fb57-4e00-a228-915e563a0d4a" />

- I also set the `Account lockout threshold` to `3` and clicked `OK` with the `Suggested Value Changes` as well

<img width="960" height="601" alt="image" src="https://github.com/user-attachments/assets/37aafd02-f026-4750-ad22-018fd95ee5aa" />

- Now, usually it takes 90 minutes for the GPOs to background refresh across a network and so we force the rules to be activated immediately via the `gpupdate /force` command. The message below was successful

<img width="961" height="600" alt="image" src="https://github.com/user-attachments/assets/6be6b2f4-6592-4008-a39e-3ea0c6f316c3" />

## Testing the configurations
- I put on my hacker hat and tried to brute-force the password to get into Alisha's account >:D. I had her username, but was her password?. As you can see, I tried 3+ passwords, and now I am locked out... D:

<img width="944" height="591" alt="image" src="https://github.com/user-attachments/assets/7918d57a-8e01-4b5c-a84b-1f4f1e88a48b" />

- To troubleshoot this now, I had to back to my DC and unlock that specific account. We can see when I clicked on `Properties` on Alisha, it says the account was locked and I can unlock it by checking the box

<img width="1138" height="709" alt="image" src="https://github.com/user-attachments/assets/eb81ed61-0774-4fb8-9eb6-483ebd39b216" />

- I was successfully able to login to her account

<img width="962" height="601" alt="image" src="https://github.com/user-attachments/assets/84775028-4f29-4879-8cb4-f87c7cb3bb48" />

## Creating GPOs for IT
- Let's create a new GPO for IT and this one is to disable the command prompt for the standard users so they can not run scripts or probe the network

<img width="964" height="607" alt="image" src="https://github.com/user-attachments/assets/fe55e184-e12e-4915-a4c4-18d4186b2f14" />

- Now, because this policy restricts user behavior, we will look inside the User Configuration folders. I navigated to the `System` folder and then found the setting for `Prevent access to the command prompt` and toggled it to be `Enabled`

<img width="960" height="601" alt="image" src="https://github.com/user-attachments/assets/52a85e49-15a5-4b69-8604-79de088e89e3" />

- We now test it on `Jack` and poor `Jack` is not able to access the command prompt

<img width="961" height="599" alt="image" src="https://github.com/user-attachments/assets/3c4278b4-f5ff-4b7f-932d-e4253a2c9b9e" />

## Security Filtering
- Security filtering is an Active Directory feature used to target a Group Policy Object (GPO) to specific users, computers, or security groups. It works by modifying the GPO's permissions so that only members of the selected groups have the authority to read and apply the policy. This allows administrators to narrow down a policy's scope so it does not automatically apply to every object inside an Organizational Unit (OU)

## Creating the Security filter
- Let's say that `Harmony` is a manager and that `Jack` is not. We want `Harmony` to have access to the command prompt but `Jack` should not. For this, I created a new group called `CMD_Blocked_Users`. Then, I went to the properties tab and only added `Jack`

<img width="962" height="599" alt="image" src="https://github.com/user-attachments/assets/78702444-90ce-4ff1-9abf-1270694e9b22" />

- I then added the `CMD_Blocked_Users` GPO underneath the `Security Filtering` tab
<img width="960" height="600" alt="image" src="https://github.com/user-attachments/assets/42d9735a-5249-48d5-ad20-58cb6d06cf93" />

- We can see here that poor `Jack` is not able to access the command still

<img width="961" height="596" alt="image" src="https://github.com/user-attachments/assets/4ece1b44-3398-4d3d-a3f0-623818662bfd" />

- However, `Harmony` can :D
<img width="958" height="599" alt="image" src="https://github.com/user-attachments/assets/dd7555b6-250c-463f-8814-615073997100" />
