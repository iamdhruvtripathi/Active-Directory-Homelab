# Splunk
- So now it is time to pull out the big guns and add Splunk to this project. The idea is that since we already have the two Windows VM setup, we can use Splunk's Universal Forwarder, a light weight agent and install it directly on both the domain controller and the Windows 11 client and send Windows Event log and Sysmon logs to Splunk
## What is Splunk?
- Splunk Enterprise Security (Splunk ES) is a SIEM platform that collects and analyzes security logs from systems, applications, and network devices. It centralizes security data to help detect threats, generate alerts, and support incident investigations. It also provides dashboards and powerful search capabilities, enabling security teams to monitor and respond to potential security incidents efficiently

### Dual-NIC for Splunk VM
- Now, one issue is that because we moved the Windows VMs back onto the isolated `vmbr1` bridge, they are completely cut off from the main network bridge where my Splunk server lives. Since I can access the Splunk Web UI on my Mac, my Splunk VM is still sitting on `vmbr0`, while the Windows VMs are in the `vmbr1` sandbox. Essentially, they are on two completely different virtual switches
- To solve this issue, my Splunk VM will have two NICs where one is named `ens18` which plugs into my home network (`vmbr0`) so I can access Splunk's dashboard from my Mac and `ens19` which plugs into my isolated sandbox (`vmbr1`) so it can collect logs from the Windows VMs
  
- First, I added another network device. As you can see below, one NIC (`ens18`) will be connected to `vmbr0` and the other one (`ens19`) will be connected to `vmbr1`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/afbb4af4-58ec-4121-b2d6-df666f0a8de3" />
</p>

- Then, I opened up the network configuration file

```
sudo nano /etc/netplan/00-installer-config.yaml
```
- As you can see, one NIC (`ens18`) has its IP address assigned by DHCP (my home router) and the other NIC (`ens19`) has its IP address set to static by me to `10.10.10.15/24`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/7987078c-26d0-421b-bb21-51a38fe5c4e1" />
</p>

- Then run the command below to apply the changes
```
sudo netplan apply
```

### Configuring Splunk receiving port

- Now, we have to make sure that Splunk is listening for incoming log data on a specific port. I configured port `9997` for this. This is because port `9997` is the standard enterprise default port where all Splunk forwarders send their raw telemetry

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/0df83409-6d87-443b-8f8d-6658536447fa" />
</p>

### Installing the Splunk Universal Forwarder
- For both the domain controller and the Windows 11 client VM, I installed the Splunk Universal Forwarder from Splunk's official site. Since my VM's do not have internet access, I just downloaded the `.msi` file from Splunk's official site and then converted it to an `.iso` file. Note that I downloaded the `64-bit` version

- To convert from `.msi` to `.iso`, I used this [link](https://www.petenetlive.com/KB/Article/0001554)

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/df85f9b1-5a65-4399-80b0-0add27d19913" />
</p>

- I then mounted the `.iso` file to the virtual CD/DVD drive in the `Hardware` tab on Proxmox and then uploaded it to the `C:\Tools` folder already present

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/21b39f9c-71eb-449d-986e-3938ed8d3b77" />
</p>

- I also made a `C:\Tools` folder on the Windows 11 VM client and then did the same thing as above
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/ab6eb27f-ce47-463c-b838-da3d8db2481f" />
</p>

### Pointing Windows to Splunk
- In this part, we will make sure that the logs are actually being sent to where Splunk is listening on (port `9997`). I clicked on `splunk-forwarder.msi` and went through the installation wizard. We are here at the first screen

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/64f72a2e-03b1-4ac7-8067-1fb0121168a6" />
</p>

- Setting my credentials here...

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/1e1ca64b-0413-4d85-8c4f-74003937b0fd" />
</p>

- This is most important part as we want our logs going to Splunk

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/fa3eb14a-02d1-4861-8af5-9ffea0157ee0" />
</p>

- BOOM!, we have successfully installed the Splunk Universal Forwarder

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/d1b789e9-7d6a-4f0d-b1dd-ff827e156ba5" />
</p>

- Note: I did the same exact process on the Windows 11 VM Client

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/2e331822-e087-44ac-ba38-ec19e5678a23" />
</p>

### Sending Window Event & Sysmon Logs
- I used Notepad to point Splunk to the logs. I opened up Notepad and selected `Run as Administrator` and navigated to `C:\Program Files\SplunkUniversalForwarder\etc\system\local\`. I selected `inputs.conf`. Note that I had to create this file as it did not exist previously

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/5f60dc40-0946-4d35-aaf8-898b2f82eb23" />
</p>

- I pasted this into notepad. Basically, this collects Sysmon, Windows Security, System and Application logs
```
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = false
renderXml = true
index = main
sourcetype = XmlWinEventLog:Sysmon

[WinEventLog://Security]
disabled = false
index = main

[WinEventLog://System]
disabled = false
index = main

[WinEventLog://Application]
disabled = false
index = main
```

- Note: I did all of these steps for both the Windows 11 Client VM and the Domain Controller. Note that you have to do `Restart-Service SplunkForwarder` in Powershell for it to actually work
- BOOM!, we are able to see 30,000+ events into Splunk including Window Event logs as well Sysmon logs

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/2656c64d-bc95-4e5c-b02d-8346716b4508" />
</p>

## Splunk Log Analysis (Window Event logs)
- Now, it is time to generate some logs that we can analyze deeply and see if we can understand what is going on. For the sake of this lab, I am going to generate some of the most common Window Event logs for learning purposes as well as Sysmon logs

### Event ID `4720`
- Let's test for the Windows Event Log ID `4720` which is generated whenever a new user account is successfully created in Active Directory and it identifies who performed the action and the details of the newly created account. For this, I created a new account in the domain controller

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/4083d6f4-6365-4dd8-8a77-3a40c410a435" />
</p>

- We can confirm in `Events Viewer` that an account named `Hanna` was created

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/134b05c3-5581-41ee-be00-92a7d5d1af2a" />
</p>

- Look at that... we see the Security log in Splunk. Lets dissect this log

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/2ce20551-0cc3-478c-a6ea-3642cfe5b381" />
</p>

- This log clearly came today, as of `7/16/2026` and the `EventCode` is `4720` which we know that someone created a new user account. It was created on the domain controller as denoted by `ComputerName` and the DC's name is `DCO1.homelab.local`. We can see the `Message` and know what this log is about. We know that from the `Subject` line, it was the `Administrator` who created this account from `Account Name` in the domain `HOMELAB`. Furthermore, the `Security ID` ends in `-500` which means this was the built-in admin account. For the new account created under `New Account`, it was `hanna` in the `HOMELAB` domain. We can also see the new account's `Attributes` such as their `SAM Account Name`, `Display Name` which is `hanna` and `Hanna` respectively and `User Principal Number` which is `hanna@homelab.local`. We also know the `Security ID` ends in `-1118` which means this was a new identity created in the domain. The `Primary Group ID` is `513` which means `Hanna` is a domain user. `512` would be for a domain admin. For the `UAC`, we can see underneath that the account is disabled which happens by default until configured, the password is not required and that this is a typical user account as indicated by `Normal Account`

- If we expand the log, we can see both the `Field` and `Value` columns

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/5faa02be-d37a-4cc9-afe2-678268841d3a" />
</p>

### Event ID `4771`
- Now, let's look for another Event ID, namely the Event ID `4771` which is associated with Kerberos authentication. This one specifically records a failed Kerberos pre-authentication attempt such as me entering the wrong password 3+ times and then getting locked out (Whoops!). Well... I succeeded at that

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/b97168a0-9bb9-499b-ac87-b0654ec82356" />
</p>

- I looked in `Events Viewer` to confirm that this log did get generated. We can see that it was indeed `Alisha` that typed in her password wrong 3+ times

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/10083df6-55c4-48f7-b139-1eb8be4e41a6" />
</p>

- Yes, we can see it was `Alisha` who failed to login

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/feb30fcd-bfab-4af0-a904-7ac0f043b106" />
</p>

- Note that most fields are the same but we can see that the `EventCode` is `4771` now. It was on the Domain controller and the `Keywords` is `Audit Failure` instead of `Audit Success` unlike last time. The message is `Kereberos pre-authenication failed` which means `Alisha` typed in her password wrong. We can see in the service name that it was `krbtgt/HOMELAB` meaning the KDC service account. When the user logins, they ask the `krbtgt` service for a TGT or a ticket granting ticket. We can see the `Client Address` which we set the IP address as `10.10.10.50` in previous parts of this lab. The `Failure Code` is `0x12` which means that the user account exists but the user typed in the wrong password. If it was `0x6` that means the username itself does not exist

### Event ID `4726`
- Now, we can do Event ID `4726` which is means a user account is deleted. Let's delete that account we had created for `Hanna`. We can see in `Event Viewer` that `Hanna`'s account was in fact deleted

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/9784e2cc-24f4-465a-b429-97ea2909d20b" />
</p>

- Now, we can see in Splunk the same log and examine it

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/504aea85-b073-40ad-9aae-a02edd6f066a" />
</p>

- This one was a bit short but we can tell it was `Hanna`'s account was deleted by the administrator's account. The `EventCode` was 4726 as expected and the `Message` confirms our thoughts
### Event ID `1102`
- Lastly, we can see Event ID `1102` which gets created when the system's security audit log is manually cleared. As always, lets verify in `Event Viewer` and we can see the specific log

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/d8a07d12-43dc-4a80-8304-8f15197e578d" />
</p>

- Then, in Splunk we can see the same information as well

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/38f7891e-893e-44e4-8eca-8a4950efa574" />
</p>

- We can tell by the `EventCode` which is `1102`, that it happened on the Domain Controller, it was a `Audit Success`, the `TaskCategory` this time was a `Log clear` and the `Message` clearly states what happened. Finally, we can see who did it in the `Subject` area which was the built in administrator account

## Splunk Log Analysis (Sysmon logs)
- We can also search for Sysmon logs as well. Let's test `Event ID 1`, `Event ID 3`, `Event ID` and `Event ID 22`

### `Event ID 1`
- As we know, this event ID tracks process creation. Let's test these two commands below
```
whoami
```
```
net user administrator /domain
```
- Since Sysmon logs in Splunk can look very messy, what we can do is filter via regex and knowing the field names we can search specifically and make it into a table for a nicer display
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/5e9aa145-5af8-4f31-95c7-d720783d8485" />
</p>

- I had ran `whoami` and so let's write the splunk query to see the specific log. You can see below it is filtered perfectly and we know who it was by looking at the top log as well as what the command was and the parent command

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/d802c167-8d94-4074-a58e-eede7cf9483c" />
</p>

- Below is the SPL
```
index=main sourcetype="XmlWinEventLog:Sysmon" whoami
| rex field=_raw "<EventID>(?<Event_ID>[^<]+)"
| rex field=_raw "<Computer>(?<Computer_Name>[^<]+)"
| rex field=_raw "<Data Name='Image'>(?<Process_Path>[^<]+)"
| rex field=_raw "<Data Name='CommandLine'>(?<Command_Line>[^<]+)"
| rex field=_raw "<Data Name='User'>(?<User_Account>[^<]+)"
| rex field=_raw "<Data Name='CommandLine'>(?<Command_Line>[^<]+)"
| rex field=_raw "<Data Name='ParentCommandLine'>(?<Parent_Command_Line>[^<]+)"
| table _time, Computer_Name, User_Account, Process_Path, Command_Line, Event_ID, Command_Line, Parent_Command_Line
| sort - _time
```

- Basically, what this does is it looks for Sysmon Windows event logs that contain the command `whoami`. It then uses the `rex` command to extract specific values from the raw XML event (`field=_raw`) using regular expressions. For example, in the expression
```
<EventID>(?<Event_ID>[^<]+)
```
- it looks for the `<EventID>` tag, then uses `(?<Event_ID>)` to create a new field named `Event_ID`. The `?<Event_ID>` part tells `rex` to save the matching text into a field with that name. The `[^<]+` part captures one or more characters that are not `<` and so it grabs everything after the `<EventID>` tag until it reaches the next `<` character. That captured value is then stored in the `Event_ID` field

- We can see the same results here for `net user administrator /domain`
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/648ed6c4-9bdb-407b-90d5-52831e7d3402" />
</p>

### `Event ID 3`
- This event ID tracks whenever a TCP or UDP connection is created or detected. I locally created a Python server and connected to it via my web browser. We can see below the other connections but I wanted to highlight that it was our browser on `10.10.10.10` on the DC connecting to the Python server on `10.10.10.10` because the Python server was running locally, essentially we are connecting to ourselves. The destination port confirms that we are running the Python server on port `80` and that it was a `python.exe` process running
