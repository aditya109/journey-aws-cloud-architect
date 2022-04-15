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

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/05-aws-fundamentals-elb-and-asg/assets/health-check.svg)

- Health checks are crucial for Load Balancers

- They enable the lb to know if the instances it forwards traffic to are available to reply to requests.

- The health check is done on a port and a route (/health is common)

- If the reponse is not 200 (OK), then the instance is unhealthy

#### Types of LBs

AWS has 4 kinds of managed Load Balancers:

| LB type              | Year of Launch | Supported Protocol                                | Remarks             |
| -------------------- | -------------- | ------------------------------------------------- | ------------------- |
| Classic LB - CLB     | 2009           | HTTP, HTTPS, TCP, SSL (secure TCP)                | v1 - old generation |
| Application LB - ALB | 2016           | HTTP, HTTPS, WebSocket                            | v2 - new generation |
| Network LB - NLB     | 2017           | TCP, TLS (secure TCP), UDP                        | v2 - new generation |
| Gateway LB - GWLB    | 2020           | Operates at layer 3 (Network layer) - IP Protocol | latest              |

> It is recommended to use the newer generation lbs as they provide more features.
> 
> Some lbs can be setup as internal (private) or external (public) ELBs.

#### LB Security Groups

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/05-aws-fundamentals-elb-and-asg/assets/lb-security-group.svg)

|                                               | LB Security Group                             | Application Security Group (allow traffic only from lb) |
| --------------------------------------------- | --------------------------------------------- | ------------------------------------------------------- |
| *type-protocol-port range-source-description* | HTTP-TCP-80-0.0.0.0/0-allow HTTP from an....  | HTTP-TCP-80-`sg-ID1`-allow HTTP from an....             |
| *type-protocol-port range-source-description* | HTTP-TCP-443-0.0.0.0/0-allow HTTP from an.... |                                                         |

### Sticky Sessions

### Cross Zone Load Balancing

### SSL Certificates

### Connection Draining

## CLB (v1)

- Supports TCP (layer 4), HTTP & HTTPS (layer 7)

- Health checks are TCP or HTTP based

- Fixed hostname: `XXX.region.elb.amazonAWS.com`

**Hands-On**

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/05-aws-fundamentals-elb-and-asg/assets/clb-hands-on.svg)

1. Launch an EC2 instance with a application running.

2. Go to `Load Balancers`. 

3. Create a load balancer > Click on `Classic Load Balancer`.

4. Enter details in `Define Load Balancer`.
   
   1. Within `Basic Configuration`, 
      
      - Select `Create an internal load balancer` if you want to make a private lb.
        
        > Next we have protocol:port rules.
        > 
        > | Key                    | Value   |
        > | ---------------------- | ------- |
        > | LB Protocol:Port       | HTTP:80 |
        > | Instance Protocol:Port | HTTP:80 |
   
   2. Assign a Security Group.
   
   3. Configure Security Settings.
   
   4. Configure Health Check. (*health check response timeout < health check interval*)
   
   5. Add EC2 Instances.
   
   6. Add Tags.
   
   7. Click to create.

> The instances moves from <span style="color:orange">OutOfService</span> -> <span style="color:orange">InService</span>.

Open the DNS name from the CLB info section.

> *What could be wrong if say my instances are still* <span style="color:orange">OutOfService</span> ?
> 
> Possibly check security groups if say the HTTP rules are not present in the Inbound rules, the LB would not be able to forward requests to the instances.
> 
> *Ok, but right now I can access my instance directly from outside of the internet as well from the LB, how should I tighten the security?*
> 
> You can change security groups Inbound rules, by delete the existing HTTP rule <span style="color:orange">Source</span>, and then create a new rule with the <span style="color:orange">Source</span> being the CLB security group.

## ALB (v2)

- ALB is Layer 7 (HTTP).

- It supports:
  
  - Load balancing to multiple HTTP apps across machines (target groups)
  
  - Load balancing to multiple apps on the same machine (ex. containers)

- It supports:
  
  - For HTTP/2 and Websocket
  
  - Redirects (from HTTP to HTTPS)
  
  - Routing table to different target groups:
    
    - Routing based on path in URL (example.com/users & example.com/posts)
    
    - Routing based on hostname in URL (one.example.com & other.example.com)
    
    - Routing based on Query strings, Headers (example.com/users?id=123&order=false)
  
  - ALB is a great fit for microservices & container-based application (example, Docker, Amazon ECS)
  
  - Supports a port mapping feature to redirect to a dynamic port in ECS.

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/05-aws-fundamentals-elb-and-asg/assets/alb-design.svg)

> <span style="color:cyan">Good to know:</span>
> 
> - On creating an ALB, we get a fixed hostname (XXX.region.elb.amazonaws.com).
> - The application servers don't see the IP of the client directly.
>   - The true IP of the client is inserted in the header **X-Forwarded-For**
>   - We can also get Port (**X-Forwarded-For**) and proto (X-Forwarded-Proto)

### Target Groups

- can be:
  
  - EC2 instances (can be managed by an Auto Scaling Group) - HTTP
  
  - ECS tasks (managed by ECS itself) - HTTP
  
  - Lambda functions - HTTP request is translated into a JSON event.

- Target groups must have IP addresses - must be private IPs.

- ALBs can route to multiple target groups.

- Health checks are at the target group level.

### Query strings/parameters routing

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/05-aws-fundamentals-elb-and-asg/assets/alb-query-based-routing.svg)







**Hands-On**



## NLB

**Hands-On**

## GWLB

**Hands-On**

### ASG Overview

**Hands-On**

### Scaling Policies
