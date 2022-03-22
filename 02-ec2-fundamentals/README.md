**Table of Contents:**



# EC2 Fundamentals

**Caution:** [AWS Budget setup](https://www.udemy.com/course/aws-certified-developer-associate-dva-c01/learn/lecture/26100826?start=30#overview)

## EC2 Basics

- EC2 or Elastic Compute Cloud = Infrastructure as a Service
- It consists in the capability of :
  - Renting virtual machines (EC2)
  - Storing data on virtual drives (EBS)
  - Distributing load across machines (ELB)
  - Scaling the services using an auto-scaling group (ASG)

### EC2 sizing & configuration options

- OS: Linux, Windows or MacOS
- How much compute power & cores (CPU)
- How much random-access memory (RAM)
- How much storage space:
  - Network-attached (EBS & EFS)
  - Hardware (EC2 Instance store)
- Network card: speed of the card, public IP address
- Firewall rules: **security group**
- Bootstrap script (configure at first launch): EC2 User Data

### EC2 User Data

- It is possible to bootstrap our instances using an EC2 User data script.
- `bootstrapping` means launching commands when a machine starts.
- The script is only run once at the instance first starts.
- EC2 user data is used to automate boot tasks such as:
  - Installing updates
  - Installing software
  - Downloading of common files from the internet
- EC2 user data script runs with the root user.

### Create EC2 instance with EC2 user data to have a website

1. We start off with an AMI (Amazon Machine Image, a template that contains the software configuration (OS, app server, and applications) required to launch your instance).
2. Then we choose Instance Type and configure instance.
3. Configure storage and add tags.
4. We then configure Security Groups (A Security Group is a set of firewall rules that control the traffic for your instance)

Few things which I noticed:

- When we start `Start` an instance, it gets a public and a private IP address.
- We can `Start`, `Stop` or `Terminate` an instance.
- Whenever we restart an instance, its private IP remains the same, but the public IP changes.

## EC2 instance type basics

An instance is a virtual server in the cloud. Its configuration at launch is a copy of the AMI that you specified when you launched the instance.

We have 7 different types of instances:

1. General purpose
2. Compute optimized
3. Memory optimized
4. Accelerated computing
5. Storage optimized
6. Instance features
7. Measuring instance performance

AWS has the following naming conventions:

```bash
m5.2xlarge
```

- `m` - instance class
- `5` - generation (AWS improves them over time)
- `2xlarge` - size within the instance class

Use [EC2Instances.info](https://instances.vantage.sh/) for easy Amazon EC2 instance comparison.

### EC2 - General Purpose

General purpose instances provide a balance of *compute*, *memory* and *networking* resources, and can be used for a variety of diverse workloads. 
They are ideal for applications that use resources in equal proportions such as web servers and code repositories.

### EC2 - Compute Optimized

Compute optimized instances are ideal for compute bound applications that benefit from high performance processors. 
They are ideal for batch processing workloads, media transcoding, high performance web servers, high performance computing (HPC), scientific modeling , dedicated gaming servers and ad server engines, machine learning inference and other compute intensive applications.  

### EC2 - Memory Optimized

Memory optimized instances are designed to deliver fast performance for workloads that process large data sets in memory.
It is ideal for memory-intensive applications such as :
1. high-performance, relational/non-relational databases, 
2. distributed web scale cache stores (in-memory caches),
3. in-memory databases optimized for BI (business intelligence),
4. applications performing real-time processing of big unstructured data.

### EC2 - Storage Optimized

Storage optimized for storage-intensive tasks that require high, sequential read and write access to large datasets on local storage. They are optimized to deliver tens of thousands of low-latency, random I/O operations per seconds (IOPS) to applications.
Usecases:
1. High frequence online transaction processing (OLTP) systems,
2. Relational & NoSQL databases,
3. Cache for in-memory databases,
4. Data warehousing applications,
5. Distributed file systems 

## Security Groups and classics ports overview

### Introduction to Security Groups

Security groups are the fundamental of network security in AWS.
They control how traffic is allowed into or out of our EC2 instances.
Security groups only contain **<span style="color:green">allow</span>** rules. They can reference by IP or by security group.

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/e036c1e99108e5d21bc2f41b22b52f174c326970/02-ec2-fundamentals/assets/security-group-introduction.svg)

### Security Groups - A deep dive

Security groups are acting as a *firewall* on the EC2 instances. 
They regulate:

1. access to ports
2. authorized IP ranges - IPv4 and IPv6
3. control of inbound network (from other to the instance)
4. control of outbound network (from the instance to other)


Type  | Protocol | Port Range | Source | Description |
---------|----------|---------|---------|---------| 
 HTTP | TCP | 80 | 0.0.0.0/0 | test http page |
 SSH | TCP | 22 | 122.149.196.85/32 | |
 Custom | TCP | 4567 | 0.0.0.0/0 | java app |

#### Security Groups Diagram

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/02-ec2-fundamentals/assets/security-group-diagram.svg)









## SSH Overview

## SSH Troubleshooting

## EC2 Instance connect

## EC2 Instance roles demo

## EC2 Instance launch types



