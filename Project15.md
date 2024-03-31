AWS Cloud Solution For 2 Company Websites Using A Reverse Proxy Technology 
============================================

In this project, we will build a secure infrastructure inside AWS VPC (Virtual Private Cloud) network for a fictitious company that uses WordPress CMS for its main business website, and a Tooling Website **(`https://github.com/QuadriBello/tooling`)** for their DevOps team. As part of the company’s desire for improved security and performance, a decision has been made to use a reverse proxy technology from NGINX to achieve this.

Cost, Security, and Scalability are the major requirements for this project. Hence, implementing the architecture designed below, ensures that infrastructure for both websites, WordPress and Tooling, is resilient to the Web Server’s failures, can accomodate to increased traffic and, at the same time, has reasonable cost.

![152831445-844e3865-0317-4bf4-969a-490a7c1e06ba](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/a07fce80-4415-4e3c-961e-ff5e1af744cf)

## Introduction to AWS VPC

An Amazon Virtual Private Cloud (VPC) is like your own private section of the Amazon cloud, where you can place and manage your resources (like servers or databases). You control who and what can go in and out, just like a gated community.
An AWS Cloud setup usually comes with a default VPC. The Default VPC is like a starter pack provided by Amazon for your cloud resources. It's a pre-configured space in the Amazon cloud where you can immediately start deploying your applications or services. It has built-in security and network settings to help you get up and running quickly, but you can adjust these as you see fit.

![VPC](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/0d8d5f0b-d623-4461-a24c-b5bfb7c32366)

A Default VPC, which Amazon provides for you in each region (think of a region as a separate city), is like a pre-built house in that city. This house comes with some default settings to help you move in and start living (or start deploying your applications) immediately. But just like a real house, you can change these settings according to your needs. A VPC can also have smaller sections called subnets.

Subnets are like smaller segments within a VPC that help you organize and manage your resources. Subnets are like dividing an office building into smaller sections, where each section represents a department. In this analogy, subnets are created to organize and manage the network effectively.

![subnet](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e95a3a4b-9b4d-4ce0-b264-80acd061d715)

## Project Prerequisites

There are few requirements that must be met before we begin our project implementation:

1. We properly configure our AWS account and Organization Unit [by following the steps here](https://youtu.be/9PQYCc_20-Q)

    * Create an [AWS Master account](https://aws.amazon.com/free/). (*Also known as Root Account*)
    * Within the Root account, create a sub-account and name it **DevOps**. (We will need another email address to complete this) 
    * Within the Root account, create an **AWS Organization Unit (OU)**. Name it **Dev**. (We will launch Dev resources in there)
    * Move the **DevOps** account into the ***Dev*** **OU**.
    * Login to the newly created AWS account using the new email address.

2. We create a free domain name for your fictitious company at Freenom domain registrar [here](https://www.freenom.com).

3. We create a hosted zone in AWS, and map it to your free domain from Freenom. [Watch how to do that here](https://youtu.be/IjcHp94Hq8A) 

***NOTE*** : As we proceed with configuration, we ensure that all resources are appropriately tagged, for example:

* Project: `<Give our project a name>`
* Environment: `<dev>`
* Automated: `<No>` (If we create a recource using an automation tool, it would be `<Yes>`)

#### Setting Up Infrastructure

We need to always make reference to the architectural diagram and ensure that our configuration and infrastructure is aligned with it.

1. Create a [VPC](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
2. Create subnets as shown in the architecture
3. Create a route table and associate it with public subnets
4. Create a route table and associate it with private subnets
5. Create an [Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)
6. Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accisble from the Internet)
6. Create 3 [Elastic IPs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html) 
7. Create a [Nat Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) and assign one of the Elastic IPs (*The other 2 will be used by [Bastion hosts](https://aws.amazon.com/quickstart/architecture/linux-bastion/))
8. Create a [Security Group](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html#CreatingSecurityGroups) for:
