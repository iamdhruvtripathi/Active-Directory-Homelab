# DNS
- This part details more about understanding how DNS supports Active Directory and to configure a reverse lookup zone

## What is DNS?
- DNS enables devices to locate IP addresses by translating human-readable hostnames into IP addresses. This translation is typically performed through A or AAAA record lookups. An A record maps a hostname to an IPv4 address, while an AAAA record maps a hostname to an IPv6 address. DNS also supports many other record types, such as CNAME (alias), MX (mail exchange), TXT (text information), NS (name server), and SRV (service location), each serving a specific purpose in DNS resolution and network configuration

## Forward Lookup Zone
- Let's explore the forward lookup zone specifically when we click on DNS in the image below. A forward lookup zone basically translates names into IP addresses
<img width="963" height="599" alt="image" src="https://github.com/user-attachments/assets/3571e625-d901-41d9-ac1d-a74df4bc3cc0" />

- We see many folders but the one I focused on was the `Forward lookup zone` folder
<img width="982" height="613" alt="image" src="https://github.com/user-attachments/assets/2c1e628e-2895-4f02-bcc1-0fd529e173ed" />
