Table of Contents

# High Availability and Scalability: ELB & ASG

## High Availability, Scalability and Load Balancing

Scalability means than an application/system can handle greater loads by adapting.
There are two kinds of scalability:

1. Vertical scalability;

2. Horizontal scalability (=elasticity).

>  *Scalability is linked but linked to high availability.*

High availability means running your application/system in at least 2 data centers (AZ).
The high availability can be:

1. Passive (for RDS Multi AZ for example);

2. Active (for horizontal scaling).

> The goal of high availability is to survive a data center loss. 

Load balancers are servers that forward traffice to multiple servers downstream.

*Why use a load balancer?*

- Spread load across multiple downstream instances.

- Expose a single point of access (DNS) to your application.

- Seamlessly handle failures of downstream instances.

- Do regular health checks to your instances.

- Provide SSL termination (HTTPS) for your websites.

- Enforce stickiness with cookies.

- High availablility across zones.

- Separate public traffic from private traffic.

## ELB Overview

An Elastic Load Balancer is 

- a <span style="color:yellow">managed load balancer</span>,
  
  - AWS guarantees that it will be working
  
  - AWS takes care of upgrades, maintenance, high availability
  
  - AWS provides only a few configuration knobs

- cheap to setup your own load balancer but it will be a lot more effort on your end

- integrated with many AWS offerings/services
  
  - EC2, EC2 Auto scaling groups, ECS
  
  - AWS Certificate Manager (ACM), CloudWatch
  
  - Route 53, AWS WAF, AWS Global Accelerator

### Health Checks

### Sticky Sessions

### Cross Zone Load Balancing

### SSL Certificates

### Connection Draining

## CLB

**Hands-On**

## ALB

**Hands-On**

## NLB

**Hands-On**

## GWLB

**Hands-On**

### ASG Overview

**Hands-On**

### Scaling Policies
