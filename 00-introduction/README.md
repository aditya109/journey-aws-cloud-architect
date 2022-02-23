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

