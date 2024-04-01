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

![AWS organisations account](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/f6845afd-f717-46aa-8932-38ffb8c0d153)

2. We create a free domain name for your fictitious company at Freenom domain registrar [here](https://www.freenom.com).

3. We create a hosted zone in AWS, and map it to your free domain from Freenom. [Watch how to do that here](https://youtu.be/IjcHp94Hq8A) 

***NOTE*** : As we proceed with configuration, we ensure that all resources are appropriately tagged, for example:

* Project: `<Give our project a name>`
* Environment: `<dev>`
* Automated: `<No>` (If we create a recource using an automation tool, it would be `<Yes>`)

#### Setting Up Infrastructure

We need to always make reference to the architectural diagram and ensure that our configuration and infrastructure is aligned with it.

1. Create a [VPC:](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html) 

* We navigate to the VPC dashboard in AWS and the we click on **`Your VPCs > Create VPC`**. Next we enter the Name tag and the IPv4 CIDR and then we click on the "Create VPC" button.

![create VPC](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e909e16a-101f-4ac1-8696-ffe6e69fdea6)

![VPC created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c51ca9a2-41e3-401c-a718-cbf73666193b)

* AS we can see in the image above, our VPC was successfully created. However, DNS hostnames is disabled. To enable this, we click on **`Actions > Edit VPC Settings`**, then we select the check box to Enable DNS hostnames and we click on Save.

![enable DNS hostnames](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/14831105-f414-47f8-b572-c7587a1f1423)

![dns hostname enabled](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/af236a80-35bd-4a2b-9cb2-db51779e2512)
 
2. Create subnets as shown in the architecture:

 * From the VPC dashboard we navigate to **`Subnets > Create subnet`**, then we select our VPC in the VPC dropdown box and as shown in the image below, we enter the details for subnet settings to create all our subnets.

![subnet settings](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/91a823d8-b5a2-4da9-9ffb-10dfccd9d4fc)

![subnets created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/f1a5028d-39d3-44af-ba36-901591be7423)

3. Create a route table and associate it with public subnets:

 * From the VPC dashboard we navigate to **`Route tables > Create route tables`**, then we enter the route table name and we select our VPC in the VPC dropdown box and click on "Create route table".

 ![create public route table](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/45287965-74c5-4b99-808a-bc6621a771a0)

 * Next we associate the route table with our public subnets by selecting the route table and navigating to **`Subnet associations > Edit subnet associations`**.and then we select our public subnets and we click on "Save associations".

![associate public subnets](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/58472e11-c8bc-41a4-a117-5023204eb6f7)

4. Create a route table and associate it with private subnets:

 * From the VPC dashboard we navigate to **`Route tables > Create route tables`**, then we enter the route table name and we select our VPC in the VPC dropdown box and click on "Create route table".

 ![create private route table](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/b67f6412-9ad1-4358-b51a-b1faa1ff9ef8)

* Next we associate the route table with our private subnets by selecting the route table and navigating to **`Subnet associations > Edit subnet associations`**.and then we select our private subnets and we click on "Save associations".

![associate private subnets](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c1e73208-f253-42c8-b09a-1b089649edcd)

5. Create an [Internet Gateway:](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)

* From the VPC dashboard we navigate to  **`Internet gateways > Create internet gateway`**, then we enter the name tag and click on "Create internet gateway".

  ![create internet gateway](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/1ad68088-d9a7-47b1-8cab-c575e41983a7)

* Next, we click on "Attach to a VPC" then in the resulting page, we select our VPC in the dropdown box and we click on "Attach internet gateway". 

![attach to a vpc](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/bef044c6-ee0b-49d8-bf80-1759bfb932a1)

![attach to a vpc 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/3d89a686-731b-4f64-9c4d-2031a71e110a)

6. Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accisble from the Internet)

* We select the public route table and we navigate to **`Actions > Edit routes`** then we click on "Add route" and then we edit and save changes as shown in the image below:

![edit route associate internet gateway](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/6769250d-a74c-4546-b855-c87f1e6899df)

7. Create 3 [Elastic IPs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)

* From the VPC dashboard we navigate to  **`Elastic IPs > Allocate Elastic IP address`**, then we give a name tag and we click on "Allocate".

![allocate elastic IP](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/4b8f61f4-68ca-4f49-8cdc-06a4bd5b4c14)

8. Create a [Nat Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) and assign one of the Elastic IPs (*The other 2 will be used by [Bastion hosts](https://aws.amazon.com/quickstart/architecture/linux-bastion/))
  
* From the VPC dashboard we navigate to  **`NAT gateways > Create NAT gateway`**, then we proceed to enter the NAT gateway settins as shown in the image below, and afterwards, we click on "Create NAT gateway".

 ![create NAT gateway](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/d8645cbf-cc0d-43e9-b715-2d1b4d72673a)

* Next we proceed to add the NAT gateway to our private route table.

![add NAT gateway to private route table](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/75478b8c-1100-4665-b895-844091a31425)

9. Create [Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html#CreatingSecurityGroups)

From the VPC dashboard we navigate to  **`Security groups > Create security group`**, then we proceed to create security groups in accordance with the following:

**i.** `Application Load Balancer`: Our external ALB will be available from the Internet.

![security group ALB nginx servers](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/03562318-36a3-4512-b804-c7821045c21b)

**ii.** `Bastion Servers`: Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation public IP address. To get this information, simply go to your terminal and type `curl www.canhazip.com`

![security group for bastion](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/459a6079-26b1-4d3f-93c9-92bc1c800b92)

**iii.** `Nginx Servers`: Traffic access to Nginx reverse proxy servers should only be allowed from a [Application Load balancer (ALB)](https://aws.amazon.com/elasticloadbalancing/application-load-balancer/). However, we also allow SSH access from the bastion server.

![security group for nginx reverse proxy](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/0177df9f-a9c5-4151-9c08-e6bbfa9a96fa)

**iv.** `Internal Application Load Balancer`: Our Internal ALB will only allow traffic from the Nginx reverse proxy servers.

![security group for internal application load balancer](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/8244f3b0-7507-44d0-b41f-9cdb73eb290d)

**v.** `Webservers`: Access to Webservers should only be allowed from the internal ALB servers. We also allow SSH access to the webservers from the bastion.

![security group for webservers](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/611d32b7-a401-4824-8a77-2c33eac6f951)

**vi.** `Data Layer`: Access to the Data layer, which is comprised of [Amazon Relational Database Service (RDS)](https://aws.amazon.com/rds/) and [Amazon Elastic File System (EFS)](https://aws.amazon.com/efs/) must be carefully desinged - only bastion and  webservers should be able to connect to **RDS**, while Nginx and Webservers will have access to **EFS Mountpoint**.

![security group for data layer](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/189160b7-dc96-400a-9104-ebdbb39f4d58)

#### Creating With Compute Resources 

Next, we need to set up and configure compute resources inside our VPC. The recources related to compute are: 

* [TLS Certificates](https://en.wikipedia.org/wiki/Transport_Layer_Security)
* [EC2 Instances](https://www.amazonaws.cn/en/ec2/instance-types/)
* [Launch Templates](https://docs.aws.amazon.com/autoscaling/ec2/userguide/LaunchTemplates.html)
* [Target Groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html)
* [Autoscaling Groups](https://docs.aws.amazon.com/autoscaling/ec2/userguide/AutoScalingGroup.html)
* [Application Load Balancers (ALB)](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)

#### TLS Certificates From [Amazon Certificate Manager (ACM)](https://aws.amazon.com/certificate-manager/)

We will need TLS certificates to handle secured connectivity to our Application Load Balancers (ALB).

**i.** Navigate to AWS ACM
**ii.** Request a public wildcard certificate for the domain name you registered in Freenom
**iii.** Use DNS to validate the domain name
**iv.** Tag the resource
