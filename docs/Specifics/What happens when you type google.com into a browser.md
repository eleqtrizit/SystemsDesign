# What happens when you type "google.com" into a browser?

## Typing google.com
List of things that happen when you type "google.com" into your browser:
- Hit "g" (then oogle.com)
	- Interrupt occurs, and the brower's autocomplete function kicks in.
	- Suggestions are made
	- Browser passes back control to the CPU
- Hit "enter"
	- Browser begins parsing the URL
		- if protocol is absent (e.g. http) or text is not valid domain
			- send as query to default search engine
	- covert not ascii characters with punycode (if needed)
	- check hsts list for domains that have already declared they only accepted https
		- if so, add https protocol else http

## DNS Lookup
- DNS lookup
	- checks for domain name in local cache
	- check for domain in hosts file
	- check with dns server specified in config or from server specified via dhcp
		- If the DNS server is on the same subnet the network library follows the `ARP process` below for the DNS server.
		- If the DNS server is on a different subnet, the network library follows the `ARP process` below for the default gateway IP.
- ARP (Address Resolution Protocol)
	- Needs IP and MAC to send broadcast
	- Checks ARP cache
	- Using the route table, checks to see if the IP address falls within the subnet
	- If not in one of the connected subnets, 
		- use the default gateway
		- else lookup MAC on subnet
	- ARP Request
		- `Sender MAC: interface:mac:address:here
		- `Sender IP: interface.ip.goes.here`
		- `Target MAC: FF:FF:FF:FF:FF:FF (Broadcast)`
		- `Target IP: target.ip.goes.here`
	- Response
		- directly connected to router
			- get direct reply
		- Hub
			- broadcast out on all ports
			- if router is present, it'll reply if needed
		- Switch
			- will check its MAC table to see if it recognizes the MAC it'll send out on one port
				- If not, it will broadcast to all ports
			- if router is present, it'll reply if needed
- DNS Server Found
	- open UDP connection to port 53
		- if response is too large TCP will be used instead
	- if local DNS does not have the answer, a recursive lookup begins up the chain of DNS servers until the SOA is reached, and an answer is returned.
	- [[Networking#The 8 steps in a DNS lookup]]

## Opening a Socket
Once we have IP address and the port number (80, 443 or unique via :some_num), requests TCP socket stream from OS.  Then it passes through these layers:
- **Layer 4: Transport Layer** where a TCP segment is crafted. The destination port is added to the header
- **Layer 3: Network Layer**, which wraps an additional IP header. The IP address of the destination server as well as that of the current machine is inserted to form a packet.
- **Layer 2:  Link Layer**. A frame header is added that includes the MAC address of the machine's NIC as well as the MAC address of the gateway (local router).
- **Layer 1: Physical Layer** Out to the internet.

## Out in the Wild

### SNAT
As the packet leaves the network, if the client is behind local address space **SNAT** will be performed at the router.

### Traveling the internet
Each router along the way extracts the destination address from the IP header and routes it to the appropriate next hop. The time to live (TTL) field in the IP header is decremented by one for each router that passes. The packet will be dropped if the TTL field reaches zero or if the current router has no space in its queue (perhaps due to network congestion).

-   Server receives SYN and if it's in an agreeable mood:
    -   Server chooses its own initial sequence number
    -   Server sets SYN to indicate it is choosing its ISN
    -   Server copies the (client ISN +1) to its ACK field and adds the ACK flag to indicate it is acknowledging receipt of the first packet
-   Client acknowledges the connection by sending a packet:
    -   Increases its own sequence number
    -   Increases the receiver acknowledgment number
    -   Sets ACK field
-   Data is transferred as follows:
    -   As one side sends N data bytes, it increases its SEQ by that number
    -   When the other side acknowledges receipt of that packet (or a string of packets), it sends an ACK packet with the ACK value equal to the last received sequence from the other
-   To close the connection:
    -   The closer sends a FIN packet
    -   The other sides ACKs the FIN packet and sends its own FIN
    -   The closer acknowledges the other side's FIN with an ACK

As the packet comes back, it may do **DNAT** at the router to get back to the local IP'ed client.

## TLS Handshake
[[Networking#TLS handshake]]


## If a packet is dropped
Sometimes, due to network congestion or flaky hardware connections, TLS packets will be dropped before they get to their final destination. The sender then has to decide how to react. The algorithm for this is called [TCP congestion control](https://en.wikipedia.org/wiki/TCP_congestion_control). This varies depending on the sender; the most common algorithms are [cubic](https://en.wikipedia.org/wiki/CUBIC_TCP) on newer operating systems and [New Reno](https://en.wikipedia.org/wiki/TCP_congestion_control#TCP_New_Reno) on almost all others.
-   Client chooses a [congestion window](https://en.wikipedia.org/wiki/TCP_congestion_control#Congestion_window) based on the [maximum segment size](https://en.wikipedia.org/wiki/Maximum_segment_size) (MSS) of the connection.
-   For each packet acknowledged, the window doubles in size until it reaches the 'slow-start threshold'. In some implementations, this threshold is adaptive.
-   After reaching the slow-start threshold, the window increases additively for each packet acknowledged. If a packet is dropped, the window reduces exponentially until another packet is acknowledged.

## HTTP protocol
If the web browser used was written by Google, instead of sending an HTTP request to retrieve the page, it will send a request to try and negotiate with the server an "upgrade" from HTTP to the SPDY protocol.

- Client connects and sends requests and headers, then a single newline to indicate it is done.
- Server response with response code, headers then a single new line.  
- Server then adds the HTML payload
	- if the client sends the last modification date and the data is not newer, it will respond with `304 not modified`

## HTTP Server Request Handle
What happens:
- Host breaks down the requests
- checks if it is configured to answer the domain
- checks if it can handle the request type
- Pulls the content via the handler (if handler is needed)
- Responds to client
