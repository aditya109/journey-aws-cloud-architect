# Getting started with AWS

## AWS Cloud overview - regions & AZ

### AWS Regions

https://infrastructure.aws/

AWS has regions all around the world.

Names can be `us-east-1`, `eu-west-3`, etc.

A region is a **cluster of data centers**. Most AWS services are region-scoped

- AWS Availability Zones
- AWS Data centers
- AWS Edge Locations/ Points of presence

#### How to choose an AWS Region ?

*If you need to launch a new application, where should you do it ?*

**Ans.** Factors affecting the decision:

1. ***<span style="color:orange">Compilance</span>*** **with data governance and legal requirements**: data never leaves a region without explicit permission. 
1. ***<span style="color:orange">Proximity</span>*** **to customers**: reduced latency.
1. ***<span style="color:orange">Available services</span>*** **within a Region**: new services and new features aren't available in every region.
1. ***<span style="color:orange">Pricing</span>***: pricing varies region to region and is transparent in the service pricing page.

#### AWS Availability Zones:

- Each region has many availability zones (usually 3, min is 2, max is 6). Example: 
  - `ap-southeast-2a`
  - `ap-southeast-2b`
  - `ap-southeast-2c`
- Each availability zone (AZ) is one or more discrete data centers with redundant power, networking and connectivity.
- They are separate from each other, so that they're isolate from disasters.
- They are connected with high bandwidth, ultra-low latency networking.

#### AWS Points of Presence (Edge Locations)

- AWS has 216 Points of Presence (205 Edge Locations & 11 Regional Caches) in 84 cities across 42 countries.
- Content is delivered to end users with lower latency.

### Tour of the AWS Console

- AWS has Global Services:
  - Identity and Access Management (IAM)
  - Route 53 (DNS service)
  - CloudFront (Content Delivery Network)
  - WAF (Web Application Firewall)
  
- Most AWS services are region-scoped:
  - Amazon EC2 (Infrastructure as a Service)
  - Elastic Beanstalk (Platform as a Service)
  - Lambda (Function as a Service)
  - Rekognition (Software as a Service)
  
  
