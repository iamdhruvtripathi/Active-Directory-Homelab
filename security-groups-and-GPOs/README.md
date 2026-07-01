# Security Groups
- Now that we are finished with the setup of both the DC and Client VM and we also have created 4 users, OUs, it is time to create and learn about security groups

## What are Security Groups?
- Security groups are collections of users and computers to which permissions can be assigned. Instead of assigning permissions to each user individually, permissions are assigned to the security group, and all members of that group inherit those permissions. For example, if a security group is denied access to a particular folder or file and Alisha is a member of that group, she will not be able to access it unless I remove her from that specific group

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
