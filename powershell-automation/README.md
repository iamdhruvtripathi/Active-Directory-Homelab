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
- Not only can we store one value, we can store an object. Let's create Alice's "profile"
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

## If-else statements
- We can use if-else statement to stimulate a basic security compliance check

<img width="1182" height="738" alt="image" src="https://github.com/user-attachments/assets/bb888c1f-f5bf-47fd-b9f7-960642a3e6dd" />

### Functions
- We can create a function to make our lives a bit easier
<img width="1183" height="737" alt="image" src="https://github.com/user-attachments/assets/83e97cb2-64d9-48fe-aef9-2b90aa302ac9" />

- Now that I have gotten the basics of Powershell, let's use it in our lab
### The `Remove-ADUser` command
- Remember `barry`? Well, he left the company and so we don't need his information anymore. Let's delete him from the Users folder. If we do the following

<img width="564" height="354" alt="image" src="https://github.com/user-attachments/assets/b3b06da1-b3a5-41c5-84c9-2e96f96122d9" />

- We can see `barry` is gone. We will miss you `barry`
<img width="1184" height="735" alt="image" src="https://github.com/user-attachments/assets/8dd31f76-e2fa-489f-893f-9857d237d175" />

### The `Set-ADUser` command
- Let's say `Jack` gets a promotion and we update his job title attribute. For now, `Jack` has no title or `Description` but we can set it
<img width="1184" height="735" alt="image" src="https://github.com/user-attachments/assets/a551d1db-988b-4315-bdad-d78ab7ff5d41" />

### Creating and Linking GPOs
- We don't even have to touch the Group Policy Management console because Powershell has a built in `GroupPolicy` module. Let's create a new GPO and link it to the HR folder. First, I created the GPO
<img width="1182" height="739" alt="image" src="https://github.com/user-attachments/assets/33c98835-25b5-4dea-b869-10beb4f32283" />

- We can then link it
<img width="1182" height="739" alt="image" src="https://github.com/user-attachments/assets/77b318f7-3733-48ca-9f2f-d802ba343f19" />

- We can see the GPO has been successfully created and linked
<img width="1184" height="739" alt="image" src="https://github.com/user-attachments/assets/dab93818-9178-4547-8681-566da97c53ee" />

- However, the GPO doesn't actually enforce anything per se so we have to configure the specific registry settings inside this particular GPO. Before that we should know what Windows registry is

## What is Windows Registry?
- The Windows Registry is a central database that stores configuration settings and options for the Windows operating system, hardware, and installed applications. It helps Windows and programs remember preferences, system settings, and device information

- To configure registry settings inside the GPO, we can use the following commands below and I will explain them. The first one is below
<img width="1180" height="737" alt="image" src="https://github.com/user-attachments/assets/6ff0deb2-4766-4f55-bade-f1958d0f2922" />

- Basically, this setting targets the `ScreenSaverActive` registry value in the Windows Registry, which controls whether the screen saver is enabled. Setting it to `1` turns the screen saver on. The idea is that if a user walks away from their computer, the screen saver can start after a period of inactivity, helping protect the system from unauthorized access. This registry value is stored under the registry path specified by the `-Key` parameter, which contains the user's desktop-related settings
- It's important to note that this setting doesn't actually lock the computer by itself. It only enables the screen saver. In most environments, it's paired with two additional registry settings where one that specifies how long Windows waits before starting the screen saver, and another that requires the user to enter their password when resuming from the screen saver

<img width="1182" height="740" alt="image" src="https://github.com/user-attachments/assets/10a8d807-c5df-4144-8e9e-cba7e3608725" />

- Now, this registry setting allows the computer to sit idle for 60 seconds before starting the screensaver. We have the last one below

<img width="1182" height="739" alt="image" src="https://github.com/user-attachments/assets/48552d6e-4add-4146-aec7-d31b7c174f6e" />

- This registry setting is basically saying that once the 60 seconds are up, if `ScreenSaverActive` is enabled and the screen saver is configured to require a password (value of `1` in this case means yes, it requires a password) on resume, the user's session is locked. When the user comes back, they must enter their password to continue

- Now, how are these registry settings actually applied? Whenever, a computer uses LDAP (Lightweight Directory Access Protocol) to talk to the AD DS to get which GPOs apply to that specific computer and where they are located. However, LDAP doesn't actually return the actual GPO data, instead the computer has the GPOs properties and a specific attribute called the `gPCFileSysPath`
- The `gPCFileSysPath` holds the actual file paths to where the GPOs actual data (registry settings, scripts, etc.) is located in a folder called `SYSVOL`
- After this, the computer uses `SMB` (Server Message Block) to fetch the GPO data and apply the changes to itself. The registry settings of the client are overwritten (not always but in this case they are) and the GPO overwrites it to the specified `-Key` path
