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

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/82591413-e559-4d70-b59a-dbeb71e6cdb0" />
</p>

# W.I.P
