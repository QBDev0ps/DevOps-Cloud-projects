![image](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/9e45e935-42ae-4f7e-bc4c-d4d8645e5cd5)## AWS Networking implementation (VPC, Subnets, IG, NAT, Routing)

#### What is an Amazon VPC?

An Amazon Virtual Private Cloud (VPC) is like your own private section of the Amazon cloud, where you can place and manage your resources (like servers or databases). You control who and what can go in and out, just like a gated community.

**The essential steps to creating a VPC and configuring core network services. The topics to be covered are:**

+ The Default VPC
  
+ Creating a new VPC
  
+ Creating and configuring subnets
  
#### The Default VPC

The Default VPC is like a starter pack provided by Amazon for your cloud resources. It's a pre-configured space in the Amazon cloud where you can immediately start deploying your applications or services. It has built-in security and network settings to help you get up and running quickly, but you can adjust these as you see fit.

![VPC](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/0d8d5f0b-d623-4461-a24c-b5bfb7c32366)

A Default VPC, which Amazon provides for you in each region (think of a region as a separate city), is like a pre-built house in that city. This house comes with some default settings to help you move in and start living (or start deploying your applications) immediately. But just like a real house, you can change these settings according to your needs.

![VPC region](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/99d061f5-a719-4cec-899f-d8290691011a)

#### Creating a new VPC

As we want to learn step by step and observe the components, choose the "VPC only" option, we'll use the "VPC and more" option later. Enter "first-vpc" as the name tag and "10.0.0.0/16" as the IPv4 CIDR. The "10.0.0.0/16" will be the primary IPv4 block and you can add a secondary IPv4 block e.g., "100.64.0.0/16". The use case of secondary CIDR block could be because you're running out of IPs and need to add additional block, or there's a VPC with overlapping CIDR which you need to peer or connect. See this [blog post](https://aws.amazon.com/blogs/networking-and-content-delivery/how-to-solve-private-ip-exhaustion-with-private-nat-solution/) on how a secondary CIDR block is being used in an overlapping IP scenario.

![create VPC](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/fc6bb704-da3a-4aaf-936e-14d987156731)

Leave the tags as default, you can add more tags if you want and click **`CREATE VPC`**

![tags VPC](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/f13176f5-1696-49fa-9e47-aa1914edb302)

As soon as the VPC is created, it's assigned with a vpc-id and there's a route table created that serves as the main route table - **`rtb-02a7937200231cb27`** in below example.

![first VPC](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/83f5c094-f772-4352-9eee-938657fbdf58)

Now you have a VPC and a route table, but you won't be able to put anything inside. If you try to create an EC2 instance for example, you can't proceed as it requires subnets.

#### Creating and configuring subnets

##### What are Subnets?

Subnets are like smaller segments within a VPC that help you organize and manage your resources. Subnets are like dividing an office building into smaller sections, where each section represents a department. In this analogy, subnets are created to organize and manage the network effectively.

![subnet](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e95a3a4b-9b4d-4ce0-b264-80acd061d715)

|**Subnet name**	|**AZ**	        |**CIDR block**|
|-----------------|---------------|--------------|
| subnet-public1a	|  eu-north-1a	| 10.0.11.0/24 |
|subnet-public2b	|  eu-north-1b	| 10.0.12.0/24 |
|subnet-private1a	|  eu-north-1a	| 10.0.1.0/24  |
|subnet-private2b	|  eu-north-1b	| 10.0.2.0/24  |

Go to VPC > Subnets > Create Subnets and select the VPC that you've created previously 

![create subnets](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/85ca64ee-53a9-4457-80b2-2ebb271889b4)

click on **`CREATE SUBNET`**

![create subnet 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/af574caa-9ab5-4435-b91e-9ae16eca7ba3)

Enter the subnet settings detail. Don't click the **"Create subnet"** button just yet, click the Add new subnet button to add the remaining subnets then after completing all the required subnets, click **"Create subnet"** Note: if you don't choose a zone, it will be randomly picked by AWS.

![subnet settings](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e2dd4161-b594-4591-a098-27695a311b18)

Once done, you should see all the subnets you just created on the console. If you missed any, just create a subnet and select your desired VPC. As of now, you can deploy EC2 instances into the VPC by selecting one of the subnets, but the public subnet doesn't have any Internet access at this stage. When you select a public subnet > route, you'll see it uses the main route table and only has the local route, no default route for Internet access.

![subnets created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/948ea22d-7ef0-4763-bc95-e96b9c7bc819)

#### Understanding Public and Private Subnets in AWS VPC

In the world of AWS VPC, think of subnets as individual plots in your land (VPC). Some of these plots (subnets) have direct road access (internet access) - these are public subnets. Others are more private, tucked away without direct road access - these are private subnets.

#### Creating a Public Subnet

Creating a public subnet is like creating a plot of land with direct road (internet) access. Here's how you do it:

+ Go to the AWS VPC page.

+ Find **'Subnets'**, click on it, then click **'Create subnet'**.

+ Give this new plot a name, select the big plot (VPC) you want to divide, and leave the IP settings as they are.

+ Attach an Internet Gateway to this subnet to provide the road (internet) access.

+ Update the route table associated with this subnet to allow traffic to flow to and from the internet.

#### Creating a Private Subnet

Creating a private subnet is like creating a secluded plot without direct road (internet) access. Here's how you do it:

+ Go to the AWS VPC page.

+ Find **'Subnets'**, click on it, then click **'Create subnet'**.

+ Give this new plot a name, select the big plot (VPC) you want to divide, and leave the IP settings as they are.

+ Don't attach an Internet Gateway to this subnet, keeping it secluded.

+ The route table for this subnet doesn't allow direct traffic to and from the internet.
  
#### Working with Public and Private Subnets

Public subnets are great for resources that need to connect to the internet, like web servers. Private subnets are great for resources that you don't want to expose to the internet, like databases.

Understanding public and private subnets helps you to organize and protect your AWS resources better. Always remember, use public subnets for resources that need internet access and private subnets for resources that you want to keep private.

#### Internet Gateway and Routing Table

#### Introduction to Internet Gateway and Routing Table

Just like in a real city, in your virtual city (VPC), you need roads (Internet Gateway) for people (data) to come and go. And you also need a map or GPS (Routing Table) to tell people (data) which way to go to reach their destination.

##### What is an Internet Gateway?

An Internet Gateway in AWS is like a road that connects your city (VPC) to the outside world (the internet). Without this road, people (data) can't come in or go out of your city (VPC).

#### Deep Dive into Internet Gateways

To give your public subnet access to the main road (internet), you need an Internet Gateway. This acts like the entrance and exit to your property. We'll show you how to create and attach an Internet Gateway to your VPC.

#### Public Subnets

Technically, the subnets are still private. You'll need these to make it work as public subnets:

+ An Internet Gateway (IGW) attached to the VPC
  
+ Route table with default route towards the IGW
  
+ Public IP assigned to the AWS resources (e.g., EC2 instances)
  
Go to VPC > Internet gateway and click **"Create internet gateway"**

![internet gateways](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e9fad9d3-975a-4f7a-be97-e3a8bffa343e)

Put a name tag and click **"create internet gateway"**

![test gateway](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c1cc7d87-302f-4f88-8f07-0a65fe15302a)

Attach the IGW to the test-vpc

![attach to a vpc](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/4ee73697-a2a0-487b-ab12-608aecbcad8e)

Select the VPC

![attach to vpc 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/817bef99-dea3-4854-a258-d1f9064663dd)

We want the private subnets to be private, we don't want the private subnets to have a default route to the Internet. For that, we'll need to create a separate route table for the public subnets.

##### What is a Routing Table?

A Routing Table is like a map or GPS. It tells the people (data) in your city (VPC) which way to go to reach their destination. For example, if the data wants to go to the internet, the Routing Table will tell it to take the road (Internet Gateway) that you built.

##### Creating and Configuring Routing Tables

Now that we have our entrance and exit (Internet Gateway), we need to give directions to our resources. This is done through a Routing Table. It's like a map, guiding your resources on how to get in and out of your VPC.

Let's go to the route table menu and create a route table for the public subnets.

