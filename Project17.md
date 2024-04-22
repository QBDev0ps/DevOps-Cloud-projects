# Automate Infrastructure With IaC using Terraform. Part 2

In this project, we shall go deeper in to automating other parts of our infrastructure on AWS. However, before we begin, it is very important to fully understand certain concepts around Networking.

### Understanding Basic Network Concepts

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

### Continue Infrastructure Automation with Terraform

In continuation from where we stopped in the previous project, we shall proceed with the following steps:

#### Step 1: Create Private Subnets.

We shall create 4 private subnets keeping in mind the following principles:

- Make sure you use variables or `length()` function to determine the number of AZs.

- Use variables and `cidrsubnet()` function to allocate `vpc_cidr` for subnets.

- Keep variables and resources in separate files for better code structure and readability.

- Tag all the resources you have created so far. Explore how to use `format()` and `count` functions to automatically tag subnets with its respective number.

Due to the AZ of eu-central-1 region is not up to 4 which will return error since it is 4 private subnet that is needed, therefore random_shuffle resource is introduced and then specifying the maximum subnet:

```
resource "random_shuffle" "az_list" {
  input        = data.aws_availability_zones.available.names
  result_count = var.max_subnets
}
```

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
  
