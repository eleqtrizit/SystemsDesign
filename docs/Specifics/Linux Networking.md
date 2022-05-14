# IP

## What is IPv4?

**IPv4** is an IP version widely used to identify devices on a network using an addressing system. It was the first version of IP deployed for production in the ARPANET in 1983. It uses a 32-bit address scheme to store 2^32 addresses which is more than 4 billion addresses. It is considered the primary Internet Protocol and carries 94% of Internet traffic.

## What is IPv6?

**IPv6** is the most recent version of the Internet Protocol. This new IP address version is being deployed to fulfill the need for more Internet addresses. It was aimed to resolve issues that are associated with IPv4. With 128-bit address space, it allows 340 undecillion unique address space. IPv6 is also called IPng (Internet Protocol next generation).

### KEY DIFFERENCE
-   IPv4 is 32-Bit IP address whereas IPv6 is a 128-Bit IP address.
-   IPv4 is a numeric addressing method whereas IPv6 is an alphanumeric addressing method.
-   IPv4 binary bits are separated by a dot(.) whereas IPv6 binary bits are separated by a colon(:).
-   IPv4 offers 12 header fields whereas IPv6 offers 8 header fields.
-   IPv4 supports broadcast whereas IPv6 doesn’t support broadcast.
-   IPv4 has checksum fields while IPv6 doesn’t have checksum fields
-   When we compare IPv4 and IPv6, IPv4 supports VLSM (Variable Length Subnet Mask) whereas IPv6 doesn’t support VLSM.
-   IPv4 uses ARP (Address Resolution Protocol) to map to MAC address whereas IPv6 uses NDP (Neighbour Discovery Protocol) to map to MAC address.
- Because an hexadecimal number uses 4 bits this means that an IPv6 address consists of **32 hexadecimal numbers.**

![[Pasted image 20220416090231.png]]
**Note:** Because of the length of IPv6 addresses various **shortening techniques** are employed.
The main technique being to omit **repetitive 0’s** as shown in the example above.

![[Pasted image 20220416090649.png]]

The upper 64 bits are used for **routing.**
- upper 64 bits in more detail we can see that it is split into 2 blocks of **48** and **16 bits** respectively the lower 16 bits are used for **subnets** on an internal networks, and are controlled by a network administrator.

The lower 64 bits identify the address of the interface or node, and is derived from the actual physical or **MAC address**

### Address Types and Scope
IPv6 addresses have three types:
-   **Global Unicast Address** –Scope Internet- routed on Internet
-   **Unique Local** — Scope Internal Network or VPN internally routable, but **Not routed** on Internet
-   **Link Local** – Scope network link- **Not Routed** internally or externally.

![[Pasted image 20220416091004.png]]

### IPv6 Loop Back

The IPv6 loopback address is ::1.

## IPV4 w/o DHCP
The 169.254.0.0/16 network is used for **Automatic Private IP Addressing**, or [APIPA](https://www.bing.com/search?q=Link-local+address&filters=sid%3a89db65fe-f1b0-5140-92c9-9d752ab18cd5&form=ENTLNK). If a DHCP client attempts to get an address, but fails to find a DHCP server after the timeout and retries period it will randomly assume an address from this network.


# Networking 

## Why /etc/resolv.conf and /etc/hosts files are used?
**/etc/resolv.conf:** It is used to configure DNS name servers as it contains the details of the nameserver i.e., details of your DNS server. The DNS server is then used to resolve the hostname of the IP address.   
  
**/etc/hosts:** It is used to map or translate any hostname or domain name to its relevant IP address.


## Advantages of using NIC teaming?
NIC (Network Interface Card) teaming has several advantages as given below: 
-   Load Balancing  
-   Failover  
-   Increases uptime

## Network Bonding/Teaming
Network Bonding, also known as NIC Teaming, is a type of bonding that is used to connect multiple network interfaces into a single interface. It usually improves performance and redundancy simply by increasing network throughput and bandwidth.

## Network bonding modes
***Doubt anyone will ask this question - just know there are different modes***
Different network bonding modes used in Linux are listed below: 
-   **Mode-0 (balance-rr):** It is the default mode and is based on round-robin policy. It offers features like fault tolerance and load balancing. 
-   **Mode-1 (active-backup):** It is based on an active-backup policy. In this, only one node responds or works at the time of failure of other nodes. 
-   **Mode-2 (balance-xor):** It sets an XOR (exclusive-or) mode for providing load balancing and fault tolerance.  
-   **Mode-3 (broadcast):** It is based on broadcast policy. It sets a broadcast mode for providing fault tolerance and can be used only for specific purposes.  
-   **Mode-4 (802.3ad):** It is based on IEEE 802.3ad standard also known as Dynamic Link Aggregation mode. It sets an IEEE 802.3ad dynamic link aggregation mode and creates aggregation groups that share the same speed and duplex settings. 
-   **Mode-5 (balance-tlb):** It is also known as Adaptive TLB (Transmit Load Balancing). It sets TLB mode for fault tolerance and load balancing. In this mode, traffic will be loaded based on each slave of the network.  
-   **Mode-6 (balance-alb):** It is also known as Adaptive Load Balancing. It sets ALB mode for fault tolerance and load balancing. It doesn’t need any special switch support.

## Default ports used for DNS, SMTP, FTP, SSH, DHCP and squid
![[Pasted image 20220416083407.png]]


## Location of Network Interface Config File
Old way:
`/etc/network/interfaces`

Redhat/CentOS
`/etc/sysconfig/network`

New way:
`/etc/sysconfig/network-scripts`
Use:  `netplan`

![[Pasted image 20220416134700.png]]

## Hostname lookup order
List order of host name search. Typically look at local files, then NIS server, then DNS server.
`/etc/nsswitch.conf`












