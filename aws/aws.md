# AWS

## Services

### S3

S3 is an object storage service.

#### AWS S3 Standard

* Used for frequently-accessed data
* Low latency; high throughput
* 99.99% availability

#### AWS S3 Intelligent-Tiering

* Used when access frequency is unknown
* Automatically moves data to most cost-effective access tiers, based on access frequency
* 99.9% availability

#### AWS S3 Express One Zone

* Faster than S3 Standard, but limited to a specific availability zone
* 99.95% availability

#### AWS S3 Standard-Infrequent Access (Standard-IA)

* Used when data is accessed less frequently, but requires rapid retrival
* Single-AZ
* 99.5% availability

#### AWS S3 Glacier

* Lowest-cost storage option
* Has three storage retrieval classes: Instant Retrieval (milliseconds), Flexible Retrieval (minutes-to-hours), and Deep Archive (hours).


### ELB

[Elastic Load Balancers](https://aws.amazon.com/elasticloadbalancing/) distribute network traffic across targets/applications in one or more AZs.

They consist of application, gateway, network, and classic load balancers.

#### ALB

![](alb.png)

Application load balancers operate at OSI Layer 7 (HTTP/HTTPS).

#### GLB

![](glb.png)

Gateway load balancers deploy/scale/manage things like firewalls, intrusion detection and prevention systems, and packet inspection systems. They operate at OSI Layer 3 (the network layer).

#### NLB

![](nlb.png)

Network load balancers operate at OSI Layer 4 (TCP/UDP).

#### CLB

Classic load balancers operate at OSI Layer 4 and 7 (TCP/SSL/HTTP/HTTPS).

### VPC

AWS VPC is a "virtual private cloud".

### EC2

#### Pricing

* On-demand instances are billed by the hour depending on their instance class. You pay full price for an on-demand instance.
* Spot instances are offered at a (up to) 90% discount. These are "spare" EC2 instances that can be evicted at any time. The use-case is highly niche - stuff like data analysis, batch jobs, background processing, and optional tasks. 
* Under the "savings plan" model, the end-user commits to a certain amount of hourly usage over the course of 1 (or 3) years. This is very similar to the "reserved instance" model, but intended to be a bit more flexible.
* Reserved instances offer a (up to) 72% discount, but you lease the server per-instance-class for 1-3 years.

### EMR (Elastic Map Reduce)

EMR is a managed Hadoop framework. Hadoop is an Apache tool that does "MapReduce"-based analysis on big data, which is used to do parallelized data analysis.

### Athena

Athena does SQL queries against S3 buckets.

### ElastiCache

ElastiCache is a managed Redis/Memcached service.

### DynamoDB

DynamoDB is a schemaless NoSQL database service. It scales without incurring downtime. 

### Trusted Advisor

Trusted Advisor offers recommendations related to cost optimization, performance, security, fault tolerance, service limits, and operational excellence.

It's sort of like a best-practices guide.

### Inspector

Inspector is a vulnerability management scanning tool, targeting EC2 instances, ECR container images, and AWS Lambda functions.

### Shield

Shield is a DDoS protection service.

## Concepts

### IaaS vs. PaaS vs. SaaS

* "Infrastructure-as-a-Service" refers to providing infrastructure, such as networking, virtual machines, and storage (i.e. AWS EC2).
* "Platform-as-a-Service" refers to providing development environments with automated deployments (i.e. AWS Lambda). 
* "Software-as-a-Service" refer to providing a completely managed software application over the internet.
