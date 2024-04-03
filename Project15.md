AWS Cloud Solution For 2 Company Websites Using A Reverse Proxy Technology 
============================================

In this project, we will build a secure infrastructure inside AWS VPC (Virtual Private Cloud) network for a fictitious company that uses WordPress CMS for its main business website, and a Tooling Website **(`https://github.com/QuadriBello/tooling`)** for their DevOps team. As part of the company’s desire for improved security and performance, a decision has been made to use a reverse proxy technology from NGINX to achieve this.

Cost, Security, and Scalability are the major requirements for this project. Hence, implementing the architecture designed below, ensures that infrastructure for both websites, WordPress and Tooling, is resilient to the Web Server’s failures, can accomodate to increased traffic and, at the same time, has reasonable cost.

![152831445-844e3865-0317-4bf4-969a-490a7c1e06ba](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/a07fce80-4415-4e3c-961e-ff5e1af744cf)

### Introduction to AWS VPC

An Amazon Virtual Private Cloud (VPC) is like your own private section of the Amazon cloud, where you can place and manage your resources (like servers or databases). You control who and what can go in and out, just like a gated community.
An AWS Cloud setup usually comes with a default VPC. The Default VPC is like a starter pack provided by Amazon for your cloud resources. It's a pre-configured space in the Amazon cloud where you can immediately start deploying your applications or services. It has built-in security and network settings to help you get up and running quickly, but you can adjust these as you see fit.

![VPC](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/0d8d5f0b-d623-4461-a24c-b5bfb7c32366)

A Default VPC, which Amazon provides for you in each region (think of a region as a separate city), is like a pre-built house in that city. This house comes with some default settings to help you move in and start living (or start deploying your applications) immediately. But just like a real house, you can change these settings according to your needs. A VPC can also have smaller sections called subnets.

Subnets are like smaller segments within a VPC that help you organize and manage your resources. Subnets are like dividing an office building into smaller sections, where each section represents a department. In this analogy, subnets are created to organize and manage the network effectively.

![subnet](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e95a3a4b-9b4d-4ce0-b264-80acd061d715)

### Project Prerequisites

There are few requirements that must be met before we begin our project implementation:

1. We properly configure our AWS account and Organization Unit [by following the steps here](https://youtu.be/9PQYCc_20-Q)

* Create an [AWS Master account](https://aws.amazon.com/free/). (*Also known as Root Account*)
* Within the Root account, create a sub-account and name it **DevOps**. (We will need another email address to complete this) 
* Within the Root account, create an **AWS Organization Unit (OU)**. Name it **Dev**. (We will launch Dev resources in there)
* Move the **DevOps** account into the ***Dev*** **OU**.
* Login to the newly created AWS account using the new email address.

![AWS organisations account](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/f6845afd-f717-46aa-8932-38ffb8c0d153)

2. We create a free domain name for our fictitious company at Freenom domain registrar [here](https://www.freenom.com).

3. We create a hosted zone in AWS, and map it to our free domain from Freenom. [Watch how to do that here](https://youtu.be/IjcHp94Hq8A) 

***NOTE*** : As we proceed with configuration, we ensure that all resources are appropriately tagged, for example:

* Project: `<Give our project a name>`
* Environment: `<dev>`
* Automated: `<No>` (If we create a recource using an automation tool, it would be `<Yes>`)

#### Setting Up Infrastructure

We need to always make reference to the architectural diagram and ensure that our configuration and infrastructure is aligned with it.

1. Create a [VPC:](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html) 

**i.** We navigate to the VPC dashboard in AWS and the we click on **`Your VPCs > Create VPC`**. Next we enter the Name tag and the IPv4 CIDR and then we click on the "Create VPC" button.

![create VPC](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e909e16a-101f-4ac1-8696-ffe6e69fdea6)

![VPC created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c51ca9a2-41e3-401c-a718-cbf73666193b)

**ii.** As we can see in the image above, our VPC was successfully created. However, DNS hostnames is disabled. To enable this, we click on **`Actions > Edit VPC Settings`**, then we select the check box to Enable DNS hostnames and we click on Save.

![enable DNS hostnames](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/14831105-f414-47f8-b572-c7587a1f1423)

![dns hostname enabled](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/af236a80-35bd-4a2b-9cb2-db51779e2512)
 
2. Create subnets as shown in the architecture:

 **i.** From the VPC dashboard we navigate to **`Subnets > Create subnet`**, then we select our VPC in the VPC dropdown box and as shown in the image below, we enter the details for subnet settings to create all our subnets.

![subnet settings](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/91a823d8-b5a2-4da9-9ffb-10dfccd9d4fc)

![subnets created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/f1a5028d-39d3-44af-ba36-901591be7423)

3. Create a route table and associate it with public subnets:

 **i.** From the VPC dashboard we navigate to **`Route tables > Create route tables`**, then we enter the route table name and we select our VPC in the VPC dropdown box and click on "Create route table".

 ![create public route table](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/45287965-74c5-4b99-808a-bc6621a771a0)

 **ii.** Next we associate the route table with our public subnets by selecting the route table and navigating to **`Subnet associations > Edit subnet associations`**.and then we select our public subnets and we click on "Save associations".

![associate public subnets](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/58472e11-c8bc-41a4-a117-5023204eb6f7)

4. Create a route table and associate it with private subnets:

 **i.** From the VPC dashboard we navigate to **`Route tables > Create route tables`**, then we enter the route table name and we select our VPC in the VPC dropdown box and click on "Create route table".

 ![create private route table](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/b67f6412-9ad1-4358-b51a-b1faa1ff9ef8)

**ii.** Next we associate the route table with our private subnets by selecting the route table and navigating to **`Subnet associations > Edit subnet associations`**.and then we select our private subnets and we click on "Save associations".

![associate private subnets](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c1e73208-f253-42c8-b09a-1b089649edcd)

5. Create an [Internet Gateway:](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)

**i.** From the VPC dashboard we navigate to  **`Internet gateways > Create internet gateway`**, then we enter the name tag and click on "Create internet gateway".

  ![create internet gateway](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/1ad68088-d9a7-47b1-8cab-c575e41983a7)

**ii.** Next, we click on "Attach to a VPC" then in the resulting page, we select our VPC in the dropdown box and we click on "Attach internet gateway". 

![attach to a vpc](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/bef044c6-ee0b-49d8-bf80-1759bfb932a1)

![attach to a vpc 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/3d89a686-731b-4f64-9c4d-2031a71e110a)

6. Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accessible from the Internet)

**i.** We select the public route table and we navigate to **`Actions > Edit routes`** then we click on "Add route" and then we edit and save changes as shown in the image below:

![edit route associate internet gateway](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/6769250d-a74c-4546-b855-c87f1e6899df)

7. Create 3 [Elastic IPs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)

**i.** From the VPC dashboard we navigate to  **`Elastic IPs > Allocate Elastic IP address`**, then we give a name tag and we click on "Allocate".

![allocate elastic IP](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/4b8f61f4-68ca-4f49-8cdc-06a4bd5b4c14)

8. Create a [Nat Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) and assign one of the Elastic IPs (*The other 2 will be used by [Bastion hosts](https://aws.amazon.com/quickstart/architecture/linux-bastion/))
  
**i.** From the VPC dashboard we navigate to  **`NAT gateways > Create NAT gateway`**, then we proceed to enter the NAT gateway settins as shown in the image below, and afterwards, we click on "Create NAT gateway".

 ![create NAT gateway](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/d8645cbf-cc0d-43e9-b715-2d1b4d72673a)

**ii.** Next we proceed to add the NAT gateway to our private route table.

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

#### Creating Compute Resources 

Next, we need to set up and configure compute resources inside our VPC. The resources related to compute are: 
1. [TLS Certificates](https://en.wikipedia.org/wiki/Transport_Layer_Security)
   
2. [EC2 Instances](https://www.amazonaws.cn/en/ec2/instance-types/)
 
3. [Launch Templates](https://docs.aws.amazon.com/autoscaling/ec2/userguide/LaunchTemplates.html)
 
4. [Target Groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html)

5. [Autoscaling Groups](https://docs.aws.amazon.com/autoscaling/ec2/userguide/AutoScalingGroup.html)

6. [Application Load Balancers (ALB)](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)

#### TLS Certificates From [Amazon Certificate Manager (ACM)](https://aws.amazon.com/certificate-manager/)

We will need TLS certificates to handle secured connectivity to our Application Load Balancers (ALB). To obtain the certificates, we proceed with the following steps:

**i.** Navigate to AWS Certificate Manager

**ii.** Request a public wildcard certificate for the domain name you registered.

**iii.** Use DNS to validate the domain name

![AWS cert](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/7699f683-c653-4829-97ac-01f8f64efbfd)

**iv.** Tag the resource and Click on Request.

**v.** Create DNS CNAME records in Amazon Route 53. 

![DNS CNAME record route53](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/5957bed9-8aea-404d-95a4-e65e34c45bc8)

#### Setup Amazon EFS

[Amazon Elastic File System (Amazon EFS)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEFS.html) provides a simple, scalable, fully managed elastic [Network File System (NFS)](https://en.wikipedia.org/wiki/Network_File_System) for use with AWS Cloud services and on-premises resources. In this project, we will utulize EFS service and mount filesystems on both Nginx and Webservers to store data.

**i.** Create an EFS filesystem: From the EFS dashboard we click on "Create file system". Then we enter the filesystem name and select the VPC, then we click on "customize".

![create file system](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/ce4a13c2-504d-4544-8dae-db8441ff4c3b)

**ii.** In the following page, we add a Name tag and we click on the "Next" button.

![name tag](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/ab3805db-2b2e-41b6-b5b0-114b27fd8aac)

**iii.** In the Network Access page, we need to specify our EFS Mount targets per Availability Zone in the VPC and associate it with our private subnets 1 and 2 according to our project diagram.

![network access](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/a64a8734-7c00-43a4-8aea-c9300564b8d9)

**iv.** Associate the Security groups created earlier for data layer.

**v.** And in the next page, we review our configuration and we click on "Create".

![review and create](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/cfdf9ea4-45ff-41a0-b300-21e407e0e8e5)

**vi.** Next we create two access points for our Wordpress and Tooling web servers respectively. We navigate to **`ACS-filesystem > Access points > Create access point`** and then we configure the access points as shown in the images below:

![wordpress access point](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/dba81c61-9de6-4773-9ebc-c041a7596bf2)

![tooling access point](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/03c62eaa-9bbe-4952-a647-5068f7fd8e67)

#### Setup Amazon RDS 

***Pre-requisite***: Create a KMS key from [Key Management Service (KMS)](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html) to be used to encrypt the database instance.

* From the AWS Key Management Service dashboard, we click on "Create a Key" and then we configure key as shown in the images below:

![configure KMS key](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/a6456c21-f31f-49b3-8dfc-c0b194b02aa0)

![configure KMS key2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/20482dad-12a6-402a-ac0c-ed9880c74ae5)

![configure KMS key3](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/7a5d528a-fd50-4763-938b-ce7494da26a3)

[Amazon Relational Database Service (Amazon RDS)](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html) is a managed distributed relational database service by Amazon Web Services. This web service running in the cloud designed to simplify setup, operations, maintenans & scaling of relational databases. *Without RDS, Database Administrators (DBA) have more work to do, due to RDS, some DBAs have become jobless*

To ensure that our databases are highly available and also have failover support in case one availability zone fails, we will configure a [multi-AZ](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html) set up of [RDS MySQL database](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_MySQL.html) instance. In our case, since we are only using 2 AZs, we can only failover to one, but the same concept applies to 3 Availability Zones. We will not consider possible failure of the whole Region, but for this AWS also has a solution - this is a more advanced concept that will be discussed in following projects.

To configure RDS, we follow steps below:

**i.** Create a subnet group and add 2 private subnets (data Layer). From the Amazon RDS dashboard, we navigate to **`Subnet groups > Create DB subnet group`** and we configure as shown in the image below:

![create subnet group](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/315396ca-0f60-42f6-be9b-f85e0823377e)

**ii.** Create an RDS Instance for `mysql 8.*.*`. We navigate to **`Dashboard > Create database`**, then we configure and create our database as shown in the images below:

![RDS config 1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/77769ba8-e532-4824-ad6e-2e7e28d52fb6)

**iii.** To satisfy our architectural diagram, we will need to select either **Dev/Test** or **Production** Sample Template. But to minimize AWS cost, we can select the free tier sample template then we configure other settings accordingly (*For test purposes, most of the default settings are good to go*). In the real world, we will need to size the database appropriately. We will need to get some information about the usage. If it is a highly transactional database that grows at 10GB weekly, you must bear that in mind while configuring the initial storage allocation, storage autoscaling, and maximum storage threshold.

![RDS config 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/eb6b0317-7eb4-46c2-bdad-41a0bafc425f)

**iv.** Next, we configure VPC and security (ensure the database is not available from the Internet), we configure backups and retention and we encrypt the database using the KMS key created earlier.

![RDS config 3](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/bcac1b17-6f14-4b54-ab89-d894004e130c)
 
**v.** Enable [CloudWatch](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/monitoring-cloudwatch.html) monitoring and export `Error` and `Slow Query` logs (for production, also include `Audit`)


#### Proceed With Compute Resources 

We will need to set up and configure compute resources inside our VPC. The resources related to compute are: 

1. [EC2 Instances](https://www.amazonaws.cn/en/ec2/instance-types/)
  
2. [Launch Templates](https://docs.aws.amazon.com/autoscaling/ec2/userguide/LaunchTemplates.html)
   
3. [Target Groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html)
 
4. [Autoscaling Groups](https://docs.aws.amazon.com/autoscaling/ec2/userguide/AutoScalingGroup.html)

5. [TLS Certificates](https://en.wikipedia.org/wiki/Transport_Layer_Security)

6. [Application Load Balancers (ALB)](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) 

#### Set Up Compute Resources for Nginx

##### Provision EC2 Instances for Nginx

**i.** We create an EC2 Instance based on CentOS [Amazon Machine Image (AMI)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) in any 2 [Availability Zones (AZ) in any AWS Region](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html) ([it is recommended to use the Region that is closest to our customers](https://docs.aws.amazon.com/AmazonElastiCache/latest/mem-ug/RegionsAndAZs.html)). We use EC2 instance of [T2 family](https://aws.amazon.com/ec2/instance-types/t2/) (e.g. t2.micro or similar)

![launch ec2 instances](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/2d0f17d8-3f95-4244-966a-496894c95309)
   
**ii.** Then we ssh into the instance and we ensure that it has the following software installed:
        
    * `python`
    * `ntp`
    * `net-tools`
    * `vim`
    * `wget`
    * `telnet`
    * `epel-release`
    * `htop`

4. [Create an `AMI`](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/tkv-create-ami-from-instance.html) out of the EC2 instance

* Select Nginx instance, then navigate to **`Actions > Image and templates > Create image`** and then we configure and create the AMI as shown in the image below:

###### Prepare Launch Template For Nginx (One Per Subnet)

1. Make use of the AMI to set up a launch template
2. Ensure the Instances are launched into a public subnet 
3. Assign appropriate security group
4. Configure [Userdata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) to update `yum` package repository and install `nginx`

###### Configure Target Groups

* From the EC2 dashboard, we navigate to **`Target groups > Create target group`** and then we do the following:

1. Select Instances as the target type
2. Ensure the protocol `HTTPS` on secure TLS port `443`
3. Ensure that the health check path is `/healthstatus`
4. Register `Nginx` Instances as targets
5. Ensure that health check passes for the target group

###### Configure Autoscaling For Nginx 

* From the EC2 dashboard, we navigate to **`Auto Scaling Groups > Create Auto Scaling group`**

1. Select the right launch template
2. Select the VPC
3. Select both public subnets
4. Enable Application Load Balancer for the AutoScalingGroup (ASG) 
5. Select the target group you created before
6. Ensure that you have health checks for both `EC2` and `ALB`
7. The desired capacity is 2
8. Minimum capacity is 2 
9. Maximum capacity is 4
10. Set [scale out](https://docs.aws.amazon.com/autoscaling/ec2/userguide/AutoScalingGroupLifecycle.html) if CPU utilization reaches 90%
11. Ensure there is an [`SNS`](https://docs.aws.amazon.com/sns/latest/dg/welcome.html) topic to send scaling notifications

#### Set Up Compute Resources for Bastion

###### Provision the EC2 Instances for Bastion

**i.** We create an EC2 Instance based on CentOS Amazon Machine Image (AMI) per each Availability Zone in the same Region and the same AZ where we created Nginx server.

![launch ec2 instances](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/2d0f17d8-3f95-4244-966a-496894c95309)
   
**ii.** Then we ssh into the instance and we ensure that it has the following software installed:

    * `python`
    * `ntp`
    * `net-tools`
    * `vim`
    * `wget`
    * `telnet`
    * `epel-release`
    * `htop`

![bastion ami installation 1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/0e4800d3-82ec-48ae-9e9f-725c833e1016)

![bastion ami installation 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/64b24662-5094-497d-a6e7-e563fc1344d4)
    
4. [Associate an Elastic IP](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html#using-instance-addressing-eips-associating) with each of the Bastion EC2 Instances
5. Create an `AMI` out of the EC2 instance

* We select the Bastion instance, then navigate to **`Actions > Image and templates > Create image`** and then we configure and create the AMI as shown in the image below:

###### Prepare Launch Template For Bastion (One per subnet)

* From the EC2 dashboard, we navigate to **`Launch Templates > Create launch template`** and then we do the following:

1. Make use of the AMI to set up a launch template
2. Ensure the Instances are launched into a public subnet 
3. Assign appropriate security group
4. Configure Userdata to update `yum` package repository and install `Ansible` and `git`

###### Configure Target Groups

1. Select Instances as the target type
2. Ensure the protocol is `TCP` on port `22`
3. Register `Bastion` Instances as targets
4. Ensure that health check passes for the target group

###### Configure Autoscaling For Bastion 

* From the EC2 dashboard, we navigate to **`Auto Scaling Groups > Create Auto Scaling group`**
  
1. Select the right launch template
2. Select the VPC
3. Select both public subnets
4. Enable Application Load Balancer for the AutoScalingGroup (ASG) 
5. Select the target group you created before
6. Ensure that you have health checks for both `EC2` and `ALB`
7. The desired capacity is 2
8. Minimum capacity is 2 
9. Maximum capacity is 4
10. Set scale out if CPU utilization reaches 90%
11. Ensure there is an `SNS` topic to send scaling notifications

#### Set Up Compute Resources for Webservers 

##### Provision the EC2 Instances for Webservers 

Now, you will need to create 2 separate launch templates for both the WordPress and Tooling websites

1. Create an EC2 Instance (`Centos`) each for WordPress and Tooling websites per Availability Zone (in the same Region).
2. Ensure that it has the following software installed

    * `python`
    * `ntp`
    * `net-tools`
    * `vim`
    * `wget`
    * `telnet`
    * `epel-release`
    * `htop`
    * `php`

3. Create an `AMI` out of the EC2 instance

* Select Webserver instance, then navigate to **`Actions > Image and templates > Create image`** and then we configure and create the AMI as shown in the image below:
  
##### Prepare Launch Template For Webservers (One per subnet)

1. Make use of the AMI to set up a launch template
2. Ensure the Instances are launched into a public subnet 
3. Assign appropriate security group
4. Configure Userdata to update `yum` package repository and install `wordpress` (*Only required on the WordPress launch template*)

###### Configure Target Groups for both Wordpress and Tooling servers

* From the EC2 dashboard, we navigate to **`Target groups > Create target group`** and then we do the following:

1. Select Instances as the target type
2. Ensure the protocol `HTTPS` on secure TLS port `443`
3. Ensure that the health check path is `/healthstatus`
4. Register targets accordingly
5. Ensure that health check passes for the target group


##### Create external load balancer

* From the EC2 dashboard, we navigate to **`Load Balancers > Create load balancer > Application load balancer > Create`** and then we do the following:

##### Create internal load balancere

* From the EC2 dashboard, we navigate to **`Load Balancers > Create load balancer> Application load balancer > Create`** and then we do the following:

* After the load balancer is created, we select it and navigate to **`Listeners and rules > Manage rules > Add rule `** and then we do the following:

1. Add a Name tag.
2. Define Rule conditions.
3. Define Rule actions.
4. Set rule priority.
5. Review and Create.

  
