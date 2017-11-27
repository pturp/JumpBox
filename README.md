# JumpBox
AWS Public/Private relationships between 2 instances  

## Objective: How to connect to a private EC2 through another public one and getting access to internet from the private
Use case: 2 servers , one as Web front-end, the other one as a back-end (eg: Database). For obvious reason, the back-end should not be accessible from internet.  

### Needs  

1 VPC   
2 subnet associated with the created VPC, with one being public, the other one private 
2 EC2 Instances  
1 IGW (internet Gateway) 
1 Nat Gateway  
2 Routes  
___

### VPC and subnets
CIDR: 10.0.0.0/16 . 
Name: VPC\_PUBLIC\_PRIVATE

Public subnet (part of the VPC VPC\_PUBLIC\_PRIVATE):  
CIDR 10.0.0.0/24  
Name: Public_subnet  

Private subnet (part of VPC\_PUBLIC\_PRIVATE:  
CIDR 10.0.1.0/24  
Name: Private_subnet  
____

### EC2 instances (2)  
Name: Public\_EC2  
Member of the Public\_subnet  

Name: Private\_EC2  
Member of the Private_subnet  

### The Internet gateway  
Name: IGW\_public\_private  
Member of the VPC VPC\_Public\_Private  

### NAT gateway  
Name: Nat\_Gateway\_in\_public  
Member of Public\_Subnet  
**Warning**: you need an Elastic IP 

### Route Tables
Name: Route\_Public (must be flagged as main table)  
Everything going to 10.0.x.x will be routed to the local adresses, towards internet through the internet gateway otherwise.  

Destination   |Target        |Status |Propagated        
--------------|--------------:|-------:|----------
10.0.0.0/16   |local         |Active |No
0.0.0.0/0     |igw-a823c8cf  |Active |No

Name: Route\_Private

Everything going to 10.0.x.x will be routed to the local adresses, towards internet through the NAT gateway otherwise.  

Destination   |Target                 |Status |Propagated   
--------------|-----------------------|-------|----------
10.0.0.0/16   |local                  |Active |No
0.0.0.0/0     |nat-07bf7c928a462a737  |Active |No

## Validation:

The EC2 Public\_EC2 should be accessible from the internet  
The EC2 Private\_EC2 should **NOT** be accessible from the internet  

To access the EC2 Private\_EC2 after having logged in the Public\EC2, we need to put the public-private key file to be transferred to the Public\_EC2 within the EC2 user home.   

> ssh -i "LinuxAccess.pem" ec2-user@52.208.10.217  
> sftp "LinuxAccess.pem" ec2-user@52.208.10.217  
> put LinuxAccess.pem   

Then from the public EC2 we can get access to the private and chacking we can send-receive from internet thru the NAT Gateway  

> ssh -i "LinuxAccess.pem" ec2-user@10.0.1.10  
> uname -a  
> Linux ip-10-0-1-10 4.9.62-21.56.amzn1.x86_64 #1 SMP Thu Nov 16 05:37:08 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux  
> ping www.google.com  
> PING www.google.com (74.125.197.106) 56(84) bytes of data.  
> 64 bytes from 74.125.197.106: icmp_seq=1 ttl=28 time=143 ms  
> ...
