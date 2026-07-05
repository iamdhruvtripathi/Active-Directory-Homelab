# Monitoring & Logging [REDO PARTS]
- Well... we are here now. Look at us. We developed a fully mini enterprise environment with a Client VM, DC, DHCP server, DNS server, File Server, AD, AD DS, AD CS, and a Print server. Now is the time to move to focusing on Windows Event Logs and auditing

## What is Events Viewer?
- Windows Event Viewer is a built in tool that records important events on a computer, such as errors, warnings, and information about how Windows and programs are running. It can be thought of it as a logbook that helps analysts see what happened on the PC and troubleshoot problems when something goes wrong

## Auditing Logs
- Let's get to auditing some logs but first I have to make some configurations. Let's say for example `Bob` tries to access a protected folder, Windows won't actually log that failure unless we explicitly tell it to audit object access. First I typed `gpedit.msc` in the run box and we arrive here
<img width="960" height="598" alt="image" src="https://github.com/user-attachments/assets/8d0bbc18-62da-4854-9a06-25a066c7d5ab" />

- I doubled clicked on `Audit File Share`
<img width="963" height="602" alt="image" src="https://github.com/user-attachments/assets/6b01c4de-6d94-4d80-8d7a-ac9312d8723d" />

- I then checked both of these boxes `Success` and `Failure`
<img width="958" height="597" alt="image" src="https://github.com/user-attachments/assets/6a2ea8b9-0299-4c7e-85c8-8cb6556a461b" />

- Now, I logged in as `Harmony` and tried to access the `CompanyShare` file but was denied access. I made `Bob` the only one who could access it this time instead of `Jack`
<img width="958" height="596" alt="image" src="https://github.com/user-attachments/assets/362f6014-504d-4b84-8120-e7587c89c69a" />

- I then typed her password wrong 3+ times and it locked my out of her account D:
<img width="964" height="601" alt="image" src="https://github.com/user-attachments/assets/453717b0-2783-4948-a81f-924f0a517467" />
