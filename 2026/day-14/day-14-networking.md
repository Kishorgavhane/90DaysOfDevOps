## Day 14 – Networking Fundamentals

- I understood that OSI is mainly for learning and troubleshooting, while TCP/IP is what actually runs on real systems and the internet. OSI vs TCP/IP: OSI is conceptual, TCP/IP is practical.
- Instead of memorizing layers, I focused on where problems usually occur. IP = Internet layer, TCP/UDP = Transport, HTTP/DNS = Application.
- When I run `curl https://google.com` , it’s an application request traveling over TCP and IP behind the scenes.
<img width="841" height="193" alt="image" src="https://github.com/user-attachments/assets/ef6242c9-1732-4f63-a20b-894ffef44394" />


## OSI vs TCP/IP 
OSI (L1–L7) explains networking in detail, mainly for learning & troubleshooting.
TCP/IP (4 layers) is what actually runs on systems and the internet.

# Mapping:
- OSI L1–L2 → TCP/IP Link
- OSI L3 → TCP/IP Internet
- OSI L4 → TCP/IP Transport
- OSI L5–L7 → TCP/IP Application

# Where things sit
- IP → Internet layer
- TCP/UDP → Transport layer
- DNS, HTTP, HTTPS → Application layer

# OSI Model – Layers, Protocols, Examples & Ports

**OSI (Open Systems Interconnection) model** 
I use it mainly as a **thinking and troubleshooting tool**,
---

## Layer 1 – Physical Layer

This layer is about **physical transmission of data**.
It moves raw bits (0s and 1s) over cables or wireless signals.

### Examples
- Ethernet cable
- Fiber cable
- Wi-Fi radio signals

### Protocols / Standards
- Ethernet (IEEE 802.3)
- Wi-Fi (IEEE 802.11)
- Bluetooth

### Ports
No port numbers here (this layer doesn’t understand ports)

### Real example
> Plugging a LAN cable into a server or connecting to Wi-Fi.

---

## Layer 2 – Data Link Layer

This layer handles **communication between devices on the same network**.
It uses **MAC addresses** and ensures error-free frame delivery.

### Protocols
- ARP
- Ethernet
- VLAN (802.1Q)

### Ports
No port numbers

### Real example
> ARP finds the MAC address of another machine in the same subnet.

---

## Layer 3 – Network Layer

This layer handles **IP addressing and routing**.
It decides how data moves from one network to another.

### Protocols
- IP (IPv4 / IPv6)
- ICMP
- IPSec

### Ports
No ports (ports start at Layer 4)

### Real example
> `ping google.com` uses ICMP at this layer.

---

## Layer 4 – Transport Layer

This layer ensures **end-to-end communication**.
It manages **ports, reliability, and data flow**.

### Protocols
- TCP (reliable, connection-based)
- UDP (fast, connectionless)

### Ports
✔️ Ports start here

### Examples
- TCP 80 → HTTP
- TCP 443 → HTTPS
- TCP 22 → SSH
- UDP 53 → DNS

### Real example
> TCP ensures your file download completes correctly.

---

## Layer 5 – Session Layer

This layer manages **sessions between applications**.
It controls session start, stop, and recovery.

### Protocols
- NetBIOS
- RPC
- PPTP

### Ports
Depends on application (no fixed ports)

### Real example
> Keeping a login session active while you interact with a server.

---

## Layer 6 – Presentation Layer

This layer handles **data format, encryption, and compression**.
It ensures data is readable by the application.

### Protocols / Formats
- SSL / TLS
- JPEG, PNG
- ASCII, UTF-8

### Ports
Uses application ports (like 443 for TLS)

### Real example
> HTTPS encrypts data using TLS before sending it.

---

## Layer 7 – Application Layer

This is the layer **users and applications interact with directly**.
It provides network services to applications.

### Common Protocols & Ports

| Protocol | Port | Purpose |
|--------|------|--------|
| HTTP | 80 | Web traffic |
| HTTPS | 443 | Secure web |
| FTP | 21 | File transfer |
| SSH | 22 | Secure login |
| SMTP | 25 | Send email |
| DNS | 53 | Name resolution |
| MySQL | 3306 | Database |
| PostgreSQL | 5432 | Database |
| Redis | 6379 | Cache |
| MongoDB | 27017 | NoSQL DB |

### Real example
> Running `curl https://example.com` talks directly to the application layer.

---

## Simple Flow Example

```text
curl https://google.com
↓
Application (HTTP)
↓
Transport (TCP 443)
↓
Network (IP)
↓
Data Link (MAC)
↓
Physical (Cable / Wi-Fi)

```


### Hands-on Checks Command Observations
- Checking my IP helped confirm that my system is connected to the network and the interface is active. `hostname -I` → verified private IP.
  
<img width="962" height="341" alt="1️⃣ Identity – Check your IP" src="https://github.com/user-attachments/assets/21272976-8637-48f9-8399-5e2f363b9a4b" />

-`Ping google.com` → ping gives the fastest signal — if this fails, something basic is broken.

<img width="962" height="231" alt="Reachability – Ping" src="https://github.com/user-attachments/assets/973c62da-25ad-412a-bb0a-9c9e5d7a058c" />

- `Traceroute` → Traceroute showed multiple hops, and a few timeouts reminded me how common firewalls are in cloud networks.
  
<img width="962" height="246" alt="Path – Traceroute" src="https://github.com/user-attachments/assets/caae3624-544d-4e0c-87ed-9965c911dc15" />

- `SSH on port 22` → Seeing SSH listening on port 22 confirmed that remote access is ready on this machine.
  
<img width="1833" height="351" alt="Ports – What’s listening on your machine" src="https://github.com/user-attachments/assets/a7261526-eda6-456f-9c27-c406104ed082" />

- DNS resolution: DNS resolution worked fine, which means the system knows how to translate domain names into IPs.
  
<img width="732" height="718" alt="4 Name Resolution – DNS" src="https://github.com/user-attachments/assets/e86ba8b4-2654-4015-aeff-69d790062278" />

- HTTP check: A successful curl response confirmed that the application layer is reachable and healthy.
  
<img width="1833" height="351" alt="HTTP Check – Application layer" src="https://github.com/user-attachments/assets/16c173db-4373-4ba6-bbc0-0905c42ad026" />


### Port Probe
- Tested port 22 using nc → reachable locally.
- Confirms SSH service is running.
<img width="707" height="285" alt="Mini Task – Port Probe   Interpret" src="https://github.com/user-attachments/assets/4b8c2610-b686-4553-9c22-01f2549764b3" />


### Reflection
- `ping` → Ping, because it quickly tells me whether the network path exists.

<img width="707" height="285" alt="Mini Task – Port Probe   Interpret" src="https://github.com/user-attachments/assets/a395bf7e-2412-401c-ba57-89145acb7520" />

- `DNS failure` → Application layer → I would first look at the application layer and DNS configuration.
- `HTTP 500` → Application layer → This usually means the server is reachable, but something is wrong inside the application.

### This practice made networking feel less scary and more logical when broken step by step. ###
