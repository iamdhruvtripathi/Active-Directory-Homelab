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
