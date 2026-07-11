# Sysmon
- Since we already have looked at some of the most important Windows Events log, we shall also take a look at some captured advanced telemetry like `Event ID 1` or `Event ID 3`

## What is Sysmon?
- Sysmon (System Monitor) is an advanced Windows tool that captures advanced telemetry. While Windows has built-in logs, Sysmon provides much more detail tracking exactly which programs (processes) are running, what files they modify, which servers they communicate with across the internet, and so much more

## Installing Sysmon
- First, we have to install Sysmon. I headed over Microsoft's official Sysinternals page and downloaded the `Sysmon.zip` package. This [link](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon) was useful

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/5d334c61-d377-445b-bd13-e43c15d9d897" />
</p>

- Another thing is we can use SwiftOnSecurity's configuration file. The reason for this is because Sysmon generates a lots of logs which can flood our system and so we can specify which specific processes we want to log and the ones we want to ignore

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/489c4715-dc92-4faf-bf71-944398fdd15d" />
</p>

- I then copied all of the content of `sysmonconfig.xml` on the Github (once you click the link) into a plain text file and named it `sysmonconfig.xml`, unzipped the `Sysmon.zip`, took the `Sysmon64.exe` (this `.exe` was for a Intel/AMD x64 operating system), put that into one folder named `sysmon` and converted it to a `.iso` file. I did this one macOS so the process may be a little different for another OS. This guide was pretty useful: [link](https://www.petenetlive.com/KB/Article/0001554). Lastly, I uploaded it into Proxmox

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/98327a4c-f516-4a8b-9ec6-485072f90d50" />
</p>

- Note: Remember to go to the `Hardware` tab in the Proxmox web UI dashboard and swap out the `.iso` file in the `CD/DVD Drive`

- Next, I created a folder named `Tools` and put the `Sysmon64.exe` and `sysmonconfig` in there

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/680244ac-c212-442e-a343-2472d59f135a" />
</p>

- I, then ran the installation
```
.\Sysmon64a.exe -i sysmonconfig.xml -accepteula
```

- BOOM!, it worked :D

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/573b2d38-37e2-4510-9655-8a346ea840cc" />
</p>

- I opened up `Event Viewer` > `Applications and Services Logs` > `Microsoft` > `Windows` > `Sysmon` > `Operational`. We can see some of the logs here. `Event ID 1` tracks process creation as well as who ran it, exact command-line argument used, and the file cryptographic hash, `Event ID 3` tracks whenever a process talks over the network and records the source/destination IPs and ports, `Event ID 11` tracks when files are created and lastly, `Event ID 22` tracks DNS queries

- Note that `Event ID 3` and `Event ID 11` are not shown below but these are pretty common IDs to watch out for

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/82591413-e559-4d70-b59a-dbeb71e6cdb0" />
</p>

### `Event ID 1`
- Let's test this by typing in
```
net user administrator /domain
```
- which stimulates an attacker searching for information about the admin account and I typed this by opening Powershell. If we look in `Event Viewer` in the `CommandLine` row, we can see our exact command that I ran

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/6657782b-9103-4916-b03a-d26b54720c49" />
</p>

- We can also see the parent process that spawned that child process above which was Powershell. When I typed in the command, Windows used the command-line network tool `net.exe` to query the Domain Controller (`DCO1.homelab.local`) for information about the Administrator account

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/c0a3e22d-d269-4049-a6db-386db5f4c07a" />
</p>

### `Event ID 3`
- This event ID tracks whenever a process talks over the network such as downloading something from the Internet. To test this, instead of using the Internet because I couldn't as this is an isolated lab, I created a mini Python server
```
python -m http.server 80
```
<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/6049e5c8-7608-4cb9-8656-1247a510d466" />
</p>

- And then I opened up a web browser and went to

```
http://192.168.1.200
```

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/2f7680e3-1d54-48d8-b4ce-6cf7fbc3ea86" />
</p>

- If we go into `Event Viewer` and filter the logs, we can see the Python process being launched in the `Image` row (`python.exe`) which was our python server as well as the protocol being used which is `tcp`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/bb6a58ab-8ae9-48c1-9eea-2ad822262d06" />
</p>

- You can also barely see at the top the `SourceIp: 192.168.1.200`, `DestinationIp: 192.168.1.200`, `DestinationPort: 80` and `DestinationPortName: http`. This proves that a Python process was running and it accepted a network connection over tcp on port 80 by my web browser and Sysmon captured that specific activity. Note that the web server connected to itself as I had the python web server running on `127.0.0.1` as you can notice by the same source and destination IP address

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/a98986c5-cb49-4027-8cc1-b0e1acdbc3a0" />
</p>

### `Event ID 11`
- This event ID records File Create operations. It triggers whenever a new file is created or an existing file is overwritten. Let's create a plain text PowerShell script file with a single comment in it
```
New-Item -Path "C:\Tools\suspicious_script.ps1" -ItemType "file" -Value "# Fake Malicious Script"
```

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/7948f99a-b94d-4e9b-a430-77c36ba7bda0" />
</p>

- We can see it was a Powershell process that spawned and our file that we created :D

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/43ac835a-1590-46a8-b33b-cb649fca1728" />
</p>

### `Event ID 22`
- This event ID captures DNS queries made by processes on a Windows system. This is useful because malware frequently relies on domain names to locate command-and-control (C2) servers or download malicious payloads. I made a fake DNS query but obviously it is not going to work because we are in a isolated setting

```
[System.Net.Dns]::GetHostAddresses("evildomain-c2-beacon.com")
```

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/d39763b3-8f56-4b02-81d5-2b7649229162" />
</p>

- However, Sysmon still capture that DNS request and we can see the exact query name we wanted to resolve appear below

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/49a67269-8aaa-4a68-b475-cf880ebb6339" />
</p>
