# Networking Basics
## Models
### TCP/IP Model
- Application: represents data to the user, with encoding and dialog control
- Transport: Supports communication between devices across multiple networks
- Internet: Determines the best path through the network
- Network Access: The hardware devices and media - MAC addresses, switches
### OSI Model

| Layer        | Description                                                                                                     | Protocol Data Unit                    |
| ------------ | --------------------------------------------------------------------------------------------------------------- | ------------------------------------- |
| Application  | contains protocols used for process-to-process communications                                                   | Data                                  |
| Presentation | common representation of the data                                                                               | Data                                  |
| Session      | organize its dialogue and manage data exchange                                                                  | Data                                  |
| Transport    | defines services to segment, transfer and reassemble the data for individual communications between end devices | Segment                               |
| Network      | provides services to exchange data over the network between identified devices                                  | Packet                                |
| Data Link    | describes methods for exchanging data frames between devices over common media                                  | Frame                                 |
| Physical     | Describes the way of bit transmission                                                                           | Bits |

### Comparison

| OSI Model                              | TCP/IP Model   |
| -------------------------------------- | -------------- |
| Application<br>Presentation<br>Session | Application    |
| Transport                              | Transport      |
| Network                                | Internet       |
| Data Link<br>Physical                  | Network Access |

## Network Access Layer
Addressing: 48 bits, (8-bit segment)
Media: switches, Ethernet cable  
Switches use MAC addresses to forward frames. It builds the MAC address table with source MAC address of each frame.  
Broadcast: FF-FF-FF-FF-FF-FF, floods out all Ethernet switch ports except the incoming one, not forwarded by router  
Multicast:
- destination MAC address is 01-00-5E ( or 33-33 for IPv6) when encapsulated data is an IPv4 (of IPv6) multicast packet
- there are other reserved multicast destination MAC addresses for when encapsulated data is not IP, like Spanning Tree Protocol and Link Layer Discovery Protocol
- if configured, router and switch can multicast
Unknown unicast: When a switch receives a frame but doesn't know who the destination host is, it forwards the frame to all hosts. The non-destined host discarded the rest of the frame.  
## Internet Protocol
Media: routers, NIC
Network minimal config for a host: IP address, subnet mask, default gateway (should be router to access other networks or internet), DNS server address (to translate hostname to IP address)
### IPv4
Addressing: 32 bits (8-bit segment), base 10 - ex: 192.168.5.11/24, subnet mask /24 equivalent to 255.255.255.0  
Broadcast: last IP address of (sub)network - all ones in host portion, not forwarded by router  
Multicast: 224.0.0.0 to 239.255.255.255 OR ff00::/8  
Loopback: 127.0.0.0/8  
Link-local: 169.254.0.0/16 (used by Windows DHCP client if no DHCP servers available)
#### Header fields (20 bytes)
- Version - 4-bit - value 0100
- Internet Header Length - 4-bit - validates the packet
- Differentiated Services (former Type of Service) - 8-bit - determines the priority of the packet
- Total length - 16-bit - validates the packet
- Identification - 16-bit - identification number for fragmented packet
- Flag - determines if packet is a fragmented packet
- Fragment offset - for fragmented packet
- Time to Live (TTL) - 8-bit - source defines its value that is decrementing by one each time the packet is processes by a router. If TTL reaches 0, the router discards the packet and sends a ICMPv4 Time Exceeded message to the source
- Protocol - 8-bit - Next level protocol: ICMP(1), TCP(6), UDP(17)
- Header checksum - detects corruption in the IPv4 header
- Source IPv4 address
- Destination IPv4 address

#### Dynamic Host Configuration Protocol (DHCPv4)
- automatically assigns addressing configuration
- DHCP server is configured with pool of IPv4 addresses
- Each host is a DHCP client
- Protocol:
  1. Client needs an IPv4 address --> Sends a DHCP Discover message with broadcast destination (IP and MAC)
  2. All hosts receive the broadcast frame but only DHCP server reply --> reply with DHCP Offer suggesting an IPv4 address
  3. Client asks to use the suggested address --> Client sends DHCP Request
  4. Server responds with DHCP Acknoledgment

### IPv6
Addressing: 128 bits (16-bit segment), base 16 - ex: 2001 : 0db8 : 0000 : 00a3 : abcd : 0000 : 0000: 1234
- Rule 1: Omit leading zeros
- Rule 2: Double colon (use only once in an address)
  - ex from above: 2001:db8::a3:abcd::1234
  - other ex: 2001:0db8:0000:1111:0000:0000:0000:0200 -> 2001:db8:0:1111::200
#### Header fields (40 bytes)
- Version: 4 bits - value 0110
- Traffic class: 8 bits - like DS from IPv4 header
- Flow label: 20 bits - suggests that packets with same Flow label receive the same type of handling by routers
- Payload length: 16 bits - length of data portion
- Next Header: 8 bits - equivalent to Protocol field from IPv4
- Hop limit: 8 bits - replaces TTL field, router sends an ICMPv6 if value reaches 0
- Source IP address
- Destion IP address

### Network Address Translation (NAT)
Router translates public/private address to communicate into the internet. 
The packet is transformed just before leaving the network by the router.
A client (192.168.1.15) communicates with a distant server (200.100.5.5), ISP has assigned a registered public IP (200.100.58.2) to the client network:
1. Client sends a packet with [SRC: 192.168.1.15, DEST: 200.100.5.5]
2. Router sees the packet, assigns the source IP to a public registered address into a table and modifies the packet - [SRC: 200.100.58.51, DEST: 200.100.5.5]

| Private address | Registered public address |
| --------------- | ------------------------- |
| 192.168.2.1     | 200.100.58.50             |
| 192.168.1.15    | 200.100.58.51             |

3. Server processes the packet and sends another packet destinated to client IP [SRC: 200.100.5.5, DEST: 200.100.58.51]
4. Router sees the packet, inspect the destination IP and look into its table and forward the packet [SRC: 200.100.5.5, DEST: 192.168.1.15]

### Address Resolution Protocol (ARP) for IPv4 / Neighbor Discovery (ND) for IPv6
protocol used to resolve IPv4 addresses and MAC addresses and to map addresses using a table
Protocol (ARP Operations):
1. **ARP Request**: The sending host sends a frame addressed to a broadcast MAC address and intended destination host IPv4 address
2. **ARP Reply**: Each host receives the broadcast frame. Only the corresponding IPv4 address responds with its MAC address to the sender

Notes:
- The sending host receives the message and store the MAC address and IPv4 address information in the ARP table
- Routers do not flood ARP requests out other ports like with broadcast
- ARP table is a cache. Entries are time stamped and expire. When an entry expires, it is deleted from the table
- Static map entries do not expire but must be manually removed from the table
- for IPv6, equivalent is ICMPv6 ND
- if destination is on another network, MAC address is set to default gateway in the ARP table

Protocol:
1. **Neighbor solicitation**
2. **Neighbor advertisement messages**
#### ARP Spoofing / ARP Poisoning
Attacker reply to an ARP request (ARP Spoofing) and sends back its own MAC address to the victim (ARP Poisoning).  
To mitigate, Dynamic ARP Inspection

## Transport Layer protocols
- Transmission Control Protocol: breaks a message into segments. Segments are numbered in sequence. TCP keeps track of segments that have been sent to the destination host and can resend the segments that were lost.
- User Datagram Protocol: best effort delivery system, doesn't require acknoledgment of receipt. Preferable for streaming, VoIP. Some loss but negligeable.
- Port number: assigned by ICANN organization, range from 1-65,535
  - Well-known ports: 1-1023, destination ports associated with common network applications
  - Registered ports: 1024-49151, can be used as source or destination and can be used by organizations to register specific applications such as instant messaging apps.
  - Private ports: 49152-65535, often used as source ports

## Application Layer services
### Components of Uniform Resource Identifier (URI)
- Protocol/scheme: HTTP(S), (S)FTP, mailto, NNTP, SSH, etc.
- Hostname: www.example.com
- Path and file name: /author/book.html
- Fragment - #page155

Uniform Resource Locator (URL): Protocol/scheme + hostname + path and file name
Uniform Resource Name (URN): URL without protocol/scheme
### Domain Name System (port 53)
1. Host type www.example.com into the browser
2. DNS client (host) asks DNS server (address provided by ISP to home router) the IP address
3. DNS server sends the right IP
4. Host sends HTTP request 
Commands: `dig`, `nslokup`
Most common high level domains: .com, .edu, .net
### HTTP 80 - HTTPS 443
### FTP 21
### Telnet 23, SSH 22
### Email and messaging
SMTP (port 25): Email client send messages to its local email server then server forward the messages to the destined mailbox
POP3 (port 110): Email server delivers message to client when connected and by default deletes it when message has been accessed by the client
IMAP4 (port 143): Server used as a mailbox that receives and stores messages

## Troubleshooting commands
- `ip`, `ipconfig` `ifconfig`
- `ping`
- `netstat`, `ss`
- `tracert`
- `nslookup`, `dig`
- `arp`

## Cloud and Virtualization
### Cloud Models
- Public clouds: open to public
- Private clouds: intendend for specific entities like government
- Hybrid clouds: part private, part public, separate object but connected using a single architecture
- Community clouds: created for a specific community like healthcare organizations. Must be compliant with policies and laws
### Cloud Services
- Software as a Service: on demand software like Office 365
- Platform as a Service: provides a platform (like for dev to develop an app)
- Infrastructure as a Service: Virtual computing
### Virtualization
- Type 1 Hypervisor: Bare metal approach
- Type 2 Hypervisor: Hosted approach