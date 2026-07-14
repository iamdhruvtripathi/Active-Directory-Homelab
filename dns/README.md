# DNS
- This part details more about understanding how DNS supports Active Directory and to configure a reverse lookup zone

## What is DNS?
- DNS enables devices to locate IP addresses by translating human-readable hostnames into IP addresses. This translation is typically performed through A or AAAA record lookups. An A record maps a hostname to an IPv4 address, while an AAAA record maps a hostname to an IPv6 address. DNS also supports many other record types, such as CNAME (alias), MX (mail exchange), TXT (text information), NS (name server), and SRV (service location), each serving a specific purpose in DNS resolution and network configuration

## Forward Lookup Zone
- Let's explore the forward lookup zone specifically when we click on DNS in the image below. A forward lookup zone basically translates names into IP addresses

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/3571e625-d901-41d9-ac1d-a74df4bc3cc0" />
</p>

- We see many folders but the one I focused on was the `Forward lookup zone` folder

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/2c1e628e-2895-4f02-bcc1-0fd529e173ed" />
</p>

- We can see below the two `A` records which hold the IP address for `dco1` and the Windows client VM named `DESKTOP-HREURTQ`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/8c833e8b-c01f-4b9f-9236-c143426d7584" />
</p>

- DNS works when, for example, a computer queries `dco1`, then it would search its records for a matching A record and if it finds one, it returns the corresponding IP address to the requesting computer, which can then use that IP address to communicate with `dco1`

## Creating the Reverse Lookup Zone
- Active Directory automatically creates Forward zones, but it leaves Reverse Lookup Zones (which translate an IP address back into a computer name) blank by default. Security tools and SIEM scanners rely on Reverse zones to figure out what computer an IP belongs to. We will create a new zone as shown below

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/c122f716-8c65-4327-b3b1-58576843fe8c" />
</p>

- After clicking through a bunch of `Next`'s, I came to this choice and clicked `IPv4 Reverse Lookup Zone`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/323f9811-17f3-4a6a-9e13-0dcca238e592" />
</p>

- The network ID is `10.10.10`

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/2c8e1b55-099e-46a0-b4da-53ea63f77b79" />
</p>

- I clicked `Finish` and we are done with creating the `Reverse Lookup Zone`. Now, it is time to test it via the client VM

<p center="align">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/7b714fb6-b780-4a77-91b2-202db3d8dc23" />
</p>

- By using the command `nslookup dco1.homelab.local` we can see if the DNS resolution working properly, but... oh... it's not working properly? What's happening here?
- The error that is shown below is because we haven't made a pointer record (`PTR`) when I made the `Reverse Lookup Zone`. DNS is successfully able to resolve the hostname to the IP address and that is why we see the `Address` at the bottom successfully under `Name` but it can not resolve the address to the hostname, hence the `Unknown` (it can not resolve its own name because the `PTR` record does not exist)

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/64bf7f17-0fdb-4ac1-9520-66e872767687" />
</p>

- To troubleshoot, let's go back and create a new `PTR`. I typed the `.10` and the hostname

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/0e90ce24-d83e-424b-b9ea-0e4c221cd0cb" />
</p>

- Before I ran this command I did `ipconfig /flushdns` to clear the computers local DNS resolver cache and I ran the command below again, it worked successfully. Note that the output is similar but the top half shows which DNS server the computer was asking the information for (which is on the DC itself) and the bottom half is the hostname and the IP address

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/91deaf65-5fa0-47d8-8b08-e8d29215788a" />
</p>

- Now let's try the reverse by typing in the IP address and we can see below it worked successfully

<p align="center">
<img width="90%" height="90%" alt="image" src="https://github.com/user-attachments/assets/3e950efd-dee6-4e1b-b278-941000965dd2" />
</p>
