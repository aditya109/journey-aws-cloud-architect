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
