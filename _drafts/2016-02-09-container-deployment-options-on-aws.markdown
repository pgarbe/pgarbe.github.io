---
layout: post
title: "Docker Deployment Options on AWS: EC2-Bootstrap"
date: 2016-02-09 13:15:29 +0100
author: Philipp Garbe
comments: true
published: false
# categories: [Docker, AWS, Continuous Delivery]
keywords: "Docker, AWS ECS, EC2 Container Service, CloudFormation, Docker Compose"
description: "An overview of different deployment options for Docker containers on AWS"
---

Is Docker ready for production? Many would argue so. It's easy to start applications locally without installing any dependencies. Even running a local cluster is not a big thing anymore. But what options do I have when it comes to deployment? Especially, deployment to Amazon Web Services (AWS)?


https://github.com/ElizabethGagne/microservices-playground
https://clusterhq.com/2016/02/08/flocker-swarm-in-30-minutes-aws/
https://docs.clusterhq.com/en/latest/docker-integration/cloudformation.html

## The Setup
In order to compare the different options I need a sample application. What can be better than the famous [Voting-App](https://github.com/docker/example-voting-app) of Docker itself?

But which criteria are used to compare the different approaches?

##### Cycle Time
The cycle time is the time it takes to release a code commit to production. With a strong background in Continuous Deployment, for me this is a very important criteria. The faster the better. I don't want to wait too long to get feedback for my commits.

##### Configuration as Code
Automation is King. It's good when the deployment can be done by (mutliple) API calls, but it's even better to have a declarative configuration file which is then used to apply the changes.

##### Scaling
A very prominent feature on AWS is [AutoScaling](LINK) for EC2 instances. Can ASG be used for Containers? If not, what is needed for Docker Containers to scale up and down? Can it be automated?

##### Continuous Deployment
In order to support multiple releases I have to trust my Continuous Deployment pipeline to release my application without interruptions for the users. How easy is it to setup [blue/green deployment](LINK) or support [canary releases](LINK)?

As everyone has different needs, I don't use grades to compare the result, but I'll summarize the advantages and disadvantages for each of these categories.

## Part 1: EC2-Bootstrap

Let's start with the first approach: __EC2 Bootstrap__

> Definition: Run each Docker container on it's own EC2 instance.

???PIC???

### Show me the code
You can have a look to the code in my [GitHub repository](https://github.com/pgarbe/docker-cloud-deployment/blob/master/ec2-bootstrap). Here I'd like to mention only the highlights.

TODO: UserData with Version
TODO: ASG (Health-Check)
TODO: Blue/Green


### Results

##### Cycle Time
There are two approaches with different advantages and disadvantages.

__Immutable EC2 instances__   
Immutable EC2 instances means that whenever you have a new build artifact you start a new machines and replace the existing machines.

\+ Simple setup of your configuration because you have to take care only about the startup of your machine. Updates happens only by starting a new instance.  
\+ Amazon's [AutoScaling Group (ASG)](LINK) and [Elastic LoadBalancer (ELB)](LINK) are a good match and very mature. There are many options to control the way how many and how fast new machines are rolled out.

\- It takes a couple of minutes to start a new EC2 instance and add it to the ELB. When you run several instances it takes even longer to replace all of them.


__Mutable EC2 instances__   
Mutable EC2 instances means you replace the EC2 instances only when the configuration has been changed. When only your build artifact has been changed you do an [update](LINK) on the machine.
(Changes on host -> immutable, changes in docker container -> mutable)

\+ Complex configuration (start and update)  
\+ Machine needs to be removed manually(?) from ELB?

\- Faster (re-load jar)


##### Configuration as Code
\+ This solution can be fully automated. The tools on AWS are mature and stable.  
\+ Configuration as code is completely supported. The CloudFormation template file (JSON) can be part of the repository.  

\- Local testing of the configuration is only partially possible. CloudFormation supports some basic syntax validations but logical errors can only be found when the CloudFormation template is deployed.  
\- CloudFormation's configuration file (JSON) can become very complex when it grows. There are already some tools which promises to make it easier (LINK).

##### Scaling
+ This solution supports the full power of the [AutoScaling groups](LINK). Each Container is scaled by it's own ASG. That allows you to define container-specific criteria for up- and down-scaling.

##### Continuous Deployment
\+ AWS supports rolling updates (needs immutable servers)

\- No built-in support for Blue/Green delivery
\- Canary releases have to be done manually.


### Conclusion
This approach can be a good starting point when you want to start with Docker and you're already used to work with EC2. You can get in touch with Docker and keep the experience you already have with EC2.

On the downside it doesn't help you when you want to have faster deployments. It can make the deployments even slower because Docker Images are normally bigger than the raw application (e.g. JAR-balls).

The next blog post will show how [Docker Swarm]() can be used to deploy the Voting-App.
