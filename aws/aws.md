# AWS

## Services

### IAM

IAM (identity and access management) delegates AWS permissions to *users*, *groups*, and *roles*.

IAM allows you to generate and download reports that list users, and the status of their credentials (such as passwords, access keys, and MFA devices).

It is a global service, and is not region/AZ-specific.

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

Prior to 2020, S3 was "*eventually consistent*" - meaning a `PUT` call would not be immediately visible to `GET` or `LIST` requests. It is now [*strongly consistent*](https://aws.amazon.com/blogs/aws/amazon-s3-update-strong-read-after-write-consistency/).

#### Storage Tiers

S3 bills you for stored objects, depending on their access rates and retrieval speed.

##### AWS S3 Standard

* Used for frequently-accessed data
* Low latency; high throughput
* 99.99% availability

S3 Standard replicates data across several availability zones in a region.

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

Glacier may have a "vault lock policy" attached to it, which is an immutable policy.

#### Replication

Objects can be replicated across regions, or into the same regions.

Cross-region replication requires that you enable versioning on the bucket. It also requires an IAM policy to allow S3 to replicate objects on your behalf.

#### Server Access Logging

You can enable logging to derive the following bucket specific information: the bucket owner, the bucket, the action time, the IP address (or requester and request ID), the type of operation, the S3 bucket key, HTTP response status, etc.

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

#### Internet Gateways

An internet gateway is a horizontally scaled VPC component that allows resources in public subnets of the VPC to communicate back-and-forth with the internet over IPv4 and IPv6, provided they have a public IP address.

Internet Gateways are targeted by VPC route tables to route traffic out to the internet. Internet gateways perform network address translation (NAT).

#### Elastic Network Interfaces

Elastic network interfaces are virtual network cards for VPCs. These are attached to EC2 instances launched in a particular availability zone. Each has a private IP address associated with the subnet that it sits in. They cannot be moved across subnets. They can be attached or unattached from EC2 instances freely.

If multiple network interfaces are attached to an instance (in different subnets or different VPCs), the instance is said to be "dual-homed".

#### Security groups

The default security group for a VPC allows all outbound traffic, restricts all inbound traffic, and allows instances within the security group to communicate with each other.

#### VPC Endpoints

A VPC endpoint is used to privately connect to specific AWS services. This is integrated with AWS PrivateLink. The traffic in these cases never leaves the AWS network.

There are two types of VPC endpoints: *interface endpoints* and *gateway endpoints*.

##### Interface endpoints

Interface endpoints are related to AWS PrivateLink. It is a collection of elastic network interfaces with a private IP address that serves as an entry point for traffic destined to a supported service.

##### Gateway endpoints

Gateway endpoints target specific IP routes in an Amazon VPC route table. These are used by DynamoDB and S3. Gateway endpoints are unrelated to AWS PrivateLink.

##### Network ACLs

A network access control list is an optional feature that allows you to specify firewall rules for subnets. This is similar to security groups, but more granular. There is no charge for their usage.

NACLs are stateless, meaning information about previously sent/received traffic is not saved. Security groups are stateful. For instance: an EC2 instance with inbound traffic allowed is allowed to communicate outbound responses with the message sender.

##### Subnets

Subnets (sub-networks) are ranges of IP addresses allocated within a VPC.

A subnet is **private** if it is not attached to an internet gateway. They need NAT devices to access the internet.

A subnet is **public** if it is connected to an internet gateway.

Subnets can communicate with each other by-default in an AWS VPC with default settings.

#### Routes

Routes are rules (kind of like security groups) that define how subnet/gateway traffic is directed. 

#### NAT Devices

Network Address Translation (NAT) devices allow resources in private subnets to communicate with the internet (or other VPCs, or on-premise networks). They cannot receive unsolicited requests.

A NAT *gateway* can be public or private. If it is public, it allows for outbound internet traffic.  If it is private, it allows for outbound intra-VPC/on-prem traffic. NAT gateways have to be situated in subnets that are distinct from the instances that use them, because NAT gateways route to internet gateways, and instances route to NAT gateways - the operative term being *route*, which defines relationships between subnets/gateways.

A NAT *instance* sits in a public subnet, and allows instances in private subnets to perform outbound internet traffic, by way of the NAT instance, to an Internet Gateway that allows for public internet access. These are end-of-life in favor of NAT gateways.

You seat NAT instances inside VPCs, configure a security group for it, create a NAT AMI, and then create a NAT instance. The NAT instance must have source/destination checks disabled on itself. The NAT instance also its inside of a private subnet, which in turn contains a route table, which in turn contains routes that need to be updated such that NAT instance may communicate with outside resources (such as the Internet).

### EC2

EC2 (or "elastic compute cloud") are virtual machines in AWS.

The default maximum capacity of EC2 instances per-region is 20. 

Private IP addresses for EC2 instances fall between 10.0.0.0 and 10.255.255.255, 172.16.0.0 and 172.31.255.255, and 192.168.0.0 and 192.168.255.255.

#### Auto Scaling

An Auto Scaling Group (ASG) is a collection of EC2 instances that are grouped together for automatic scaling.  You can implement health-check based replacements of instances.

The "desired capacity" determines the amount of instances you want to host by-default. Desired capacity is distributed evenly among availability zones.

Scaling policies modulate instance count between the "minimum" and "maximum" capacity.  The ASG can (1) maintain current levels, (2) be manually scaled up and down, (3) be scheduled to scale up and down, or (3) dynamically change capacity based on traffic changes.

ASGs support both on-demand and spot instances.

ASGs leverage *launch templates*, whichspecify instance configuration, including AMIs, instance types, key pairs, security groups, etc. Launch templates are versioned.

You can set policies such as step scaling, and simple scaling. Scaling policies are invoked by breaching CloudWatch alarms. Simple and step scaling are almost identical, but step scaling is preferred by AWS.

#### Spot Instances

A *spot instance* uses spare data center EC2 capacity, and costs less than a typical on-demand instance. 

  * *Spot prices* are set by AWS.
  * *Spot capacity pools* are sets of spot instances with a given instance type, in a specific availabilty zone.
  * *Spot instance requests* are requests to AWS to fulfill EC2 demand when capacity is available. It can be a one-time thing, or continual (persistent). Persistent spot instance requests are filed automatically on-instance-interruption.
  * During a *spot instance interruption*, AWS terminates/stops/hibernates the given spot instance, giving you a two minute warning.
  * AWS lets you know when spot instances are at high risk of interruption via *EC2 instance rebalance recommendations*.

#### EBS

EBS, or "elastic block store", are block-storage resources that are used with EC2 instances. They can be volumes or snapshots. 

Once created, you cannot modify the resting encryption status.

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

EC2 placement groups can be used when EC2 instances are highly interdependent. They are *free* and *optional*.  They come in three flavors:

1. *Cluster placement groups*, which pack instances close together in the same availability zone: lowering network latency. This can be used for HPC applications.
2. *Spread placement groups*, which spread instances across distinct underlying hardware to reduce correlated failures. 
3. *Partition placement groups*, which partition instances across an availability zone so that they maintain unique underlying hardware, with unique networks and power sources. This can be used for larger distributed workloads (i.e. Hadoop, Cassandra, and Kafka).

Note that dedicated instances are segregated in such a way that they do not share hosts with any other AWS customer.

#### Instance Store

EC2 Instance Store Temporary Block Storage is a feature that allows an EC2 instance to attach to an ephemeral block storage volume. The use case is if you need a temporary cache/buffer for an instance. 

The data is persisted on instance reboot, and deleted on instance stop/hibernate/termination.

Instance stores are *free*.

#### Elastic IP Addresses

Elastic IP addresses are IP addresses that are accessible from the public internet, and that map to either EC2 instances or network interfaces.

#### Enhanced Networking

"Enhanced networking" is a special type of virtualization feature for EC2 that allows for higher bandwidth, higher packet-per-second performance, lower latency between instances, and less jitter.  It's free, but `t2` instances don't support it.

### RDS 

RDS (or "relational database service") are databases in AWS.

#### Multi-AZ deployments

If "multi-AZ" is configured for an instance, either one or two "standby" database instances are created.

If *one* instance is created, it provides failover support but it does not serve read traffic. This is called a "multi-AZ instance". A multi-AZ instance has one write instance and one read replica.

If *two* instances are created, it provides failover support, **and** it serves read traffic. This is called a "multi-AZ cluster". A multi-AZ cluster has one write instance and two read replicas.  

In the event of failover, the traffic is routed to a read replica automatically.

Microsoft SQL Server and Oracle RDS instances do not support read replicas.

### EMR (Elastic Map Reduce)

EMR is a managed Hadoop framework. Hadoop is an Apache tool that does "MapReduce"-based analysis on big data, which is used to do parallelized data analysis.

### Athena

Athena does SQL queries against S3 buckets.

### ElastiCache

ElastiCache is a managed Redis/Memcached service.

Redis-based ElastiCache instances only contain a single node. Memcached-based ElastiCache instances support up to 20 nodes by-default.

### DynamoDB

DynamoDB is a schemaless, serverless NoSQL database service. It scales without incurring downtime. It is fully-managed, and you don't have to patch it. A potential use-case for DynamoDB is for session storage. DynamoDB stores data in *partitions*, which are replicated across AZs within a region.

In general, DynamoDB is optimized to query against a particular primary key. Primary keys in DynamoDB can be *simple* (using only a partition key) or *composite* (a tuple leveraging a partition key and sort key). The primary key should be chosen to maximize uniformity.

DynamoDB does not support JOIN operators, and prefers denormalized schemas.

A *parition key* is mandatory, and partitions data. Data is grouped according to the partition key. A *sort key* is optional. It determines the order of data being stored along the lines of the partition key.

Primary indices are (probably) determined by the primary key. You can define secondary indices as well - either *global* or *local* - which are specific to a table.

  * A *global secondary index* is an index with a composite key (a partition key coupled with a sort key). It is called a "global" index because queries against the index span all partitions.
  * A *local secondary index* is an index with the same partition key as its base table, but which uses a different sort key.

#### API Operations

Some basic API operations include:

| Operation      | Description                                         |
| -----------    | --------------------------------------------------- |
| `GetItem`      | Retrieves a single table item.                      |
| `BatchGetItem` | Retrieves up to 100 items.                          |
| `Query`        | Retrieves all items with a specified partion key.   |
| `Scan`         | Retrieve all items in the specified table or index. |

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

Domains can be registered from Route 53, and Route 53 can also manage external domains. 

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

#### VPN CloudHub

AWS VPN CloudHub is a service provided with site-to-site VPNs that links AWS VPC virtual private gateways with customer gateways, allowing a Direct Connect-style connection from on-premise networks to AWS VPCs. It doesn't require a VPC, confusingly.

### Config

AWS Config is a view of how AWS resources are configured, how they relate, and their past configurations. You specify a resource type, set up an S3 bucket, set up SNS to send "configuration stream notifications", and then Config can record configuration changes.

Config uses a "delivery channel" to send notifications and update configuration states. 

An AWS Config "configuration item" represents attributes of a supported AWS resource, at some point-in-time. It includes metadata, attributes, relationships, current configuration, and related events. A configuration item is created whenever it detects a change to a recorded resource. Configuration items can't be manually deleted, but Config can delete them after a minimum of 30 days.

### Redshift

Amazon Redshift is a managed, petabyte-scale data warehouse servie. It is less-configured than a data warehouse. It leverages intelligent-scaling. It does not incur charges while idle. Redshift can be queried using its own syntax, or by way of using SQL queries.

Redshift is designed for OLAP processing (by contrast, RDS is designed for more traditional OLTP processing).

Redshift uses four KMS keys: (1) an AES-256 data encryption key, (2) a database key, (3) a cluster key, and (4) a root key.

### SNS

AWS SNS (Simple Notification Service) delivers messages asynchronously from "publishers", to "topics", to "subscribers". Subscribers are services like Kinesis Data Firehose, SQS, Lambda, HTTP, email, mobile push notifications, and SMS (text).

An "SNS topic" is a communication channel between a publisher and subscriber. Topics can group multiple endpoints. Once a message is published to a topic, it cannot be deleted. SNS topic names are available for re-use about 60 seconds after they have been deleted.

SNS subscribers receive all messages published to topics, or they can filter them depending on its policy.

SNS is associated with AWS KMS.

### SQS

AWS SQS (Simple Queue Service) is a queueing service allowing you to integrate and decouple distributed software systems and components. 

SQS queues can be either "standard" or "FIFO":

  * A *standard* queue guarantees that messages are queued at-least-once, but messages may be delivered more than once. Standard queues can hold up to 120k messages at once. 
  * A *FIFO* queue (first-in, first-out) guarantees that messages are sent exactly-once. FIFO queues can hold up to 20k messages at once.

In general, producers send messages to queues. Once a message is sent to an SQS queue, it can be received and deleted. Messages are not automatically deleted after retrieval - you have to be explicit.

Messages are consumed, triggering a "visibility timeout period". The message is invisible to other consumers during the visibility timeout. Following the visibility timeout, the message is deleted. 

Unconsumed messages stay in the queue for up to 4 days by-default - this is known as the default message retention period. They can be configured to stay in a queue for 60 seconds to 14 days.

#### Polling

SQS queues can be polled using (1) short and (2) long polling techniques. They are short-polled by-default.

In a *short-polling* schema, responses are returned immediately, even if the SQS queue is empty. Short polling may fail to deliver messages.

In a *long-polling* schema, responses are returned as soon as they become available. This can reduce SQS costs because empty receives still cost money. The maximum long polling wait time is 20 seconds. 

#### Dead-letter queues

A dead-letter queue is a type of queue that other queues can target and store messages that have not been processed successfully in.  It is considered a best-practice to keep the queue and its associated dead-letter queue in the same AWS account and region.  Messages in a dead-letter queue can be examined, analyzed, etc.

### Lambda

AWS Lambda is a function-as-a-service tool. It operates elastically and does not require servers. 

Some use-cases include:
 
  * File processing, e.g. doing something after a file is uploaded to S3
  * Stream processing, e.g. using Lambda and Kinesis to process real-time streaming data, like application activity tracking, transaction order processing, etc.
  * Web applications, IoT backends, mobile backends

### SWF

AWS Simple Workflow Service is a way to create background jobs that have parallel or sequential steps.  It consists of:

  * *Domains*, which workflows run in. A domain can contain multiple workflows that can talk to one another.
  * *Workflows*, which are a collection of activities that do a high-level thing (e.g. "receive a customer order and take whatever actions are necessary to fulfill it"), and logic that coordinates the activities. Workflows in different domains can't talk to one another.
  * *Activity workers*, which perform activity tasks. 
  * *Activity tasks*, which represent logical units of work.
  * *Deciders*, which schedule activity tasks, and provide them to activity workers.

### LightSail

AWS LightSail is a service like Elastic BeanStalk that automatically provisions instances, containers, managed databases, CDN distributions, load balancers, SSD-storage, static IP addresses, DNS management, and snapshots.

### VM Import/Export

AWS VM Import/Export lets customers import virtual machine images from local virtual environments to EC2 instances, and export them back. 

It is a free service, but S3/EBS volumes/EC2 instances cost money obviously.

### Glue

AWS Glue is a serverless data integration service that allows end-users to analyze, and perform ETL on data from multiple sources. It is used for ML, analytics, and application development.

Glue has over 70 possible data sources. It loads data into data lakes. The data is then searched/queried using Athena, EMR, and/or Redshift Spectrum.

### Transfer Family

AWS Transfer Family transfers files in and out of S3 and EFS. 

It uses protocols such as SFTP (Secure Shell File Transfer Protocol), AS2 (Application Statement 2), FTPS (File Transfer Protocol Secure), and FTP (File Transfer Protocol). Note that the hilariously named SFTP differs from FTPS in that SFTP leverages SSH, whereas FPTS leverages TLS/SSL.

### Storage Gateway

AWS Storage Gateway comprises four separate services: S3 File Gateway, FSx File Gateway, Tape Gateway, and Volume Gateway.

S3 File Gateway is a file interface into S3, that allows retrieval of S3 objects via NFS.

### CloudTrail

AWS CloudTrail records actions taken by users, roles, or AWS services. It provides:

  1. An *event history*, a searchable and immutable record of actions taken in the past 90 days
  2. A *CloudTrail Lake*, which is a managed data lake
  3. *Trails*, which represent records of AWS activities, are stored in S3, and can be delivered to CloudWatch logs or EventBridge 

CloudTrail *events* are comprised of:

  1. *Management* events, which is related to actions performed **on** resources, and may consist of security configurations, new device registrations, rule configurations, logging, etc.
  2. *Data* events, which provide information about the operations performed **within** resources, and may consist of S3 API activities, Lambda executions, CloudTrail `PutAuditEvents`, SNS publish operations, etc.
  3. *Insight* events, which capture unusual API calls or error rates

### Kinesis Video Streams

AWS Kinesis Video Streams is a live-video-streaming-related service, allowing end-users to:

  * Stream live video from physical devices to AWS
  * Perform real-time video processing
  * Perform batch analytics on videos 

### Kinesis Data Streams

AWS Kinesis Data Streams collects, and processes, streams of data, in real-time.

Streaming data is defined as being:

  * *Chronologically-significant*, or time-series-based
  * *Continuously flowing*, with no beginning or end
  * *Unique*, meaning retransmission is limited
  * *Non-homogeneous*, meaning belonging to multiple different kinds of specifications
  * *Imperfect*, meaning data may have gaps or missing elements

### CloudFront

AWS CloudFront is a content distribution network that caches static/dynamic site content to serve to users. The data centers that cache requests are called "edge locations".

When a user requests content, CloudFront deliverrs it immediately from the edge location with the lowest latency - or it retrives it from an origin you define (like S3, MediaPackage, or HTTP servers), and then caches it to the edge location.

It is well-suited for static website content (like images, CSS files, JS files, etc.), or live-streaming video.

### Billing and Cost Management

AWS Billing and Cost Management is a suite of features that stes up billing, retrieves/pays invoices, and plans/organizes/analyzes/optimizes costs.

You get two free budgets per-account.

AWS Billing and Cost Management allows you to tag resources with key/value pairs to apply cost allocation tags to track costs. They can be user-generated, or AWS-generated. They take up to 24 hours to appear in the Billing dashboard (post-resource-tagging). They are allocated from the Cost Allocation Tags page.

###  Serverless Application Model

AWS Serverless Application Model is an open-source CLI IaC framework that deploys serverless applications. It's kind of like a CloudFormation code generation tool, but presumably with limited scope. AWS claims that it can result in a ~90% LoC reduction vis a vis CloudFormation, but I'm not sure why you'd want to accept the additional technical sprawl. `sam init` generates a local SAM project.

The delcared resources are `AWS::Serverless::*`, where `*` is an `API`, `Application`, `Connector`, `Function`, `GraphQLApi`, `HttpApi`, `LayerVersion`, `SimpleTable`, or `StateMachine`. For example, `AWS::Serverless::Function` will generate an AWS Lambda function (and associated resources like IAM roles and event source mappings).

### Cognito

AWS Cognito is a service that allows authorization, authentication, and provisionment of user directories to web and mobile applications. It leverages OAuth 2.0 access tokens and AWS credentials. It allows users to authenticate via Google and Facebook.

#### User Pools

When you want to provide authentication or authorization to an app or API, you create a Cognito User Pool. It functions as a user directory. It can also act as an OIDC identity provider or service provider. It is SAML 2.0-compliant.

#### Identity Pools

When you want to provide authenticated or anonymous users to AWS resources, use an identity pool. It issues AWS credentials for your application to serve resources to end-users. You authenticate with a user pool or a SAML 2.0-compliant service. It can issue credentials for guest users. It can be role-based or attribute-based. They do not have to be integrated with a user pool.

## General Concepts

### Regions and Availability Zones

Regions are broad, geographically isolated, unconnected areas containing availability zones. Resources are generally not replicated across regions unless specifically instructed.

Availability zones are connected with low latency, and consist of one (or more) data centers, each with redundant power, networking, and connectivity, in separate facilities. A single AZ may encompass multiple data centers. 

### IaaS vs. PaaS vs. SaaS

* "Infrastructure-as-a-Service" refers to providing infrastructure, such as networking, virtual machines, and storage (i.e. AWS EC2).
* "Platform-as-a-Service" refers to providing development environments with automated deployments (i.e. AWS Lambda). 
* "Software-as-a-Service" refer to providing a completely managed software application over the internet.
