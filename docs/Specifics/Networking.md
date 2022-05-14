# Networking

## OSI Model
![[Pasted image 20220417113946.png]]



## Define TCP slow start

**Tcp slow start** is a congestion control algorithm that starts by increasing the TCP congestion window each time an ACK is received, until an ACK is not received.    
* Exponential
* cwnd *= 2 MSS

The reason you want to set it is to help find the upper limit of the maximum number of MSS's (cwnd) that can be sent at one time during each RTT.
* MSS (maximum segment size) limits the size of packets, or small chunks of data, that travel across a network, such as the Internet
	
According to [RFC 5681](https://tools.ietf.org/html/rfc5681): A TCP state variable that limits the amount of data a TCP can send. At any given time, a TCP MUST NOT send data with a sequence number higher than the sum of the highest acknowledged sequence number and the minimum of cwnd and rwnd.

Also, according to the same RFC:

The congestion window (cwnd) is a sender-side limit on the amount of data the sender can transmit into the network before receiving an acknowledgment (ACK), while the receiver’s advertised window (rwnd) is a receiver-side limit on the amount of outstanding data. The minimum of cwnd and rwnd governs data transmission.

After reaching ssthresh (slow start threshold), algorithm changes to **Congestion Avoidance**.  
	* Linear
	* cwnd += 1 MSS

## Name a few TCP connections states
- **LISTEN** - represents waiting for a connection request from any remote TCP and port.
- **SYN-SENT** - represents waiting for a matching connection request after having sent a connection request.
- **SYN-RECEIVED** - represents waiting for a confirming connection request acknowledgment after having both received and sent a connection request.
- **ESTABLISHED** - represents an open connection, data received can be delivered to the user. The normal state for the data transfer phase of the connection.
- FIN-WAIT-1 - represents waiting for a connection termination request from the remote TCP, or an acknowledgment of the connection termination request previously sent.
- FIN-WAIT-2 - represents waiting for a connection termination request from the remote TCP.
- CLOSE-WAIT - represents waiting for a connection termination request from the local user.
- CLOSING - represents waiting for a connection termination request acknowledgment from the remote
- TCP.
- LAST-ACK - represents waiting for an acknowledgment of the connection termination request previously
- sent to the remote TCP (which includes an acknowledgment of its connection termination
- request).
- TIME-WAIT -represents waiting for enough time to pass to be sure the remote TCP received the
- acknowledgment of its connection termination request.
- **CLOSED** - represents no connection state at all.

![[Pasted image 20220410142943.png]]


## Define the various protocol states of DHCP
-   DHCPDISCOVER client->server : broadcast to locate server
-   DHCPOFFER server->client : offer to client with offer of configuration parameters
-   DHCPREQUEST client->server : requesting a dhcp config from server
-   DHCPACK server->client : actual configuration paramters
-   DHCPNAK server->client : indicating client’s notion of network address is incorrect
-   DHCPDECLINE client->server : address is already in use
-   DHCPRELEASE client->server : giving up of ip address
-   DHCPINFORM client->server : asking for local config parameters

## Describe TCP header format
1.  Source port
2.  Destination port
3.  Sequence number
4.  Acknowledgement number
5.  Data offset
6.  Reserved
7.  Control bits
8.  Window
9.  Checksum
10.  Urgent Pointer
11.  Options
12.  Padding
13.  Data


## Difference between TCP/UDP
-   Reliable/Unreliable
-   Ordered/Unordered
-   Heavyweight/Lightweight
-   Streaming
-   Header size

## What are the different kind of NAT available?
There is SNAT and DNAT. 

### SNAT 
Source network address translation.  

For instance if you are at home using your cable modem and want to connect to an external site such as [www.cnn.com](https://www.cnn.com/), then your router will change the source address of the TCP packet to be it’s external public IP.

### DNAT 
Destination network address translation. 

 For instance when your packet reaches the [http://www.cnn.com](http://www.cnn.com/) router, and the web server behind the router has internal IP addresses so the router changes the destination IP (public) to the LAN IP (private).


## DNS
#### How does DNS work?
![[Pasted image 20220417121520.png]]
  **Root nameserver** - The [root server](https://www.cloudflare.com/learning/dns/glossary/dns-root-server/) is the first step in translating (resolving) human readable host names into IP addresses. It can be thought of like an index in a library that points to different racks of books - typically it serves as a reference to other more specific locations.
  
  **[TLD nameserver](https://www.cloudflare.com/learning/dns/dns-server-types#tld-nameserver)** - The top level domain server ([TLD](https://www.cloudflare.com/learning/dns/top-level-domain/)) can be thought of as a specific rack of books in a library. This nameserver is the next step in the search for a specific IP address, and it hosts the last portion of a hostname (In example.com, the TLD server is “com”).
  
  **[Authoritative nameserver](https://www.cloudflare.com/learning/dns/dns-server-types#authoritative-nameserver)** - This final nameserver can be thought of as a dictionary on a rack of books, in which a specific name can be translated into its definition. The authoritative nameserver is the last stop in the nameserver query. If the authoritative name server has access to the requested record, it will return the IP address for the requested hostname back to the DNS Recursor (the librarian) that made the initial request.

### What's the difference between an authoritative DNS server and a recursive DNS resolver?

Both concepts refer to servers (groups of servers) that are integral to the DNS infrastructure, but each performs a different role and lives in different locations inside the pipeline of a DNS query. One way to think about the difference is the [recursive](https://www.cloudflare.com/learning/dns/what-is-recursive-dns/) resolver is at the beginning of the DNS query and the authoritative nameserver is at the end.

![[Pasted image 20220417122402.png]]
#### The 8 steps in a DNS lookup:
1.  A user types ‘example.com’ into a web browser and the query travels into the Internet and is received by a DNS recursive resolver.
2.  The resolver then queries a DNS root nameserver (.).
3.  The root server then responds to the resolver with the address of a Top Level Domain (TLD) DNS server (such as .com or .net), which stores the information for its domains. When searching for example.com, our request is pointed toward the .com TLD.
4.  The resolver then makes a request to the .com TLD.
5.  The TLD server then responds with the IP address of the domain’s nameserver, example.com.
6.  Lastly, the recursive resolver sends a query to the domain’s nameserver.
7.  The IP address for example.com is then returned to the resolver from the nameserver.
8.  The DNS resolver then responds to the web browser with the IP address of the domain requested initially.

#### How to do the lookup manually
Start with the root, then do @ again with the IPs given.
```
dig pollstar.com @a.root-servers.net
dig pollstar.com @192.12.94.30
dig pollstar.com @108.162.194.74
```

#### Iterative vs Recurisve Query
Recursive - DNS host communicates with other DNS servers to get the answer on behalf of the client

Iterative - Client communicates with each DNS server in the chain itself.

##### Explain the SOA record in DNS

SOA stands for Start of Authority and it contains the following entries:

```
@ IN SOA nameserver.mycomaind.com. postmaster.mydomain.com. (  
	1 ; serial number  
	3600 ; refresh [1h]  
	600 ; retry [10m]  
	86400 ; expire [1d]  
	3600 
) ; 
min TTL [1h]fire
```

* Serial number should be refreshed each time a change is made to the zone file. 
	* This is how **slave** DNS servers know to pull a change from the master.  
* Refresh is the amount of time a **slave** DNS server should wait before pulling from the master.  
- Retry is how long a **slave** should wait before retrying to get a zone file if the initial retry fails.  
- Expire is how long a secondary server will keep trying to get a zone from the master. If this time expires before a successful zone transfer, the secondary will stop answering queries.  
- TTL is how long to keep the data in a zone file.


## Security
### How does SSL work?
SSL stands for secure socket layer. It has been renamed to TLS starting from SSL v 4.0. 

TLS is a secure way of communicating through a network. A majority of secure HTTP communication on the web takes place using TLS. TLS works at session layer and presentation layer of the OSI model. 

Initially at the session layer asymmetric encryption takes place, after that at the presentation later symmetric cipher and session key are used. The basic principle behind TLS is to encrypt data going across the network using public key encryption first, followed by using a shared key. 

Also the other component of TLS is server certificate authentication which is done through a certificate authority. Clients contain a list of certificate authorities, and it uses the public key of the CA in the certificate to verify the certificate being authentic. A good reference for TLS is here [https://en.wikipedia.org/wiki/Secure_Socket_Layer](https://en.wikipedia.org/wiki/Secure_Socket_Layer).


## TCP Three-Way Handshake Process

TCP traffic begins with a three-way handshake. In this TCP handshake process, a client needs to initiate the conversation by requesting a communication session with the Server:

![Three-Way Handshake Process](https://www.guru99.com/images/1/092119_0753_TCP3WayHand1.png)

**3 way Handshake Diagram**
-   **Step 1:** In the first step, the client establishes a connection with a server. It sends a segment with SYN and informs the server about the client should start communication, and with what should be its sequence number.
-   **Step 2:** In this step **s**erver responds to the client request with SYN-ACK signal set. ACK helps you to signify the response of segment that is received and SYN signifies what sequence number it should able to start with the segments.
-   **Step 3:** In this final step, the client acknowledges the response of the Server, and they both create a stable connection will begin the actual data transfer process.


## TLS Handshake TLDR

SSL/TLS uses both asymmetric and symmetric encryption to protect the confidentiality and integrity of data-in-transit. Asymmetric encryption is used to establish a secure session between a client and a server, and symmetric encryption is used to exchange data within the secured session. 

A website must have an SSL/TLS certificate for their web server/domain name to use SSL/TLS encryption. Once installed, the certificate enables the client and server to securely negotiate the level of encryption in the following steps:

 - The client contacts the server using a secure URL (HTTPS…).
 - The server sends the client its certificate and public key.
 - The client verifies this with a Trusted Root Certification Authority to ensure the certificate is legitimate.
 - The client and server negotiate the strongest type of encryption that each can support.
 - The client encrypts a session (secret) key with the server’s public key, and sends it back to the server.
 - The server decrypts the client communication with its private key, and the session is established.
 - The session key (symmetric encryption) is now used to encrypt and decrypt data transmitted between the client and server.

## TLS handshake
  - The client computer sends a `ClientHello` message to the server with its Transport Layer Security (TLS) version, list of cipher algorithms and compression methods available.
  - The server replies with a `ServerHello` message to the client with the TLS version, selected cipher, selected compression methods and the server's public certificate signed by a CA (Certificate Authority). The certificate contains a public key that will be used by the client to encrypt the rest of the handshake until a symmetric key can be agreed upon.
  - The client verifies the server digital certificate against its list of trusted CAs. If trust can be established based on the CA, the client generates a string of pseudo-random bytes and encrypts this with the server's public key. These random bytes can be used to determine the symmetric key.
  - The server decrypts the random bytes using its private key and uses these bytes to generate its own copy of the symmetric master key.
  - The client sends a `Finished` message to the server, encrypting a hash of the transmission up to this point with the symmetric key.
  - The server generates its own hash, and then decrypts the client-sent hash to verify that it matches. If it does, it sends its own `Finished` message to the client, also encrypted with the symmetric key.
  - From now on the TLS session transmits the application (HTTP) data encrypted with the agreed symmetric key.



