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



## EC2 instance type basics

## Security Groups and classics ports overview

## SSH Overview

## SSH Troubleshooting

## EC2 Instance connect

## EC2 Instance roles demo

## EC2 Instance launch types



