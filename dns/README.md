# DNS
- This part details more about understanding how DNS supports Active Directory and to configure a reverse lookup zone

## What is DNS?
- DNS enables devices to locate IP addresses by translating human-readable hostnames into IP addresses. This translation is typically performed through A or AAAA record lookups. An A record maps a hostname to an IPv4 address, while an AAAA record maps a hostname to an IPv6 address. DNS also supports many other record types, such as CNAME (alias), MX (mail exchange), TXT (text information), NS (name server), and SRV (service location), each serving a specific purpose in DNS resolution and network configuration

## Forward Lookup Zone
- Let's explore the forward lookup zone specifically when we click on DNS in the image below. A forward lookup zone basically translates names into IP addresses
<img width="963" height="599" alt="image" src="https://github.com/user-attachments/assets/3571e625-d901-41d9-ac1d-a74df4bc3cc0" />

- We see many folders but the one I focused on was the `Forward lookup zone` folder
<img width="982" height="613" alt="image" src="https://github.com/user-attachments/assets/2c1e628e-2895-4f02-bcc1-0fd529e173ed" />

- We can see below the two `A` records which hold the IP address for `dc01` and the Windows client VM named `DESKTOP-HREURTQ`

<img width="1007" height="630" alt="image" src="https://github.com/user-attachments/assets/a8879d99-5c49-45fc-a619-51fd7fa3f334" />

- DNS works when, for example, a computer queries `dc01`, then it would search its records for a matching A record and if it finds one, it returns the corresponding IP address to the requesting computer, which can then use that IP address to communicate with `dc01`

## Creating the Reverse Lookup Zone
- Active Directory automatically creates Forward zones, but it leaves Reverse Lookup Zones (which translate an IP address back into a computer name) blank by default. Security tools and SIEM scanners rely on Reverse zones to figure out what computer an IP belongs to. We will create a new zone as shown below

<img width="965" height="604" alt="image" src="https://github.com/user-attachments/assets/c122f716-8c65-4327-b3b1-58576843fe8c" />

- After clicking through a bunch of `Next`'s, I came to this choice and clicked `IPv4 Reverse Lookup Zone`

<img width="962" height="603" alt="image" src="https://github.com/user-attachments/assets/323f9811-17f3-4a6a-9e13-0dcca238e592" />

- The network ID is `192.168.1`

<img width="958" height="602" alt="image" src="https://github.com/user-attachments/assets/0b2aefd8-64b6-466c-9681-28bdb9241ac8" />

- I clicked `Finish` and we are done with creating the `Reverse Lookup Zone`. Now, it is time to test it via the client VM

<img width="963" height="600" alt="image" src="https://github.com/user-attachments/assets/7b714fb6-b780-4a77-91b2-202db3d8dc23" />

- By using the command `nslookup DC01.homelab.local` we can see if the DNS resolution working properly, but... oh... it's not working properly? What's happening here?
- The error that is shown below is because we haven't made a pointer record (`PTR`) when I made the `Reverse Lookup Zone`. DNS is successfully able to resolve the hostname to the IP address and that is why we see the `Address` successfully but it can not resolve the address to the hostname

<img width="960" height="603" alt="image" src="https://github.com/user-attachments/assets/752515a7-5d9b-46f4-9209-d48a82be24fa" />

- To troubleshoot, let's go back and create a new `PTR`. I typed the `.200` and the hostname

<img width="962" height="601" alt="image" src="https://github.com/user-attachments/assets/f0363830-3b73-41b6-9556-59601a64b7dd" />

- When I ran the command below again, it worked successfully. Note that the output is similar but the top half shows which DNS server the computer was asking the information for (which is the DC itself) and the bottom half is the hostname and the IP address

<img width="958" height="593" alt="image" src="https://github.com/user-attachments/assets/38775d39-4a1d-4982-9c8c-15da15e8e4c5" />

- Now let's try the reverse by typing in the IP address and we can see below it worked successfully

<img width="961" height="600" alt="image" src="https://github.com/user-attachments/assets/2e75c8b6-5f20-421b-b105-74a49a424729" />
