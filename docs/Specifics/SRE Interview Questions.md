# SRE Interview Questions


## Q1: Depth of knowledge

Goals: With this question, we want to check the candidate’s ability to demonstrate some depth of knowledge with regard to operating systems and networking. Particularly, a successful candidate will be able to methodically explore the answer space and give a coherent description of what is going on at each step. Rather than concept memorization, we’re looking for the candidate to connect the ideas and reason about the answer – so it may be fine for reasonable hypotheses to be accepted as long as the candidate calls them out as such.

Topics we want covered by this question:
-   Linux system administration background knowledge, particularly a good understanding of:
	-   CLI Tools
	-   Fork/Exec, argv
	-   Networking
	-   Routing
	-   DNS
-   Specific networking topics:
	-   TCP handshake
	-   ARP
-   Security topics:
	-   Basic understanding of PKI and why it’s important
    
### Questions (One of the following – 10-15 minutes):
-   Explain in as much detail as possible what happens on the client, network, and server when you are at a bash shell prompt on a Linux host and type “ssh admin@shell.linkedin.com”
-   Explain in as much detail as possible what happens on the client, network, and server when you are at a bash shell prompt on a Linux host and type “curl https://www.linkedin.com/api/”
-   Explain in as much detail as possible what happens on the client, network, and server when you are at a bash shell prompt on a Linux host and type “scp -r my_directory admin@shell.linkedin.com:~”

### Answers
Interviewers should probe for the candidate to describe the flow systematically and feel free to ask specifically about the top level bullet points.
-   Client host
	-   Parsing shell arguments
	-   Finding the binary in the path/checking aliases
	-   Executing the binary using fork/exec
	-   Some sort of reasoning about the meaning of the arguments (common things vs trivia)
		-   Ssh: They should be able to note that admin is the user and shell.linkedin.com is the host
		-   Curl: They should note https. -L means to follow redirects
		-   SCP: In this case, a candidate should know that my_directory refers to a local directory and the “:~” refers to the remote destination
	-   Checking local configuration
		-   SSH/SCP: Candidates should have some understanding of ssh_config files and known_hosts (or at least know that these exist)
		-   CURL: That curl checks for a curlrc is likely less common knowledge but it’s nice if they mention it
    
-   Network communication
	-   DNS
		-   At least candidates should know what DNS is and its purpose
	-   How DNS works
		-   Client Cache + Servers defined in resolv.conf
		-   Recursive DNS resolution
		-   Authoritative DNS Server
-   OSI Model – (if they have some familiarity with networking)
	- [[Networking#OSI Model]]
-   Network
	-   This probably depends a lot on candidate experience with networking. There’s a lot of pieces they could talk about – give them credit for this but don’t let it take the whole time for the question!
	-   They should know about ip addresses and ports
	-   Ideally they would also know that routers and switches exist and what each is for
	-   Great depth for this time period would be to give a simple description of AS + BGP Routing on the internet and some description for why this is needed
-   Transport
	-   In this case, all 3 commands use TCP
	-   Candidates should be able to describe the TCP Handshake (SYN SYN/ACK ACK)
	- ![[Pasted image 20220418175721.png]]
	-   They should also understand the guarantees of TCP (retransmission + order)
		- Checksum
		- serial number
		- Confirm response
		- Retransmit after timeout
		- Connection management
		- flow control
		- Congestion control
	-   Great depth would be the ability to give some description for how these are accomplished
-   Application
	-   [[SSH]]
-   Basic knowledge about the protocol in each command is nice (for example how HTTP works)
    
## How to ask this question

-   [First 5 minutes] In the next 5 or so minutes, describe some high level problems that could happen to make <command> ssh/curl not work like you would expect.
	-   Guide the candidate to give very general answers here and make notes about what they say. Also don’t allow candidates to give too similar answers twice (say two answers around security – we want to get a broad scope of answers and dive on each of them).
	-   For example, say they identify “dns could be broken” and “routing could be broken – no route to host”
	-   These are two different networking topics
	-   An interviewer should combine this into one 5 minute description below (?)
-   In 5 minute chunks of time, ask about each of the candidates questions – if they were not able to give 3, propose an example problem.

### Rubric

This rubric provides a point-based system for assessing each “pillar of knowledge” regarding <command>. This allows us to assess both breadth and depth. The below tables provide possible candidate explanations that pertain to a certain level of knowledge, 0 being least knowledgeable and 3 being most.

#### Client Side Problems

1 anything pertaining to path or file permissions, possible aliases
2 issues with binary loading and execution
3 kernel issues, issues with virtual file system of physical disk  

#### Network Problems

1 “Is it connected to the internet?"
2 ICMP vs TCP, DNS resolution
3 flapping NIC

#### Security Problems

1 Firewall, port blocked
2 untrusted certificate, known host mismatch
3 cipher negotiation failure, potential spoof, failure to generate session key

#### Remote Problems

1 Remote doesn’t exist or isn’t able to receive ssh
2 Port blocked locally
3 Kernel issues, issues with virtual file system or physical disk


## Freshness Justification

These questions are open-ended enough so as to allow the candidate to explore the topics deeply and demonstrate understanding. Just knowing the question does not help the candidate to give a coherent answer, and we feel discussion will quickly reveal candidates with prepared answers.

We feel that even knowing the question or having a prepared answer, this question still gives a strong signal for the question goals here.

Specifically these questions (curl + ssh) are well used enough that the majority of candidates should be familiar with them. In addition, we give candidates the opportunity to talk about many concepts at both the operating system and network layers, which another question might not fully capture.

## Q2: Scalability and Monitoring

Goals: With this question, we want to give the candidate the opportunity to apply their knowledge of systems to solve a scalability problem in production.  Rather than a knowledge based answer (listing buzzwords), a successful candidate will demonstrate critical thinking and give reasonable ideas for debugging, monitoring, and scaling.  We expect this interview to be more discussion oriented than the first question, and candidates should ask questions to understand the scenario well before trying to come up with answers.

### Topics Covered
-   Monitoring
	-   Metrics/Logging/Synthetic Testing/RUM (real user monitoring)
-   Software profiling
-   Service Tracing
-   Scaling
	-   Horizontal Scaling
	-   Vertical Scaling
	-   Autoscaling
	-   Sharding or partitioning data
    
### Question (15-20 minutes?):

You start a new job and receive user complaints that the page load time for shipping/payment screen has been getting slower and slower the past few months.  The management team projects this slowness is causing revenue to drop, asking you to make fixing the issue your top priority.

Additionally, they want you to come up with a solution to detect issues like this in the future.

The system architecture for the service in question currently contains a frontend presentation webserver, a number of backend api services, and a mysql database.

What steps and in what order would you take to try to resolve this issue this week?

What methods would you recommend to detect and prevent such issues in the longer term?  

[First 5 minutes] For the next 5 minutes, let’s talk about some high level monitoring and metrics you want to examine.
-   Candidates should give a few high level categories (client side issues, server side issues, remote issues)
[Next 10 minutes] Talk about each (or a selection) of the categories they identified
[Final 5 minutes] Talk about how to evaluate/improve the scalability of the application.

### Rubric
1 least points, 3 most points

#### External Monitoring
1 Identifies need incase local monitoring fails on itself
2 Regional differences, ISPs down
2 Breachable SLA (99th, 95th, 50th percentiles)
3 QPS, Latency, Errors (QLE)


#### Host Metrics
1 Basic self healtcheck
2 Diskspace, memory, i/o
3 Programming language specific, such as Java GC, etc
3 **SLAs, SLOs, SLIs**
-   An SLA is a contract.
-   An SLO is a specific goal that is defined in a contract.
-   An SLI measures the extent to which teams comply with the SLO promises they make in SLA contracts.
	- The SLI (Service Level Indicator) equation is **the number of good events divided by the total number of valid events, multiplied by 100 to keep it a uniform percentage**.
	- measures how well a company actually meets the SLO promises that it sets within SLAs.
3 Alerting/reporting on SLA breaches

#### Log Aggregation
1 Why this is important in a high-availability / distributed system
2 Queryable

#### Alerting Strategies
1 Identifies need for basic alerting
2 Thresholds, statistics, etc, exceptions
3 Burn Rate, Error budget, ML/AI
* A burn-rate alerting policy **notifies you when your error budget is consumed faster than a threshold you define, measured over the alert's compliance period**.

Other Strategies
* Look for code changes that correlate to the problem beginning
* Try to reproduce with synthetic traffic (time consuming)
* Try a git-bisect (time consuming)
* Performance Profiling
* Heap dumps
* 


## Extra

Checking aliases

Checking $PATH

Hostname to IP resolution

Lookup in nsswitch.conf

For DNS, looking in client cache

DNS query out to first-level DNS server in resolv.conf

connect()/otherwise create a new network connection

Network stack determining local gateway

Using ARP to find gateway's MAC address

Shell parsing the command line (figuring out it isn't running a builtin, describing argv, etc.)

Shell finding the program physically on disk via VFS

Linking in dynamic libraries when launching binary

Detailed description of Internet routing (outside gw to our site)

Detailed discussion of environment variables/argument parsing for the binary

Correctly describes the 3-way TCP handshake

Verification of remote host via SSL certificate checking or SSH host key checking

Knowing sessions are encrypted with a symmetric session cipher, and why

Correctly describing DNS beyond the first resolver (root NS, TLD NS, authoritative)

Explaining the nuances of fork()/exec() - their differences, how they interact with the parent program's environment

Detailed discussion of encryption (cipher negotiation, MAC negotiation, more detail on cert verification, etc.)


