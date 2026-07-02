# Powershell
- This part will makes our lives easier because we will be working with Powershell and learning the basics of how to run commands in Powershell

## What is Powershell?
- PowerShell is a cross-platform task automation framework consisting of a command-line shell, an object-oriented scripting language, and a configuration management framework developed by Microsoft

### Variables in Powershell
- Let's create two variables, `name` and `age` and assign them to specific values. We can output those variables via `Write-Host [variable name]`

<img width="1185" height="739" alt="image" src="https://github.com/user-attachments/assets/52211411-ecd3-4839-8ca9-3b94468cd80e" />

- I created some more variables just for fun :D
<img width="1181" height="737" alt="image" src="https://github.com/user-attachments/assets/6a2da0ed-78ae-4df6-bab5-8813de3f7259" />

### Creating Objects
- Not only can we create store one value, we can store an object. Let's create Alice's "profile"
<img width="1181" height="739" alt="image" src="https://github.com/user-attachments/assets/2ef85356-6602-474a-a9ff-d2fe9b43b7d0" />

### Loops
- Let's write a loop, so I don't have to write the same commands over and over again
<img width="1181" height="740" alt="image" src="https://github.com/user-attachments/assets/6072c5eb-7090-4a92-92a8-7f6fcb865d0d" />

### The `Get-ADUser` command
- We can list every single user currently sitting inside the active directory domain via the below command and see the result
<img width="1185" height="739" alt="image" src="https://github.com/user-attachments/assets/fb8bdcbc-60fc-48fa-ba92-27d077212cc1" />

- Let's say we only wanted a particular person... `Jack` and when his account was created. We can use the command below instead
<img width="1119" height="674" alt="image" src="https://github.com/user-attachments/assets/3f80f185-cece-41ec-abfd-529f6860c81a" />

### The `New-ADUser` command
- Now, we can create new users without even having to touch the GUI
<img width="1181" height="740" alt="image" src="https://github.com/user-attachments/assets/b9078f7c-e234-4c89-bdca-4c0f04915521" />

- When I back to the `IT` OU, we can see that `Barry Allen` has officially joined the `IT department`
<img width="1074" height="671" alt="image" src="https://github.com/user-attachments/assets/e6b93f76-0a71-4562-bf58-439614b6d0c6" />

### Pipelines
- This character `|` takes the output of the command on the left and pipes it directly into the command on the right as its input. An example is seen below

<img width="1183" height="737" alt="image" src="https://github.com/user-attachments/assets/ec74578e-4621-4f7d-b336-4f6b37250c8a" />

### The `ForEach-Object` command
- I know we already looked at loops but this particular one works well with `|` pipes
<img width="1182" height="738" alt="image" src="https://github.com/user-attachments/assets/d1cbb875-0f2b-4800-9cdf-b45695655c2d" />
- Note: The `$_` means the current item it is at
