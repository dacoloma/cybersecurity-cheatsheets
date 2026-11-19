# Networking Basics
## Models
### TCP/IP Model
- Application: represents data to the user, with encoding and dialog control
- Transport: Supports communication between devices across multiple networks
- Internet: Determines the best path through the network
- Network Access: The hardware devices and media - MAC addresses, switches
### OSI Model
PLRTSPA

| Layer        | Description                                                                                                     | Example                                                                                                                                                                     |
| ------------ | --------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Application  | contains protocols used for process-to-process communications                                                   | HTTP(S), DNS, SMTP, IMAP, Websocket                                                                                                                                         |
| Presentation | common representation of the data                                                                               | Compressing image, encoding, encryption                                                                                                                                     |
| Session      | organize its dialogue and manage data exchange                                                                  | client and server open a conversation                                                                                                                                       |
| Transport    | defines services to segment, transfer and reassemble the data for individual communications between end devices | TCP is like cutting a big information in small pieces, assigning a number to each pieces, and resend piece that got lost. UDP for streaming, it is ok to have a little loss |
| Network      | provides services to exchange data over the network between identified devices                                  | IP addresses                                                                                                                                                                |
| Data Link    | describes methods for exchanging data frames between devices over common media                                  | MAC addresses                                                                                                                                                               |
| Physical     | Describes the way of bit transmission                                                                           | mechanical, electrical, optical, etc.                                                                                                                                       |

### Comparison

| OSI Model                              | TCP/IP Model   |
| -------------------------------------- | -------------- |
| Application<br>Presentation<br>Session | Application    |
| Transport                              | Transport      |
| Network                                | Internet       |
| Data Link<br>Physical                  | Network Access |

## Network Access Layer
Media: switches, Ethernet cable  
Switches use MAC addresses to forward frames.  
A switch builds the MAC address table with source MAC address of each frame.
## Internet Protocol
Media: routers, NIC
Network minimal config for a host: IP address, subnet mask, default gateway (should be router to access other networks or internet), DNS server address (to translate hostname to IP address)
### IPv4
Addressing: 32 bits (8-bit segment), base 10 - ex: 192.168.5.11/24, subnet mask /24 equivalent to 255.255.255.0
Broadcast: last IP address of (sub)network - all ones in host portion
Multicast: 224.0.0.0 to 239.255.255.255
Loopback: 127.0.0.0/8
Link-local: 169.254.0.0/16 (used by Windows DHCP client if no DHCP servers available)

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
protocol used to locate other devices in a network and associates IP and MAC addresses
Protocol:
1. The sending host sends a frame addressed to a broadcast MAC address and intended destination host IPv4 address
2. Each host receives the broadcast frame. Only the corresponding IPv4 address responds with its MAC address to the sender
3. The sending host receives the message and store the MAC address and IPv4 address information in the ARP table

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