# Load Balancers  

## Reverse Proxy Servers
Why would you use a reverse proxy server in front of a frontend server that already contains a 'webserver' (eg. tomcat)?
- security (rejecting bad requests, throttling etc.)  
- network performance (often better at handling connections – not always) static media offload  
- offload common tasks eg. manage login cookies, modify headers caching
- caching (e.g. nginx reverse proxy)

## L4 vs L7
TODO
- L1 - Channel multiplexing (e.g. WDM, GPRS)  
- L2 - Channel bonding (e.g. LACP)  
- L3 - IP level (e.g. BGP, OSPF)
- L4 - TCP level (e.g: NAT)  
- L7 - HTTP level (e.g. webserver remaps/rewrites)  
- Application level (e.g. providing different URIs to different clients) Client side LB (e.g. multiple A(AAA) records)

- What advantages does using HTTP LB provide us, when compared with TCP LB for the same connection (ie. an HTTP connection)?  
	- HTTP
		- Can do addtional decoration or header manipulation
		- Load balance based on endpoint
			- send different endpoints to different clusters/servers
		- block bad requests
		- cache data
		- can terminate SSL certs
		- set affinity for users to servers - keep experience for one user consistent
		- logging capabilities
	- TCP
		- more performant due to doing less
		- can due TCP packet passthrough or can terminate the packet and establish new connection to the backend
		- stateless
		- don't need to be updated often
	- Combo
		- to grow load balancers you can put TCP LBs in front of HTTP LBs
		- allow graceful upgrades/restarts of TCP LBs

## Different techniques
eg. least connections first, round robin, sticky, lowest latency first
