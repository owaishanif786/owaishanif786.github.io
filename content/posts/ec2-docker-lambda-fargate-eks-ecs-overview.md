---
title: "Ec2 Docker Lambda Fargate Eks Ecs Overview"
date: 2022-08-05T08:13:22+05:00
draft: false
toc: true
featured_image: '/images/ec2-eks-ecs-fargate-lambda.png'
image: '/images/ec2-eks-ecs-fargate-lambda.png'
---

This is an overview from the Moon about basic compute resources in aws and what technologies can help us to manage them. 

### EC2: Elastic Compute
EC2 are the machines in the AWS cloud just like you create ubuntu,windows VMs in virtualbox or VM-ware. 
you can connect them via ssh or any RDP tools.
EC2 have hourly billing model based on resources like RAM, CPU and network traffic.

**why ec2:** just like in old days we build an app. ssh to instance, clone app and run the app. very easy to work with because its basic form of computing.

### Docker
Docker is use to build,run and deploy containers. you write an image and run it as container. you can run these container inside ec2 just like you can run them on your local machine. it take base image from any repository just like you take your code from github,bit bucket or code commit. 

**why docker:**
take all app code, libraries, dependencies, binaries etc and build as one image. Next simply run them as container.


### Lambda
Lambda is function as service by AWS. It have millisecond billing model where you are charged based on millisecond and the RAM you have allocated. Their max timeout is 15 minutes. if your scripts takes more than 15 minutes then consider containers like docker or VMs like EC2. you have limited disk storage inside lambda environment.

**why lambda:**
if a sync function takes only 3 minutes to run and it should only be running few times a day then why prepare full ec2 instance and pay for hourly. 

### Fargate
Its also compute platform just like lambda and ec2. lambda gets timeout but fargate does not because it have hourly billing model based on resources like vCPUs and RAM we allocate in units. we can deploy containers like docker on fargate. it scales automatically. Fargate can be managed by EKS pods or ECS tasks.

**why fargate:**
no need to patch,scale vms like you do with ec2. with ec2 you have to use AutoScaling stuff and 24/7 monitoring. but in fargate it happens automatically.

### EKS and ECS
Both are same in terms how they manage compute resources. like both can work with fargate or ec2. EKS uses kubernetes while ECS is using AWS technology to manage compute resources.

**why EKS or ECS:**
it handles scaling, balancing, managing containers clusters automatically. you just define a criteria.

### Summary
EC2, Lambda and Fargate all are compute resources just with different billing models and how we interact and manage them.
EKS and ECS can help us manage clusters of ec2 or fargate.  



