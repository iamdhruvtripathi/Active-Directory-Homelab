# Splunk
- So now it is time to pull out the big guns and add Splunk to this project. The idea is that since we already have the two Windows VM setup, we can use Splunk's Universal Forwarder, a light weight agent and install it directly on both the domain controller and the Windows 11 client and send Windows Event log and Sysmon logs to Splunk
- I also did have Sysmon installed and I could have done it via Sysmon sending to a Wazuh agent which sends to a Wazuh server and then to Splunk but since I already had Splunk installed, it was easier to do it this way and learning Splunk is pretty important, although Wazuh is also a great tool to learn and use as well
## What is Splunk?
- Splunk Enterprise Security (Splunk ES) is a SIEM platform that collects and analyzes security logs from systems, applications, and network devices. It centralizes security data to help detect threats, generate alerts, and support incident investigations. It also provides dashboards and powerful search capabilities, enabling security teams to monitor and respond to potential security incidents efficiently

# W.I.P
