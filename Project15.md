AWS Cloud Solution For 2 Company Websites Using A Reverse Proxy Technology 
============================================

In this project, we will build a secure infrastructure inside AWS VPC (Virtual Private Cloud) network for a fictitious company that uses WordPress CMS for its main business website, and a Tooling Website **(`https://github.com/QuadriBello/tooling`)** for their DevOps team. As part of the company’s desire for improved security and performance, a decision has been made to use a reverse proxy technology from NGINX to achieve this.

Cost, Security, and Scalability are the major requirements for this project. Hence, implementing the architecture designed below, ensures that infrastructure for both websites, WordPress and Tooling, is resilient to the Web Server’s failures, can accomodate to increased traffic and, at the same time, has reasonable cost.

![152831445-844e3865-0317-4bf4-969a-490a7c1e06ba](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/a07fce80-4415-4e3c-961e-ff5e1af744cf)

#### Project Prerequisites

There are few requirements that must be met before we begin our project implementation:

1. We properly configure our AWS account and Organization Unit [by following the steps here](https://youtu.be/9PQYCc_20-Q)

    * Create an [AWS Master account](https://aws.amazon.com/free/). (*Also known as Root Account*)
    * Within the Root account, create a sub-account and name it **DevOps**. (We will need another email address to complete this) 
    * Within the Root account, create an **AWS Organization Unit (OU)**. Name it **Dev**. (We will launch Dev resources in there)
    * Move the **DevOps** account into the ***Dev*** **OU**.
    * Login to the newly created AWS account using the new email address.

2. We create a free domain name for your fictitious company at Freenom domain registrar [here](https://www.freenom.com).

3. We create a hosted zone in AWS, and map it to your free domain from Freenom. [Watch how to do that here](https://youtu.be/IjcHp94Hq8A) 

***NOTE*** : As you proceed with configuration, ensure that all resources are appropriately tagged, for example:

* Project: `<Give your project a name>`
* Environment: `<dev>`
* Automated: `<No>` (If you create a recource using an automation tool, it would be `<Yes>`)
