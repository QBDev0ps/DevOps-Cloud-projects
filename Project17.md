# Automate Infrastructure With IaC using Terraform. Part 2

In this project, we shall go deeper in to automating other parts of our infrastructure on AWS. However, before we begin, it is very important to fully understand certain concepts around Networking.

## Understanding Basic Network Concepts

#### IP Address:

An IP address is a unique address that identifies a device on the internet or a local network. IP stands for "Internet Protocol," which is the set of rules governing the format of data sent via the internet or local network.

#### VPCs

A virtual private cloud (VPC) is an on-demand configurable pool of shared resources allocated within a public cloud environment, providing a certain level of isolation between the different organizations (denoted as users hereafter) using the resources.

#### Subnets

A subnet, or subnetwork, is a segmented piece of a larger network. More specifically, subnets are a logical partition of an IP network into multiple, smaller network segments.

#### CIDR Notation

CIDR notation (Classless Inter-Domain Routing) is an alternate method of representing a subnet mask. It is simply a count of the number of network bits (bits that are set to 1) in the subnet mask.

#### IP Routing

IP routing is the process of sending packets from a host on one network to another host on a different remote network.

#### Internet Gateway

An internet gateway is a horizontally scaled, redundant, and highly available VPC component that allows communication between your VPC and the internet


#### NAT Gateway

A NAT gateway is a Network Address Translation (NAT) service. You can use a NAT gateway so that instances in a private subnet can connect to services outside your VPC but external services cannot initiate a connection with those instances.

#### Routing Tables

A routing table, or routing information base (RIB), is a data table stored in a router or a network host that lists the routes to particular network destinations, and in some cases, metrics (distances) associated with those routes. The routing table contains information about the topology of the network immediately around it.

## Continue Infrastructure Automation with Terraform

In continuation from where we stopped in the previous project, we shall proceed to carry out the following:

1. Networking: Create our Networking resources

2. AWS Identity and Access Management: Compute and Access Control configuration automation

3.a

### Networking

Firstly, we need to create our network resources.Since we have created our public subnets in the previous project, we shall start off here by creating our private subnets.

#### Step 1: Create Private Subnets.

We shall create 4 private subnets keeping in mind the following principles:

- Make sure to use variables or `length()` function to determine the number of AZs.

- Use variables and `cidrsubnet()` function to allocate `vpc_cidr` for subnets.

- Keep variables and resources in separate files for better code structure and readability.

- Tag all the resources created so far. Explore how to use `format()` and `count` functions to automatically tag subnets with its respective number.

We create the private subnet by adding the following code snippet into our **`main.tf`** file.

```
# Create private subnets
resource "aws_subnet" "private" {
  count                   = var.preferred_number_of_private_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_private_subnets
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index + 2)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]

 tags = merge(
    var.tags,
    {
      Name = format("%s-PrivateSubnet-%s", var.name, count.index)
    },
  )

}
```

![create private subnets](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c3d72f4d-40f1-4901-9042-8f0976caa136)

As can be seen above, we have incorporated tagging in a bid to effectively manage our resource. We can now tag all our resources using the same format as shown below.

```
 tags = merge(
    var.tags,
    {
      Name = format("%s-NameofResource-%s", var.name, count.index)
    },
  )
```

This also means that anytime we need to make any change to the tags, we simply do that in one single place. Also, if we look closely at the code snippet, we can see that our key-value pairs are not hard coded. Rather, we have created variables for **`name`** and **`count`** that will be attached to each resource name.

#### Step 2: Create Internet Gateway. 

- We create a file called **`internet_gateway.tf`** and enter the following lines of code:

```
resource "aws_internet_gateway" "ig" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-%s-%s!", var.name, aws_vpc.main.id,"IG")
    }, 
  )
}
```

![create internet gateway](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/b9be0a3e-30ce-44d3-8352-80f0a1129127)

In the **`tags`** section of the code above, we have used **`format()`** function to dynamically generate a unique name for this resource. The first part of the **`%s`** takes the value of **`var_name`** as stored in our **variables.tf** file while the second takes the interpolated value **`aws_vpc.main.id`** and the third **`%s`** appends a literal string **`IG`** and finally an exclamation mark is added at the end.

#### Step 3: Create NAT Gateway.

We create a file called **`nat_gateway.tf`** and then enter the following block of code to create one NAT gateway and assign an elastic IP (EIP) address to it:

```
resource "aws_eip" "nat_eip" {
  vpc        = true
  depends_on = [aws_internet_gateway.ig]

  tags = merge(
    var.tags,
    {
      Name = format("%s-EIP", var.name)
    },
  )
}

resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat_eip.id
  subnet_id     = element(aws_subnet.public.*.id, 0)
  depends_on    = [aws_internet_gateway.ig]

  tags = merge(
    var.tags,
    {
      Name = format("%s-Nat", var.name)
    },
  )
}
```

![create NAT gateway](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e705db18-c298-4391-983d-85233bc5c0e2)

The NAT gateway has a dependency on the exitence of an elastic IP address so as seen in the code block above, we create an elastic IP first and then we create the NAT gateway. We can also see the use of **`depends_on`** to indicate that the Internet Gateway resource must be available before the resources in this step can be created.

#### Step 4: Create AWS Routes.

We create a file called **`route_tables.tf`** and then we use the following code blocks to create routes for both public and private subnets.

```
# create private route table
resource "aws_route_table" "private-rtb" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-Private-Route-Table-%s", var.name, var.environment)
    },
  )
}

# associate all private subnets to the private route table
resource "aws_route_table_association" "private-subnets-assoc" {
  count          = length(aws_subnet.private[*].id)
  subnet_id      = element(aws_subnet.private[*].id, count.index)
  route_table_id = aws_route_table.private-rtb.id
}

# create route table for the public subnets
resource "aws_route_table" "public-rtb" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-Public-Route-Table-%s", var.name, var.environment)
    },
  )
}

# create route for the public route table and attach the internet gateway
resource "aws_route" "public-rtb-route" {
  route_table_id         = aws_route_table.public-rtb.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.ig.id
}

# associate all public subnets to the public route table
resource "aws_route_table_association" "public-subnets-assoc" {
  count          = length(aws_subnet.public[*].id)
  subnet_id      = element(aws_subnet.public[*].id, count.index)
  route_table_id = aws_route_table.public-rtb.id
}
```

![create route tables](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/83e7825a-34bd-40ef-b9e3-127da1d7d4a6)

It is of note that we have introduced a new variable **`var.environment`** which we have defined accordingly in our **`variables.tf`** file as shown below.

![variables specify](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/818ced82-70db-48e2-b5f0-45153cdd6af9)

Now we proceed to run the commands below.

```
$ terraform plan

$ terraform apply
```

After executing the command, we can see that the following resources have been created:

- [x] - Our main vpc 
- [x] - 2 Public subnets 
- [x] - 4 Private subnets 
- [x] - 1 Internet Gateway
- [x] - 1 NAT Gateway
- [x] - 1 EIP
- [x] - 2 Route tables

### AWS Identity and Access Management

Now that we are done with the Networking part of our AWS set-up, we proceed to move on to Compute and Access Control configuration automation using Terraform.

#### Step 1: Create IAM Roles.
