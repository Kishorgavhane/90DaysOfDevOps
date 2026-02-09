### Day 15 – Networking Concepts: DNS, IP, Subnets & Ports

## DNS (Domain Name System)

*DNS (Domain Name System) is like the phonebook of the internet. It converts human-friendly names like google.com into machine-friendly IPs like 142.250.xx.xx.*

- Computers do NOT understand names
- They only understand IP addresses
So DNS exists to translate names → IPs.
Just like: You remember a person’s name, but your phone uses a number

## What really happens when you type google.com?

# flow

1️⃣ i type google.com in the browser : (**When I type google.com, the system first checks local DNS cache. If not found, it asks a DNS server which resolves the domain to an IP address. The browser then connects to that IP using TCP and HTTP/HTTPS.**)

2️⃣ system checks:
- Browser cache
- OS cache
- `/etc/hosts` file

3️⃣ If not found → system asks DNS resolver (ISP / local DNS)

4️⃣ DNS resolver asks:
- Root server
- TLD server (`.com`)
- Authoritative DNS server (Google)

5️⃣ DNS server replies with IP address

6️⃣ Browser uses that IP to connect via TCP + HTTP/HTTPS

## DNS work finishes here DNS does NOT load the website — it only gives IP

### DNS Record Types
- **A** – Maps a domain name to an IPv4 address
- **AAAA** – Maps a domain name to an IPv6 address
- **CNAME** – Alias of one domain to another domain
- **MX** – Mail server responsible for receiving emails
- **NS** – Name servers for a domain

### LIVE PRACTICAL dig output (example)

```bash
dig google.com
```
<img width="1102" height="363" alt="dig output" src="https://github.com/user-attachments/assets/db1be64b-b438-4757-b985-565f09c92d03" />

### IP Addressing

- Every device on a network needs an IP address Without an IP, a system cannot send or receive data.
Think of IP like: Home address for your computer/server.

IPv4 address is a 32-bit number written in dotted format, such as 192.168.1.10.
It uniquely identifies a device on a network.

**Public vs Private IP**
- Public IP: Accessible over the internet (example: 8.8.8.8)
- Private IP: Used inside private networks (example: 172.31.27.38)

**Private IP Ranges**
- 10.0.0.0 – 10.255.255.255
- 172.16.0.0 – 172.31.255.255
- 192.168.0.0 – 192.168.255.255

### ip addr show

```bash
ip addr show
```

<img width="1102" height="253" alt="image" src="https://github.com/user-attachments/assets/47091af4-a149-41df-a699-b080d169f715" />

### CIDR (Classless Inter-Domain Routing) & Subnetting

**Subnetting is just dividing a big network into smaller networks.**

Think like this:
- One big apartment building → divided into floors → divided into flats

**CIDR**
- How much part is network
- How much part is host

**Example:**
```text
192.168.1.0/24
```
- `192.168.1.0` → Network
- `/24` → Network bits

IPv4 = 32 bits total
- /24 = 24 bits for network
      - 8 bits for hosts

Hosts = `2^8 = 256`

- 1 → Network address
- 1 → Broadcast address
   **Usable hosts = 254**

## CIDR → Subnet Mask

| CIDR | Subnet Mask     |
| ---- | --------------- |
| /24  | 255.255.255.0   |
| /16  | 255.255.0.0     |
| /28  | 255.255.255.240 |

## Usable Hosts

| CIDR | Total IPs | Usable Hosts |
| ---- | --------- | ------------ |
| /24  | 256       | 254          |
| /16  | 65536     | 65534        |
| /28  | 16        | 14           |

### Formula:

``` text
Usable Hosts = 2^(32 - CIDR) - 2
```

### Ports – The Doors to Services

👉 IP tells which machine
👉 Port tells which service - A port is a logical number used by the operating system to identify a service or application. Range: `0 – 65535`

Think like:
- `IP` = building address
- `Port` = flat number inside the building
**Without ports, the OS would not know which application should receive the data.**

## Port Ranges

| Range       | Meaning                            |
| ----------- | ---------------------------------- |
| 0–1023      | Well-known ports (system services) |
| 1024–49151  | Registered ports                   |
| 49152–65535 | Dynamic / temporary ports          |

## Common Ports

| Port  | Service | Why it matters      |
| ----- | ------- | ------------------- |
| 22    | SSH     | Secure server login |
| 80    | HTTP    | Web traffic         |
| 443   | HTTPS   | Secure web traffic  |
| 53    | DNS     | Name resolution     |
| 3306  | MySQL   | Database access     |
| 6379  | Redis   | Cache               |
| 27017 | MongoDB | NoSQL database      |

### Live Practical 

```bash
ss -tulpn
```
<img width="1143" height="253" alt="image" src="https://github.com/user-attachments/assets/5e92925c-241f-47ab-8e53-bb7c8ce06753" />

- Which service is running
- Which port it is listening on

Example observation:
SSH is listening on port 22, ready for incoming connections.

## Port + Protocol

- TCP → reliable (SSH, HTTP, DB)
- UDP → fast (DNS)
Example:
```test
TCP 443 → HTTPS
UDP 53 → DNS
```

## What I Learned
- DNS is required before any connection starts.
- Subnets organize and secure networks.
- Ports decide which service receives traffic.
