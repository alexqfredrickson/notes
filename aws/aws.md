# AWS

## Services

### IAM

IAM (identity and access management) delegates AWS permissions to *users*, *groups*, and *roles*.

IAM allows you to generate and download reports that list users, and the status of their credentials (such as passwords, access keys, and MFA devices).

#### Principals

IAM principals include users, roles, and the root user.

#### Entities

IAM entities include users and roles.

#### Identities

IAM identities include users, roles, and groups.

#### Users

IAM users represent individuals. Individuals can be assigned to zero-to-many groups. They have their own credentials, independent from root account credentials.

#### Groups

Groups are collections of IAM users. Groups cannot contain nested groups.

#### Roles

IAM roles are sort of like IAM users, but roles are assumable by whoever needs them. They do not have passwords or access keys like users do. Roles are associated with short-term sessons and have temporary security credentials.

Roles are used for:

1. Federated user access (in the event that you have an external identity provider)
2. Temporary IAM user permissions
3. Cross-account access (i.e. to delegate privileges to users defined in other AWS accounts)
4. Cross-service access (three types: "forward access sessions", "service roles", and "service-linked roles")
5. EC2 instances (these also use IAM instance profiles)

##### Service-linked Roles

A service-linked role is a role associated with, and assigned to, an atomic AWS service. This is similar to how permissions are delegated to IAM users, but service-linked roles are assigned to (and owned by) AWS services themselves.

#### Policies

Policies are attached to IAM identities (i.e. users, groups, roles), defining their permissions. 

Policy statements are JSON documents that include a *version* and a *statement*. The *statement* is a list of (a) effects, (b) actions, (c) resources, and (d) optional conditions.

Policy types include:

1. Identity-based policies
2. Resource-based policies
3. Permissions boundaries
4. Organizations SCPs
5. ACLs
6. Session policies

##### Identity-based policies

Identity-based policies are either (a) managed or (b) inline policies attached to IAM identities.

##### Resource-based policies

Resource-based policies are associated with services (i.e. resources); for example: S3 bucket policies. They don't have to be in the same account.

##### Permissions boundaries

Permission boundareis are managed policies that define the *maximum* permissions that identity-based policies can grant to entities. They do not grant permissions.

##### Organizations SCPs

AWS Organizations "service control policies" define maximum permissions for organizational units. They do not grant permissions.

##### ACLs

Access control lists define which principals in cross-accounts can access ACL-bounded resources. They are like resource-based policies, but do not use JSON.

##### Session policies

Session policies are used in tandem with the AWS CLI/API to assume specific roles, thereby limiting the persmissions that are granted to the session.

### IAM Identity Center

IAM Identity Center is an SSO solution that wraps over IAM. It is a free service.

### S3

S3 is an object storage service.

#### Storage Tiers

S3 bills you for stored objects, depending on their access rates and retrieval speed.

##### AWS S3 Standard

* Used for frequently-accessed data
* Low latency; high throughput
* 99.99% availability

##### AWS S3 Intelligent-Tiering

* Used when access frequency is unknown
* Automatically moves data to most cost-effective access tiers, based on access frequency
* 99.9% availability

##### AWS S3 Express One Zone

* Faster than S3 Standard, but limited to a specific availability zone
* 99.95% availability

##### AWS S3 Standard-Infrequent Access (Standard-IA)

* Used when data is accessed less frequently, but requires rapid retrival
* Single-AZ
* 99.5% availability

##### AWS S3 Glacier

* Lowest-cost storage option
* Has three storage retrieval classes: Instant Retrieval (milliseconds), Flexible Retrieval (minutes-to-hours), and Deep Archive (hours).

#### Replication

Objects can be replicated across regions, or into the same regions.

Cross-region replication requires that you enable versioning on the bucket. It also requires an IAM policy to allow S3 to replicate objects on your behalf.

### ELB

[Elastic Load Balancers](https://aws.amazon.com/elasticloadbalancing/) distribute network traffic across targets/applications in one or more AZs.

They consist of application, gateway, network, and classic load balancers.

Load balancers accept incoming traffic from clients and route the traffic to registered targets in availability zones. They also monitor health of the registered targets. 

Load balancers use "listeners", wich are processes that check for connection requests, with configurations including protocols and port numbers.

Load balancers are isolated to particular availability zones. Application load balancers require a minimum of two targets in separate availability zones.


#### ALB (Application Load Balancers)

![](alb.png)

Application load balancers operate at OSI Layer 7 (HTTP/HTTPS).

#### GLB (Gateway Load Balancers)

![](glb.png)

Gateway load balancers deploy/scale/manage things like firewalls, intrusion detection and prevention systems, and packet inspection systems. They operate at OSI Layer 3 (the network layer).

#### NLB (Network Load Balancers)

![](nlb.png)

Network load balancers operate at OSI Layer 4 (TCP/UDP).

#### CLB (Classic Load Balancers)

Classic load balancers operate at OSI Layer 4 and 7 (TCP/SSL/HTTP/HTTPS).

#### SSL Cipher Suites, SSL Protocols, and Server Order Preference

Clients and load balancers negotiate SSL protocols, and SSL cipher suites as part of the SSL specification. 

Server Order Preference is optionally supported. In "Server Order Preference", the client and the load balancer both communicate to each other which ciphers and protocols they support - ranked in order of preference - and choose which ones they want to use based off of that.

### VPC

AWS VPC is a "virtual private cloud".

You can have up to five VPCs in a region by-default.

#### Elastic Network Interfaces

Elastic network interfaces are virtual network cards for VPCs.

#### Security groups

The default security group for a VPC allows all outbound traffic, restricts all inbound traffic, and allows instances within the security group to communicate with each other.

### EC2

EC2 (or "elastic compute cloud") are virtual machines in AWS.

* The default maximum capacity of EC2 instances per-region is 20. 

#### Pricing

* On-demand instances are billed by the hour depending on their instance class. You pay full price for an on-demand instance.
* Spot instances are offered at a (up to) 90% discount. These are "spare" EC2 instances that can be evicted at any time. The use-case is highly niche - stuff like data analysis, batch jobs, background processing, and optional tasks. 
* Under the "savings plan" model, the end-user commits to a certain amount of hourly usage over the course of 1 (or 3) years. This is very similar to the "reserved instance" model, but intended to be a bit more flexible.
* Reserved instances offer a (up to) 72% discount, but you lease the server per-instance-class for 1-3 years. RIs are tied to particular accounts, but note that AWS Organizations has a "consolidated billing" feature that lets you use RI leases across accounts (kinda cool!); the accounts are treated as one for the purposes of billing.

#### UserData

EC2 instances, on-launch, have the option of passing "user data" to the instance, performing automated configuration tasks. They take the form of either (a) shell scripts or (b) cloud-init directives. 

To see user data, view EC2 → Actions → Instance settings → Edit user data.

Userdata only gets invoked at launch time. On CFN update, it does not get invoked.

#### Placement Groups

EC2 placement groups can be used when EC2 instances are highly interdependent. They come in three flavors:

1. Cluster placement groups 
2. Spread placement groups
3. Partition placement groups

They are *free* and *optional*. 

##### Cluster Placement Groups

Cluster placement groups packs instances close together in the same AZ. This lowers network latency. This can be used for HPC applications.

##### Spread Placement Groups

Spread placement groups spread instances across distinct underlying hardware to reduce correlated failures.

##### Partition Placement Groups

Partition placement groups partition instances across an AZ so that they maintain unique underlying hardware, with unique networks and power sources. This can be used for larger distributed workloads (i.e. Hadoop, Cassandra, and Kafka).

#### Instance Store

EC2 Instance Store Temporary Block Storage is a feature that allows an EC2 instance to attach to an ephemeral block storage volume. The use case is if you need a temporary cache/buffer for an instance. 

The data is persisted on instance reboot, and deleted on instance stop/hibernate/termination.

#### Elastic IP Addresses

Elastic IP addresses are IP addresses that are accessible from the public internet, and that map to either EC2 instances or network interfaces.

#### Enhanced Networking

"Enhanced networking" is a special type of virtualization feature for EC2 that allows for higher bandwidth, higher packet-per-second performance, lower latency between instances, and less jitter.  It's free, but `t2` instances don't support it.

### RDS 

RDS (or "relational database service") are databases in AWS.

### EMR (Elastic Map Reduce)

EMR is a managed Hadoop framework. Hadoop is an Apache tool that does "MapReduce"-based analysis on big data, which is used to do parallelized data analysis.

### Athena

Athena does SQL queries against S3 buckets.

### ElastiCache

ElastiCache is a managed Redis/Memcached service.

Redis-based ElastiCache instances only contain a single node.

### DynamoDB

DynamoDB is a schemaless NoSQL database service. It scales without incurring downtime. 

### Personal Health Dashboard

Personal Health Dashboard is a global service health/status dashboard. It's used when AWS is experiencing outages and/or service degradations.

### Trusted Advisor

Trusted Advisor offers recommendations related to cost optimization, performance, security, fault tolerance, service limits, and operational excellence.

It's sort of like a best-practices guide.

### Inspector

Inspector is a vulnerability management scanning tool, targeting EC2 instances, ECR container images, and AWS Lambda functions.

### Shield

Shield is a DDoS protection service.

### Route 53

Route 53 features include domain registration, DNS, traffic flow, health checking, and failover.  Route 53 does not support DHCP, routing or caching.

#### Routing Policies

When you create Route 53 records, you also select a "routing policy", which determines how Route 53 responds to DNS queries. The options include:

  * *Simple routing*, which are the default
  * *Failover routing*, which routes to different resources depending on instance health
  * *Geolocation routing*, which routes traffic depending on the geographic locale of the originating request
  * *Geoproximity routing*, which routes traffic based on the geographic locale of requested resources
  * *Latency routing*, when resources are located in multiple regions, and you want to route requests to regions with the best latency
  * *IP-based routing*, which routes based on IPs of users
  * *Mutlivalue answer routing*, when you want Route 53 to respond randomly to DNS requests
  * *Weighted routing*, when you want to probabilistically weight responses

### Service Catalog

Service Catalog is a portfolio of IaC templates (CloudFormation and/or Terraform), which Service Catalog administrators can "approve" for use on AWS. It's not entirely clear to me why you'd want to use this, and it was probably built out for a particular enterprise use-case.

### OpsWorks

OpsWorks is an AWS-managed Chef/Puppet service.

### Direct Connect

Direct Connect is a service that allows your network to privately connect directly to AWS via fiber-optic cables. The use-case is increased bandwidth throughput and bypassing ISPs. 1 Gbps and 10 Gbps speeds are available.

### KMS

AWS KMS (Key Management Service) creates cryptographic keys that can be used natively with specific services like EC2, EBS, and S3 (etc.).

### CloudFormation

AWS CloudFormation is AWS' infrastructure-as-code service.  

A CloudFormation stack is a group of AWS resources defined in CloudFormation IaC. Stacks may import and export values from other stacks. A stack that contains exports cannot be deleted until each stack that leverages those exports (as imports) are deleted first.

### Polly

Amazon Polly is a text-to-speech service. It's a bit creepy.

### Site-to-Site VPN

A Site-to-Site VPN is used to configure a VPC to talk to an on-premise network. They conceptually include:

  * *VPN connections*, which connect on-premise equipment with VPCs
  * *VPN tunnels*, through which encrypted data flows between customer networks and AWS
  * *Customer gateways*, which provide information about customer gateway devices
  * *Customer gateway devices*, which are physical devices that sit on the customer side of the VPN connection
  * *Target gateways*, which represent the on-premise VPN gateway on the AWS side of the VPN connection
  * *Virtual private gateways*, which represent the site-to-site VPN endpoint connects to an AWS VPC
  * *Transit gateways*, which connect multiple VPCs and on-premise networks, and which also represent VPN endpoints on the AWS site of the VPN connection

### Redshift

Amazon Redshift is a managed, petabyte-scale data warehouse servie. It is less-configured than a data warehouse. It leverages intelligent-scaling. It does not incur charges while idle. Redshift can be queried using its own syntax, or by way of using SQL queries.

Redshift is designed for OLAP processing (by contrast, RDS is designed for more traditional OLTP processing).

Redshift uses four KMS keys: (1) an AES-256 data encryption key, (2) a database key, (3) a cluster key, and (4) a root key.

### SNS

AWS SNS (Simple Notification Service) delivers messages asynchronously from "publishers", to "topics", to "subscribers". Subscribers are services like Kinesis Data Firehose, SQS, Lambda, HTTP, email, mobile push notifications, and SMS (text).

An "SNS topic" is a communication channel between a publisher and subscriber. Topics can group multiple endpoints. Once a message is published to a topic, it cannot be deleted. 

SNS subscribers receive all messages published to topics, or they can filter them depending on its policy.

SNS is associated with AWS KMS.

### SQS

AWS SQS (Simple Queue Service) is a queueing service allowing you to integrate and decouple distributed software systems and components. 

SQS queues can be "standard", guaranteeing that messages are queued at-least-once, or "FIFO" (first-in, first-out) - which guarantees that messages are send exactly-once.

In general, producers send messages to queues, which are consumed, triggering a "visibility timeout period". The message is then deleted from the queue during the visibility timeout period. Messages stay in a queue for up to 4 days by-default. They can be configured to stay in a queue for 60 seconds to 14 days.

Once a message is sent to an SQS queue, it can be received and deleted. Messages are not automatically deleted after retrieval - you have to be explicit.

## General Concepts

### IaaS vs. PaaS vs. SaaS

* "Infrastructure-as-a-Service" refers to providing infrastructure, such as networking, virtual machines, and storage (i.e. AWS EC2).
* "Platform-as-a-Service" refers to providing development environments with automated deployments (i.e. AWS Lambda). 
* "Software-as-a-Service" refer to providing a completely managed software application over the internet.
