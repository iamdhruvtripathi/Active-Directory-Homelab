# Monitoring & Logging
- Well... we are here now. Look at us. We developed a fully mini enterprise environment with a Client VM, DC, DHCP server, DNS server, File Server, AD, AD DS, AD CS, and a Print server. Now is the time to move to focusing on Windows Event Logs and auditing

## What is Events Viewer?
- Windows Event Viewer is a built in tool that records important events on a computer, such as errors, warnings, and information about how Windows and programs are running. It can be thought of it as a logbook that helps analysts see what happened on the PC and troubleshoot problems when something goes wrong

## Auditing Logs
- Let's get to auditing some logs but first I have to make some configurations. First, I went to the `Default Domain Policy` to make some changes here. The reason for this was so every machine on the domain automatically starts logging security events identically

  <img width="963" height="602" alt="image" src="https://github.com/user-attachments/assets/d9358e1e-a8f2-4cb1-9e93-561b0f6ec393" />

- I then went to `Audit Policies` and turned on the `Success` and `Failure` for `Account Logon: Audit Kerberos Authentication Service & Audit Credential Validation`, `Account Management: Audit User Account Management`, `Logon/Logoff: Audit Logon & Audit Logoff`, and `Object Access: Audit File Share & Audit Detailed File Share`. Now, the below screenshot only shows for `Account Logon: Audit Kerberos Authentication Service` but the others were done for the sake of brevity

  <img width="960" height="600" alt="image" src="https://github.com/user-attachments/assets/9a3829a8-6672-41ea-a97b-93fbe72babaa" />

- Then, we apply the GPOs successfully

  <img width="963" height="597" alt="image" src="https://github.com/user-attachments/assets/c74ba361-b8a0-4c67-9829-bb02c1d13943" />
