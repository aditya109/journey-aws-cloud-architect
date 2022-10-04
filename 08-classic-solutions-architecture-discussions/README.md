# Classic Solutions Architecture Discussions (Case Studies)

Table of contents

## Pillars for a well architected application

- Costs
- Performance
- Reliability
- Security
- Operational excellence

## Stateless Web App: WhatIsTheTime.com

- This is a website which allows people to know what time it is.
- We don't need a database.
- We want to start small and can accept downtime.
- We want to fully scale vertically and horizontally, no downtime.

![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/08-classic-solutions-architecture-discussions/assets/simple-solution-whattimeisitdotcom.svg)
![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/08-classic-solutions-architecture-discussions/assets/scaling-vertically-whattimeisitdotcom.svg)
![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/08-classic-solutions-architecture-discussions/assets/scaling-horizontally-whattimeisitdotcom.svg)
![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/08-classic-solutions-architecture-discussions/assets/scaling-horizontally-2-whattimeisitdotcom.svg)
![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/08-classic-solutions-architecture-discussions/assets/scaling-horizontally-3-whattimeisitdotcom.svg)
![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/08-classic-solutions-architecture-discussions/assets/scaling-horizontally-asg-whattimeisitdotcom.svg)
![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/08-classic-solutions-architecture-discussions/assets/scaling-horizontally-disaster-resilience-whattimeisitdotcom.svg)

## Stateful Web App: MyClothes.com

- MyClothes.com allows people to buy clothes online.
- There's a shopping cart
- Our website is having hundreds of users at the same time.
- We need to scale, maintain horizontal scalability and keep our web application as stateless as possible.
- Users should not lose their shopping cart.
- Users should have their details (address, etc.) in a database.

![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/08-classic-solutions-architecture-discussions/assets/simple-solution-myclothesdotcom.svg)
![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/08-classic-solutions-architecture-discussions/assets/session-affinity-myclothesdotcom.svg)
![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/08-classic-solutions-architecture-discussions/assets/client-side-user-cookies-plain-myclothesdotcom.svg)
![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/08-classic-solutions-architecture-discussions/assets/user-cookies-plain-myclothesdotcom.svg)
![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/08-classic-solutions-architecture-discussions/assets/server-sessions-myclothesdotcom.svg)
![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/08-classic-solutions-architecture-discussions/assets/persistent-storage-myclothesdotcom.svg)
![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/08-classic-solutions-architecture-discussions/assets/persistent-storage-scaling-reads-with-replication-myclothesdotcom.svg)
![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/08-classic-solutions-architecture-discussions/assets/persistent-storage-scaling-reads-with-write-thorough-cache-myclothesdotcom.svg)
![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/08-classic-solutions-architecture-discussions/assets/security-and-disaster-recovery-myclothesdotcom.svg)

## MyWordPress.com: Stateful Web App

- We are trying to create a fully scalable WordPress website.
- We want that website to access and correctly display picture uploads.
- Our user data and the blog content should be stored in a MySQL database.

![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/08-classic-solutions-architecture-discussions/assets/simple-solution-mywordpressdotcom.svg)
![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/08-classic-solutions-architecture-discussions/assets/image-storage-mywordpressdotcom.svg)
![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/08-classic-solutions-architecture-discussions/assets/image-storage-efs-mywordpressdotcom.svg)

## Instantiating Applications quickly

When launching a full stack (EC2, EBS, RDS), it can take time to:

- Install applications
- Insert initial (or recovery) data
- Configure everything
- Launch the application

But, we can leverage AWS here, for example:

1. For EC2 instances:
   - **Use a Golden AMI**: Install your applications, OS dependencies, etc. beforehand and launch your EC2 instance from the Golden AMI.
   - **Bootstrap using User Data**: For dynamic configuration, use User Data scripts.
   - **Hybrid**: Mix Golden AMI and User Data (Elastic Beanstalk)
2. RDS Databases:
   - **Restore from a snapshot**: The database will have schemas and data ready !
3. EBS Volumes:
   - **Restore from a snapshot**: The disk will already be formatted and have data.

## Elastic Beanstalk

Common developer problems on AWS:

- Managing infrastructure
- Deploying code
- Configuring all the databases, load balancers, etc.
- Scaling concerns

Elastic Beanstalk is a developer centric view of deploying an application on AWS. 
It uses all the components which we have previously seen like EC2, ASG, ELB, RDS,..., but is a managed service:

- Automatically handles capacity provisioning, load balancing, scaling, application health monitoring, instance configuration, ...
- Just the application code is the responsibility of the developer.

Yet, the developer has full control over the configuration, Beanstalk is free but you pay for the underlying instances.

### Components

- **Application**: Collection of Elastic Beanstalk components (environments, versions, configurations, etc,....)
- **Application Version**: An iteration of your application code.
- **Environment**:
  - Collection of AWS resources running an application version (only one application version at a time)
  - **Tiers**: Web server environment tier & Worker environment tier.
  - You can create multiple environments (dev, test, prod, ....)

Steps to follow:

1. Create an application
2. Upload version
   - Deploy a new version = go to step 4
3. Launch environment
4. Manage environment
   - Update version = go to step 2

#### Web server tier vs worker tier

