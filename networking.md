# Layers
## Physical(1)
physical(1) literally just trasmits elecric signals on a medium(wired copper and light or wireless radio frequencies)
with a purely physical(1) implementation if two decides send the same data there will be conflict
## Data(2)
data(2) wraps around physical 1 adding functionality such as the ability to identify a network device in a lan(local network) with frames using mac address removing physical(1) implementations conflicts
in this sample mac address 6c:24:08:86:a1:f5 identifing a devide in a data(2) lan network the 6c:24:08 is known as oui which identifies the network card manufacturer(mac address can be spoofed at a network level, but the network card will always have the given mac address)
### Frame
a data(2) frame is composed by
* 8b preamble(signals the frame is starting)
* 6b destination(mac)
* 6b source(mac)
* 2b ether type(which network(3) protocol is putting it's data in the frame)
* payload(the data itself)
* 4bframe check sequence(corruption check)
### Broadcast
when the destination address is ff:ff:ff:ff:ff the frame gets broadcasted
### Collisions
* CSMA/CD(Carrier Sense Multiple Access)/(Collision Detection) waits if the line is busy(ethernet)
* CSMA/CD(Carrier Sense Multiple Access)/(Collision Avoidance) checks before transmitting(wifi)
### Protocols
Ethernet, WiFi and L2oS
## Network(3)
network(3) adds cross network routing and identification with ip addresses encapsulating data(2) frames as part of packets allowing you to reach a network that has other networks along the way meaning that we are sending something to a specific ip network that contains an host with a specific private ip and mac address
there are two versions of ip address v4 and v6(with this being way larger)
### Packet v4(32bit)/v6(128bit)
133.33.3.7 is 10000101 00100001 00000011 00000111
* ttl(time to live) 
* protocol(what protocol is being transmitted, e.g. icmp, tcp or udp)
* other headers 
* source
* destination
* data
you may not be able to reach a certain ip address because offline
### Subnetting
they are devided in two parts, network and host to find them
convert the ip address and the subnet mask to binary
* 133.33.3.7
10000101 00100001 00000011 00000111
* 255.255.0.0 or /16(shorthand, the number of 1s in the subnet)
11111111 11111111 00000000 00000000
all of the octals are 0-256
everything with a 0 represent a host component
a /16 subnet has 65534 hosts and a /24 one just 254(because 256 minus the network id and broadcast address which are .0 and .255)
#### Possibile game company network
[Internet]
│
├── Public IP: 104.160.141.3 (Load Balancer)
│   ├── Game Server 1: 10.5.1.1
│   ├── Game Server 2: 10.5.1.2
│   └── ...
│
├── Database Subnet: 10.5.2.0/24 (Private, no direct internet access)
│   ├── Primary DB: 10.5.2.1
│   └── Replica DB: 10.5.2.2
│
└── Monitoring Subnet: 10.5.3.0/24 (Private, no direct internet access)
    ├── Prometheus: 10.5.3.1
    └── Grafana: 10.5.3.2
#### AWS specific setup
In AWS, this would map to something like this
* VPC: 10.5.0.0/16 (Biggest container)
    Public Subnet: 10.5.0.0/24 (Load Balancer)
    Private Subnets:
        10.5.1.0/24 (Game Servers in Auto Scaling Group)
        10.5.2.0/24 (RDS Databases)
        10.5.3.0/24 (CloudWatch/Monitoring)
#### From DeepSeek
##### Is the Load Balancer the Only Public Resource?
Yes, usually. In modern cloud setups:
* Load Balancer (Public): Handles player traffic.
* NAT Gateway (Public IP but only allows outbound traffic from private subnets): e.g. to download patches.
* Everything else (game servers, DBs): Private, unreachable from the internet.
Exceptions are Bastion Hosts/Jumpboxes with a single public IP for SSH access (locked down to admin IPs).
##### Is It Common to Reserve More Than /24?
Yes, but carefully.
###### Why /16 (65,536 IPs) for a VPC?
* Future-proofing: Room for more subnets (e.g., dev/staging, analytics).
* Isolation: Each environment (prod/test) gets its own /24.
* Avoids renumbering: Changing IP ranges later is painful.
###### Real World Example
Riot’s VPC: 10.5.0.0/16 (65k IPs total).
    10.5.1.0/24 → Game servers (254 IPs).
    10.5.2.0/24 → Databases.
    10.5.3.0/24 → Monitoring.
    10.5.4.0/24 → Future expansion.
### Protocols
* ICMP(Internet Control Message Protocol) used by ping and traceroute
* ARP(Address Resolution Prococo) mapping ips to mac address in lan
* Routing protocols such as RIP, OSPF and BGP determining the best paths for the packets
## Transport(4) and Session(5)
Every single network(3) packet it's is own thing, depending on network condictions it's possibile that a packet out of let's say 6 arrives in the wrong order(as in 3 arrives to destination before 2) and packets can also be lost in the way
To solve these we have transport(4) protocols, TCP and UDP
* TCP is connection oriented, slower and reliable
* UDP isn't connection oriented, faster and less reliable
### TCP segments
Building on top of network(3) capabilities and adding critical features for the modern internet to work such as ordered, error checked and flow controlled data delivery
This is what it's composed of
* Source Port(16b) and Destination Port(16b), allowing for multiple streams of communication at the same time beetween two devices
This are some ports: 80/tcp is HTTP, 443/tcp is HTTPS and 22/tcp is SSH
Communicating with AWS from your computer, playing a game and watching a show, all at the same time is possible because of this
Reason why an EC2 instance can have both an HTTP server and a SSH server running at the same time
* Sequence Number(32b), ensures the correct ordering of packets
Each byte(B) in a stream is assigned a sequence number allowing reassembling the data in the correct order
* Aknowledgments Number(32b), used for reliable delivery to confirm reciepts of data(e.g. recieved segment 1, 2 and 3, waiting for 4, 5 and 6)
* Header Lenght(4b) specifies the lenght of the header which will typically be of 20B
* Control Flags(9b with 6b commonly used), used for connection management(SYN, ACK, FIN, RST)
* Window Size(16b), implements flow control preventing the sender to overwhelm the reciever with too much data
* Checksum(16b) discards corrupted segments
* Options(variable lenght and optional)
* Data(it's the packet data)
### ARQ
Stands for Automatic Repeat reQuest and allows for just that, if a segment is lost, it's going to be resent and then the segment will be reassambled
### TCP connection
In a client and server architectures we might have a client on tcp/23060 requesting some data to the server on tcp/443, because this is a connection for the server to send data back to the client, the ports are flipped meaning that tcp/443 is now used to send data after getting the request
Ephemeral ports such as 23060 are temporary ports used by clients and 443 is a well known port always used by a protocol
On a network ACL within AWS you need two sets of rules, one for the initial request from the client and another for the response from the server
#### The 3 way handshake
You can't send data before making a connection
Here are the steps:
* SYN("hey let's talk") sets the cs initial sequence number 
* SYN-ACK("sure") picks a random ss(Sends Segments) and also sends cs+1 which is the acknowledgment
SYN sycronizes sequence numbers and SYN-ACK also acknoledges
* ACK("let's go") sends ss+1 and cs+1 also acknowledging and the connetion is established
After a TCP connection the devices will be sending IP packets to each other
## Presentation(6)
Does encryption(TLS/SSL), compression(gzip) and serialization(JSON/XML/protobuf) meaning HTTPS encrypts at presentation(6)
## Application(7)
End user protocols such as HTTP, SMTP, DNS and Websockets, anything that humans interact with
Meaning YouTube's API is application(7), APIs can be REST, SOAP(no), gRPC(binary protocol which is faster then JSON) and GraphQL
## VLANS
To reduce the broadcast traffic on big data(2) networks
In a physical only world if we had to divide three floors of positions in an office let's say developers, testers an sales, we would need to have three switches!
This would leviate the crazy amount of broadcast traffic but making the three groups unable to communicate
The 802.1Q ethernet frame expands the origina data(2) frame with a new segment
* 802.1Q(32b) that contains the VLAN ID or VID with 0 meaning "no VLANs" and 1 meaning "management VLAN ID"
Access ports do not use 802.1Q frames
Trunks ports do
There is also 802.1AD(nested QinQ VLANS)
## SSL and TLS
TLS is the better SSL
During the key exchange a server certificate which contains a public key trusted by a CA(certificate authority)
## Firewalls
iptables(with commtrack module), nftables and firewalld are stateful by default
tc(traffic control) isn't
linux's port of openbsd's pf is
Jumboframes are bigger frames up to 9000 bytes in payload, but they cannot be used everywhere inside AWS
## Hashing
A hashing function like MD5, SHA256 or BLAKE3 take some data(e.g. a message or an image) and return a unique hash identifing that data
They are one way and used in databases for passwords for example
MD5 is less trusted because collision(2 different images can end up with the same hash) can happen
Checksum are just this!
You can hash whatever you made available for download, so that when someone downloads it can also take the hash and verify the data hasn't been altered
## More
IPSEC VPN is used to create a secure connection in an insecure network
Devided in the IKE 1 and IKE 2 phases
Nftables can be a layer 7 firewall option
Asymmetric encryption is slow, but necessary to securely exchange keys to then switch to a symmetric encryption
Fiber optic cables are composed by core, cladding, buffer and the colored jacket, the optic transivers at the end of the medium will covert data to light and viceversa
Steganography allows you to hide an encrypted message(or more) inside something else like an image, the recipient of the message has to be aware of this because the image would look almost identical to the original one
HSM(Hardware Security Modules)s devices arae separated from your whole infrastracture handles everything related to "encryption keys" and they are even able to handle certificates and offload your web servers SSL/TLS work
## Related to AWS
* EC2 instancies will get a mac and ip address
* VPC networking uses subnetting for segmentation
* ENI(Elasting Network Interface) is a virtual network card
* Global Accelerator or Route 53(latency-based routing) directs players to the nearest load balancers and each region load balancer would have it's own public ip
* Network ACL is a stateless firewall and won't understand the state of the TCP segment and both directions of the connection will need a rule(e.g. "allow outbound tcp/23060 to tcp/443" and "allow inbound tcp/443 to tcp/23060", in stateful(Security Groups) firewalls there is no need to specify the reverse rule
* AWS reserves 5 ips per subnet leaving you with 251 usuable ips in a /24
## DNS(Domain Name Systems)
Resolves domains to IPs
DNS servers run on 53 by default(AWS Route 53, got it, duh)
### Important public DNS
Google(8.8.8.8, 8.4.4.8) https://dns.google privacy concerns
Cloudflare(1.1.1.1, 1.0.0.1) https://one.one.one.one which is faster and privacy focused
OpenDNS(208.67.222.222, 208.67.222.220) fucking sucks dick and owned by Cisco
You could host your own DNS server to block ads in your network using Blocky or such which will act as filter on the set public one
Obviously there are always "hosts" lists sending shitty websites to "0.0.0.0" which won't be resolved upon request
### Zones
A DNS Zone contains all the records for a domain(*.google.com) stored in a Zonefile
NS(Name Servers) simply hosts these Zonefiles
Private zones are used for internal domain resolution(e.g. inside a VPC)
See DNS as a database designed to find the authorative zones for the request domain
This is distrubuted globally
### Example
Structured as a reverse tree as in "play.google.com" and not "com.google.play"
With the TLD(Top Level Domain) being "com", the SLD(Second Level Domain) being "google" and the subdomain record used here being "play"
#### Walking the tree
1. You type "play.google.com" in Firefox and your OS checks for a cached DNS entry which directly has the IP address, if missing
2. The set recursive resolver which is the public DNS server you are using will query root server like "a.root-servers.net", these are spread all over the world and managaged by important entities(there are 13 find them here https://www.iana.org/domains/root/servers)
3. From that you get the all the TLD servers for ".com"(from Verisign)
4. Now you have the authorative NS(example "ns1.cloudflare.com" and "ns2.cloudflare.com") for the domain revealing where the records have been set, the domain might have been originally registered somewhere else
5. Finally you get the IP from the nameservers themselves which again, host the DNS Zone in a Zonefile for the domain which has all the set records mapped and will get you what to connect to, the IP address
#### Cached
Firefox may cache the entry for your OS to later look it up
Cached DNS entries expire after TTL and are not authorative
Changing public DNS does not clear the cache
##### Removing cached entries
###### Windows
```bat
ipconfig /flushdns
```
###### Linux
```sh
sudo systemd-resolve --flush-caches
```
### Types of records
A records are mapped to IPv4 addresses
AAAA records are mapped to IPv6 ones
CNAME records redirect from a domain to another effectively aliases
MX records are just like A and AAAA records but for email(e.g. an SMTP server might query MX records to find where to send an email)
TXT records are used for verfication(SPF and DKIM) and spam filtering
NS records specify the authorative nameserver for the domain
There is also TTL(Time To Live) whcih is how long a DNS query response will be cached on the resolver
#### Route 53
CNAME records cannot use root domains or conflict with MX and NS records
The special Alias records in Route 53 can point to root domains and other AWS resources
### DNSSEC
Adds cryptographic signing to prevent spoofing