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

- Now, before we start generating test telemetry, I cleared out the old backlog of events so we can see our fresh actions in real time without scrolling through thousands of old rows. I did this for `Security`, `Application` and `System`

  <img width="959" height="601" alt="image" src="https://github.com/user-attachments/assets/780142fd-5641-4774-bc8e-bd2a5a549f36" />

### Event ID `4720`

- Now, let's actually get to seeing the logs when we do something. For this one, I will be creating a new account named `Barry Allen`. Remember when `Barry` left the `IT Department`? Well, he is back! We can see `Barry` is right there highlighted in blue

  <img width="963" height="601" alt="image" src="https://github.com/user-attachments/assets/48f4638c-1913-42cf-92b9-4218aff45433" />

- Now, let's search for the Windows Event Log ID `4720` which is generated whenever a new user account is successfully created in Active Directory and it identifies who performed the action  and the details of the newly created account

  <img width="958" height="598" alt="image" src="https://github.com/user-attachments/assets/1c55831c-ceaf-4d0e-a9a4-aa73e7b4de14" />

- There it is and we can see the `Subject` (I, the admin created his account) and the details of the new account (`SAM Account Number`, `Display Name`, `User Principal Number`, etc.)

  <img width="966" height="602" alt="image" src="https://github.com/user-attachments/assets/412a3e8e-3893-48be-9d56-41a5c9a53fde" />

### Event ID `4624`

- This Event ID is all about a successful domain logon. I logged in as `Barry`

  <img width="947" height="595" alt="image" src="https://github.com/user-attachments/assets/465eca5d-db2e-4152-ab85-da138e965a93" />

- We can see the Event ID `4624` and notice that it was `Barry` who logged in. Also note that when `Barry` logs in, Windows doesn't just do one login. It starts many background tasks at the same time, and each one creates its own security log entry. That is why we see 9 login events even though `Barry` only entered his password once

  <img width="962" height="603" alt="image" src="https://github.com/user-attachments/assets/ef321748-9d65-474e-845a-36a8107d78cd" />

- Also, one thing to note is that this is a `Logon Type 3` because we reviewing logs from a Domain Controller or a central server, rather than a local workstation and these events are generated when devices or users authenticate over the network to access shared resources, or when automated services/scripts run

  <img width="100%" height="100%" alt="image" src="https://github.com/user-attachments/assets/c3717d72-b040-42ea-9404-0002fdb217b7" />

### Event ID `4768` & `4771`
- Event IDs `4768` and `4771` are associated with Kerberos authentication. Event ID `4768` records a TGT request (typically successful shown by `Status = 0x0` which means success), while Event ID `4771` records a failed Kerberos pre-authentication attempt. Now, armed with this knowledge, let's try to enter `Barry`'s password wrong twice and successful on the third time. We can see below a there was a successful TGT request

  <img width="962" height="601" alt="image" src="https://github.com/user-attachments/assets/30353638-d84b-4507-8eb9-ef0869bf38f4" />

- If we scroll down, we can see the `Result Code` which has the `0x0` that was mentioned above

  <img width="1034" height="647" alt="image" src="https://github.com/user-attachments/assets/ecc11883-fbb8-4c85-a776-29b82159dfcd" />

- Also, we can see that since I entered `Barry`'s password wrong twice (Sorry `Barry`!), it says `Kereberos pre-authentication failed`

  <img width="962" height="602" alt="image" src="https://github.com/user-attachments/assets/44b8054a-2807-4cb3-a562-7bae09a43ff3" />

## Event ID `4725`, `4722`, `4726` & `4724`
- Event IDs `4725`, `4722`, `4726`, and `4724` track changes to user account status in Windows Security logs. They record when a user account is enabled (`4722`), disabled (`4725`), deleted (`4726`), or has its password reset (`4724`). We can see below I had disabled `Barry`'s account and it popped up in the logs

  <img width="961" height="601" alt="image" src="https://github.com/user-attachments/assets/1b951715-278c-4002-805f-65a3db6b8e87" />

- I then enabled `Barry`s account (Event ID `4722`)

  <img width="964" height="601" alt="image" src="https://github.com/user-attachments/assets/0769acd1-7589-427a-9b72-8ec73b700126" />

- Then, I reset his password (Event ID `4724`)

  <img width="963" height="600" alt="image" src="https://github.com/user-attachments/assets/4962a348-adfa-46a9-8be1-e4ed8030f27e" />

- Then, I deleted his account (Event ID `4726`)

  <img width="961" height="601" alt="image" src="https://github.com/user-attachments/assets/5a14bf5f-4bca-4304-bb45-c850020ea93a" />

## Event ID `1102`
- Now that `Barry` is gone, I cleared the Windows Security audit log and this event pops up

  <img width="962" height="601" alt="image" src="https://github.com/user-attachments/assets/2ad36ec3-421a-4700-a21d-7c779ded04cd" />

## Event ID `4625`
- This last Event ID shows a failed logon and I tried to brute-force my way into `Jack`'s account

  <img width="955" height="597" alt="image" src="https://github.com/user-attachments/assets/cef75919-8917-408a-a504-d81a399a552b" />
