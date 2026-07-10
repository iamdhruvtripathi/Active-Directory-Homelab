# Sysmon
- Since we already have looked at some of the most important Windows Events log, we shall also take a look at some captured advanced telemtry like `Event ID 1` or `Event ID 3`

## What is Sysmon?
- Sysmon (System Monitor) is an advanced Windows tool that captures advanced telemtry. While Windows has built-in logs, Sysmon provides much more detail tracking exactly which programs (processes) are running, what files they modify, which servers they communicate with across the internet, and so much more

## Installing Sysmon
- First, we have to install Sysmon. I headed over Microsoft's official Sysinternals page and downloaded the `Sysmon.zip` package

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/463fc8f6-46b4-4692-b1cc-1b9762e3cd72" />
</p>

- Another thing is we can install SwiftOnSecurity's configuration file. The reason for this is because Sysmon generates a lots of logs which can flood our system and so we can specify which specific processes we want to log and the ones we want to ignore

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/489c4715-dc92-4faf-bf71-944398fdd15d" />
</p>
