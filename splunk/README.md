# Splunk
- So now it is time to pull out the big guns and add Splunk to this project. The idea is that since we already have the two Windows VM setup, we can use Splunk's Universal Forwarder, a light weight agent and install it directly on both the domain controller and the Windows 11 client and send Windows Event log and Sysmon logs to Splunk
## What is Splunk?
- Splunk Enterprise Security (Splunk ES) is a SIEM platform that collects and analyzes security logs from systems, applications, and network devices. It centralizes security data to help detect threats, generate alerts, and support incident investigations. It also provides dashboards and powerful search capabilities, enabling security teams to monitor and respond to potential security incidents efficiently

### Configuring Splunk receiving port
- First, we have to make sure that Splunk is listening for incoming log data on a specific port. I configured port `9997` for this. This is because port `9997` is the standard enterprise default port where all Splunk forwarders send their raw telemetry

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
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/54f0bbdc-910f-4d09-93b9-b28c70fbfce2" />
</p>

- BOOM!, we have successfully installed the Splunk Universal Forwarder. I repeated the exact same steps for the Windows 11 Client VM

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/d1b789e9-7d6a-4f0d-b1dd-ff827e156ba5" />
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
- Now, one issue is that because we moved the Windows VMs back onto the isolated `vmbr1` bridge, they are completely cut off from the main network bridge where my Splunk server lives. Since I can access the Splunk Web UI on my Mac, my Splunk VM is still sitting on `vmbr0`, while the Windows VMs are in the `vmbr1` sandbox. Essentially, they are on two completely different virtual switches
- To solve this issue, I created another network interface where one is connected to my home network (`vmbr0`) and the other one is connected to my isolated lab environment (`vmbr1`). This allows each network interface to have its own unique subnet, preventing routing ambiguity and allowing my Splunk server to communicate with both my isolated lab environment and my home network properly
