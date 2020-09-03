# AWS NETWORKING SPECIALTY

## FAQ NOTES

### General

* VPC can span multiple AZs
* subnet resides in one AZ
* $0.01 per GB data transfer cross AZ
* can use AMI in same region as VPC for EC2s same with EBS snapshots
* NAT does not use Sec Groups
* NAT uses EIP
* NAT scales from 5 gbps to 45 gbps

### NAT gateway

* NAT does not use Sec Groups
* NAT uses EIP
* NAT scales from 5 gbps to 45 gbps
* one elastic IP per NAT
* Edit route table for NAT
* Supports 55000 connections per minute

### Route Tables 

* Each subnet must be associated with a route table
* Gateway route tables are associated with an IGW or VPGW

### DHCP

* one DHCP option set per VPC

### Connectivity 

 * Internet (Internet Gateway)
 * Corporate data center (Virtual Private Gateway w/ AWS site-to-site VPN connection)
 * AWS services (IGW, NAT, VPC endpoints)
 * Other VPC (VPC peering)

### Internet Access

* Internet Gateway
* NAT 
* Route to internet through VPN or Direct Connect to corporate Data Center
  
### Traffic over Internet vs AWS backbone

* same region stays back bone even using public IP
* Inter Region Peering uses backbone
* Cross region without peer no guarantee backbone

### IP Addressing IPv4

* Can use RFC 1918 or publicly rotatable IP ranges
  * RFC 1918: 
    * 10.0.0.0 - 10.255.255.255 (/8 mask)
    * 172.16.0.0 - 172.31.255.255 (/12 mask)
    * 192.168.0.0 - 192.168.255.255 (/16 mask)
* minimum subnet is /28 mask (14 IPs)
* max subnet is /16 mask 
* max 200 subnets per VPC
* can add four secondary CIDR ranges to VPC 
* first four and last ip in every subnet is reserved by aws 
* Primary private IP are retained for instance lifetime. Secondary can be unassigned.
* EIPs only usable for instances that route to IGW not applicable for instances using NAT
* Can assign one or more secondary private IPs to an ENI
* Size of CIDR blocks are fixed cna only delete and add secondary CIDRs

### IP Addressing IPv6
* Auto assigned /56 VPC mask
* subnets are all /64
* not ENIs for IPv6

### VPC Traffic Mirroring
* Can mirror traffic to and from and ENI
* mirror to an NLB with UDP listener or another EC2
* in same VPC or diff one with peering or transit gateway
* Flow logs provide info on allowed or denied traffic but not packet level insight like traffic content, payload and such.

### ENI

* must be in same AZ as instance
* charges for Elastic IPs associated to a network interface even if not attached to running instance 

### Peering Connections

* available across all regions except China
* Peers VPC must have non over lapping IP ranges
* not cost except for data transfer
* no edge to edge routing for peered VPCs
* peering traffic within region not encrypted  
* No transitive peering
* Placement groups can span peered VPCs
* Peering traffic cross region encrypted using AEAD
* No cross region sec groups for peered VPC 
* IPv6 is supported 
  
### ClassicLink

* allows ec2-classic instances to communicate with instances in a VPC using private IP addresses
* must use private addresses
  
### Private Link

* Enabled customers to access services host on AWS in an HA and Scalable manner while keeping traiffc within AWS network
* Creates an ENI for the service private IP
* Can front with NLB
* Concept of data transfer is the same
* No cloudwatch metric currently

### Bring Your Own IP

* Can create Elastic IPs with IPs from on prem address space

### Default Limits 

* 5 VPCs per account
* 200 subnets per VPC
* 5 VPC elastic IPs per region
* One IGW per VPC

### Route Tables and IGW

* All subnets must be associated with route table
* all route implicitly associated with main route table if not associated with other
* default only local routes exist 
* Must update route table when adding IGW EIGW VGW NAT Peering Endpoint 
* No NAT IPv6










# Hybrid Networking Basics and VPNs in AWS

1. VPN Suffices These Needs

* Secure Traffic
* Throughput will grow
* Connections from multiple locations
* Fast and inexpensive
  
## Virtual Private Gateway

1. General

* Managed AWS service
* Acts as a router between your VPC and non-aws-managed networks
* Can be associated with more than one external connection
* Single attached VPC at a time
* Can detach

1. Configuration
    
* tag name value
* assign ASN number
* attach to vpc

3. ASN
   
* identify AS needed to use bgp peering connections
* 16 bit at first now 32 bit as well 
* private ASN is 64512-65534 for 32 bit 42000000000+
* VGW must assign private ASN default is 64512 previously 7224

4. Takeaways
* Single VPC multiple external 
* must have private ASN to support BGP routing
* can not edit properties after assigned such as ASN

## AWS Route Learning

Route Tables 
* Know dest networks 
* Know best next-hop target
* Route learning keeps them up to date
* Local routers know their local networks 
  
How do routers share their routes?
* Static
  * Network prefixes manually configured
  * Limited means of applying route preferences
  * Unsuitable for larger networks 
* Dynamic
  * peer routers share prefixes using routing protocol
  * optional controls over route preferences
  * Widely used in large networks

Where is this used in AWS?
* AWS site to site VPN static of dynamic 
* **Direct Connect only uses dynamic** 

How do VPC learn new routes?
* statuc routes added manually 
* routes dynamically learned using route propagation
  * enable route propagation within route table for the VGW in VPC
  * target for VGW is VGW-xxx 
  * overlapping CIDR will cause problems with CIDR ranges for propagated routes
* Conflict Route
  * **most specific route preferred (longest subnet mask) but local route always is preferred** 
  * VPC static routes take preference over matching propagated routes
  * **propagated conflicts (Direct connect > static route > VPN)**
  
## BGP Introductions 

Overview
* Exterior routing protocol
* share routes in one AS to another
* network routes prefixes are shared between mutually configured peers
* No prefixes automatically shared
* select best path to the destination 
* TCP 179 must be open for BGP
* Interior Gateway protocol
  * How its learns about networks in other AS
  * routing within an AS
* eBGP - peering in BGP routers in seperate AS
* iBGP - peering in BGP routers in same AS
* about 63000 AS in existence globally
* GBP only sees peer connections

## BGP prefixes and preference 
BGP Table 

* Network
  * (>) at beginning means best path on table
* Next Hop
  * the next router
* Metric
* Local Pref
* Weight
  * learned internal routes are always 0
  * internal are always 32768
* Path
  * i means internal
  * preceding values before i are autonomous systems
  

Logic
* Highest weight
* highest local pref
* shortest AS Path
* origin type (eBGP iBGP) Exterior over internal neighbor routes
* lowest metric

BGP only advertises best path to a AS 

Cloud Hub (VGW feature)

* feature of VGW advertises learned prefixes of connected external routes to each of the external AS then allows the AS to connect with each others
* Need dynamic routing

## BGP Prefix Selection Control

BGP has no sense of network quality (bandwidth)

LOCAL
* Manually change higher Weight of BGP routes (in scope of local router)
  * Weights only applies to the local BGP router (local only property)
* Change local pref to higher if want all other AS routers to know to take the other path 
  * shared with peered routers
* Need to change both of these to enable best path due to bandwidth in an AS

VGW
* Cant change weight on VGW or local pref (cannot edit BGP configuration)
* Can use AS Path Prepending to add another AS on AS path making AS path longer, also advertised 
* metric (multi exit discriminator) can be increased to make lower prefered path, not advertised passed peers 


Weight local pref control outbound routes

## VPN and IPsec Overview
Secure logical network connection on top of a less secure physical network connection

Types 

1. Site to Site VPN (VPN Tunnel)
2. Client to Site VPN 

* AWS managed site to site vpn only allow ipsec tunnels 

VPN endpoints on each systems must be pre-configured (IPsec)
* Identity of other endpoint
* shared auth method
* security policies 
* Interesting traffic to enable a new tunnel

Establishing IPsec tunnel

1. Interesting traffic discovered
2. Internet Key Exchange (IKE) 
   * negotiate a security policy for key exchange
   * how to encrypt etc.
   * also known as main mode
   * preform key exchange (diffie-hellman) 
   * generates a phase 1 tunnel or single two way ike security association between the peers
3. IKE phase 2 (IPSec)
   * refered to as quick mode 
4. IPSed tunnel established
   * Two, one-way IPSec security associations between networks
5. IPsec tunnel terminated after timeout

Whats a Security Association?
* Connection where parties use same security rules 
* requires parties support more than one security associations at the same time

Types of VPNS 

Policy-Based

* Admin-configured rule set define vpn permitted traffic
* one sec association created per matched rule set

Route-Based

* Traffic must point to VPN network destination 

VPN tunnels only establish from on prem to AWS never from AWS

VPN connections are not persistent

AWS site to site connections only support IPsec and IPv4

## Customer Gateways

AWS site to site vpn components 

1. Configure VGW or TGW
2. confirm customer gateway device as customer location
3. configure AWS customer gateway (identify the customer gateway device on aws)
4. COnfigure VPN connection
5. configure VPC route table
6. Configure VPN setting on CGD

CGD requirements

1. Must support  IKE 
2. must support IPsec
3. CGD must bde accessible by static public IPv4 address
4. Must support dead peer detection
   * Recognize when one peer is dead and move to other
5. BGP support is optional
6. ports UDP 500 must be open and IP protocol 50 enabled (firewall)
7. with NAT traversal include UDP 4500

CGW AWS
1. Name-tag value
2. DYnamic or staic routing (ASN if dynamic)
3. CGD public IP address of device or NAT-T device
4. Optional ACM generated certificate for IKE auth (need cert at both AWS and device) if not AWS genereated preshared key automatically

VPN config parameters (can be changed after created)
1. Name tag
2. VGW ID
3. Existing CGW ID or create new CGW (new CGW will automatically have dynamic routing)
4. Dynamic or Static routing in VPN
5. Tunnel Options 

* AWS side actually have two endpoints in different AZs
* One active one passive (reason for dead peer Detection)
* Internal range of VPN must be a /30 CIDR block in the 169.254.0.0/16 (only two network nodes endpoint of AWS and Customer Endpoint)
* VPN charges start after VPN endpoints created (charged per hour 5 cents)
* Download CGW configuration under created connection in amazon VPN

AWS site-to-site vpn categories 
* Created before Feb 6, 2019 are AWS classic VPNs
* New are AWS VPNs
* Need to replace old ones by creating totally new VPN connection and new Virtual Gateway
* AWS VPNS only support IPSec tunnels over IPv4
* AWS VPN provide two tunnels
* AWS provides downloadable instructions for tunnel configuration for both tunnels
* Route propagation will only occur if there is an active vpn connection 

## VGW And VPN Limitations

* VGW are not VPC transitive
* VPN connections are capped at 1.25 Gbps per VGW(Cant use link aggregation VGW will throttle connections)
* VGW will only use one tunnel at a time
* VPN IPsec tunnel only supports one security association pair
* Make one rule policy or use route based vpn
* only supports IPsec protocol
* only supports IPv4
* cannot recieve client to site connections

## Other VPN Options in AWS

* EC2 with VPN software
* AMIs for such in the AWS marketplace
* EC2 VPNs ussually for more advanced architectures
* Software based VPNs can forward traffic to other VPCs (VPN connection now terminates at EC2 instead of VGW)
* can used VPN within AWS to suffice encryptian compiance requirements 
* AWS only supports unicast but with a VPN within the VPC creates a logical isolated and can multicast to all the tunnels

AWS Client VPN service

* AWS managed open VPN based service
* provides VPN endpoint for individual client systems
* Can authenticate using AD(must be aws AD) or private certificates.

How to use?

1. Define server or client certs in ACM 
2. Create Client VPN endpoint
3. Associate Endpoint with VPC subnet. (Billing starts)
4. Configure Acess controls
5. Download .ovpn file to client
   
* Can access everything in VPC including gateways and peered VPCs

CLIENT VPN CONSIDERATIONS
* VPN client route table
* VPN security group on endpoint NI
* VPN authorization rules (Allows access to certain CIDR rangers) Set to AD users

Split Tunneling

* allows both client VPN endpoint route table and local route table to be used for client

Takeaways 

* Software VPN overcome limitation but not managed so variable preformance and availability
* Client VPN is managed and uses OPEN VPN

## AWS VPN Monitoring and Optimization 

Is it working the way we expect?

* Cloud Watch Metrics 
  * TunnelState
  * TunnelDataIN
  * TunnelDataOut
* EC2 Standard Metrics 
  * must be published to CloudWatch
* Client VPN Cloud Watch Metrics
* VPC flow logs
* EC2-published Log Streams
* AWS client VPN log authentication attempts
* Cloud Trail does not log network traffic

Can Performance be Improved?

* Client and site to site managed by aws
* EC2 is tied to instance type and size
* EC2 supports advanced networking
* VPN software may limit throughput
* Some VPN systems can seperate encryptian and tunnel functions.

What Can Make it Stop Working
* USER ERROR
  * Route Tables
  * Security Groups
  * NACLs
  * Authentication 
  * Customer Gateway Devices
  * Open VPN Client Configurations
  * Insufficient Permissions 
  * To use these IAM is not need only to administate 
  * To acheive HA set up two customer gateways
  * Set up two subnet Associations with client VPN to make it HA

## VPN Cost Optimization
* Endpoint subnet associations cost 10 cents an hour

# Direct Connect (DX) & Hybrid DNS

Whats wrong with VPNs?

* only initiated connection on prem
* VPN uses public infrastructure
* data transfer use out to internet billing
* limited to 1.25 Gbps

Benefits of Direct Connect

* Connection is always on
* uses dedicated infrastructure 
* reduced charge for data transfer
* flat hourly rate per port 
* 10 Gbps max speed per DX
* multiple DX connections may be created and used simultaneously 

What is Hybrid DNS?

* enables private DNS res across connected on prem and cloud resources

## Direct Connection Locations and Hardware

How does it work

* dedicated high-throughput low-latency connections to aws region
* Access via globally placed DX locations
* Customer traffic is isolated using VLANs

Hardware Config Requirements

* Single-mode fiber
* Either 1000Base-lx or 10GBase-lr transcievers
* disable auto-negotiation on all ports
* 802.1Q VLAN encapsulation suported on all ports
* support of GBP and GBP md5 auth
* Bidirectional recommended not required

## DX Connections

*Must establish connection to DX location to on prem

What is a Dedicated Connections (customer managed hardware, must work with DX location staff)

* AWS hardware at DX location connects directory to customer hardware
* Supports either 1 Gbps or 10 Gbps connection speeds to AWS Region
* Soft Limit of 10 dedicated connections per regions

Creation Process for dedicated connection

* connections request with some info 
* may take 72 hours to respond 
* can only modify tag keys and values for DX connection must submit new requrest to change
* LOA CFA authorizes DX location to connect your hardware to specfic port on AWS owned hardware
* Valid for 90 days
* DX location connects your hardware once you send LOA-CFA to DX location

Host DX location (faster and more flexible)

* AWS hardware at DX location connects directly to AWS DX partner managed hardware
* Supports bandwidth
  * Mbps 50,100,200,300,400,500
  * Gbps 1,2,5,10

Host Creation Process 

* AWS partner creates connection
* AWS customer must accept it
* can see connection in console after 

