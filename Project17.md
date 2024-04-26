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

In continuation from where we stopped in the previous project, we shall continue our implementation using the same network architecture diagram. In this project, we shall proceed to carry out the following:

1. Networking Resources: We create our Networking resources.

2. AWS Identity and Access Management: Identity Access Control configuration automation.

3. Compute Resources: Automate creation of compute resources and associated configuration.

![152831445-844e3865-0317-4bf4-969a-490a7c1e06ba](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/fc82a9b8-d0b1-4328-83ba-aa569c54dfaa)

### Networking Resources

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

![terraform plan](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/89814135-f19b-4ae0-8e7e-975f7962e157)

![terraform apply](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e8325f04-e78d-40f0-882f-606ad57c6c01)

After executing the command, we can see that the following resources have been created:

- [x] - Our main vpc 
- [x] - 2 Public subnets 
- [x] - 4 Private subnets 
- [x] - 1 Internet Gateway
- [x] - 1 NAT Gateway
- [x] - 1 EIP
- [x] - 2 Route tables

![network resources created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/8ac1739d-d9cb-4d53-a3f9-e4fc59f8c8e2)


### AWS Identity and Access Management

Now that we are done with the Networking part of our AWS set-up, we proceed to move on to Compute and Access Control configuration automation using Terraform.

#### IAM and Roles

We want to pass an [IAM](https://docs.aws.amazon.com/iam/index.html) [Role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) to our EC2 instances to give them access to some specific resources. To do this, we will need to create **`AssumeRole`** then next, we create an IAM policy for the role and afterwards, we attach the policy to the role.

#### Step 1:  Create [`AssumeRole`](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html).

Assume Role uses Security Token Service (STS) API that returns a set of temporary security credentials that you can use to access AWS resources that you may not normally have access to. These temporary credentials consist of an access key ID, a secret access key, and a security token. Typically,  **`AssumeRole`** is used within your account or for cross-account access.

We proceed by creating a new file **`roles.tf`** and adding the following code.

```
resource "aws_iam_role" "ec2_instance_role" {
  name = "ec2_instance_role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Sid    = ""
        Principal = {
          Service = "ec2.amazonaws.com"
        }
      },
    ]
  })

  tags = merge(
    var.tags,
    {
      Name = "aws assume role"
    },
  )
}
```

![create assume role](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e8416b46-3044-4c31-8246-bff846c540d2)

In the above code block, we are creating **`AssumeRole`** with **`AssumeRole policy`**. It grants to an entity, (in our case an EC2 instance) permissions to assume the role.

#### Step 2: Create an [IAM policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html) for this role.

This is where we need to define a required policy (i.e., permissions) according to our requirements. Here, we define a policy allowing an IAM role to perform an action **`describe`** applied to EC2 instances.

```
resource "aws_iam_policy" "policy" {
  name        = "ec2_instance_policy"
  description = "A test policy"
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = [
          "ec2:Describe*",
        ]
        Effect   = "Allow"
        Resource = "*"
      },
    ]

  })

  tags = merge(
    var.tags,
    {
      Name = "aws assume policy"
    },
  )

}
```

![create iam policy for role](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/b8d3a9be-166b-445c-9be0-06fd5fc50795)

#### Step 3: Attach the `Policy` to the `IAM Role`. 
    
This is where, we will be attaching the policy which we created above, to the role we created in the first step.

```
resource "aws_iam_role_policy_attachment" "test-attach" {
  role       = aws_iam_role.ec2_instance_role.name
  policy_arn = aws_iam_policy.policy.arn
}
```

![attach policy to IAM role](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/3b548c38-83ee-4cf4-8194-eddc02127be9)

#### Step 4: Create an [`Instance Profile`](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html) and interpolate the `IAM Role`.

```
resource "aws_iam_instance_profile" "ip" {
  name = "aws_instance_profile_test"
  role = aws_iam_role.ec2_instance_role.name
}
```

![create instance profile](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/68af537f-58e7-4246-b83d-4ea96d528042)

### Compute Resources

At this stage, we need to create our compute resources and all other associated resources and configuration. These include Security Groups, Target Group (for Nginx, Wordpress and Tooling), certificate from AWS certificate manager, an External Application Load Balancer and Internal Application Load Balancer, launch template (for Bastion, Tooling, Nginx and Wordpress), an Auto Scaling Group (ASG) (for Bastion, Tooling, Nginx and Wordpress), Elastic Filesystem (EFS), and a Relational Database (RDS).


#### STEP 1: Create Security Groups.

We are going to create all the security groups in a single file, then we are going to refrence this security group within each resources that needs it.

To proceed, we create a new file **`security.tf`** and paste in the following commands to create security groups for the Internal and External load balancer, the bastion server, Nginx server, the tooling and wordpress webserver and the data layer:

```
# security group for alb, to allow acess from anywhere for HTTP and HTTPS traffic
resource "aws_security_group" "ext-alb-sg" {
  name        = "ext-alb-sg"
  vpc_id      = aws_vpc.main.id
  description = "Allow TLS inbound traffic"

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "HTTPS"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }


  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "ext-alb-sg"
    },
  )

}


# security group for bastion, to allow access into the bastion host from your IP
resource "aws_security_group" "bastion_sg" {
  name        = "bastion_sg"
  vpc_id      = aws_vpc.main.id
  description = "Allow incoming HTTP connections."

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "Bastion-SG"
    },
  )
}

#security group for nginx reverse proxy, to allow access only from the external load balancer and bastion instance
resource "aws_security_group" "nginx-sg" {
  name   = "nginx-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "nginx-SG"
    },
  )
}

resource "aws_security_group_rule" "inbound-nginx-http" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.ext-alb-sg.id
  security_group_id        = aws_security_group.nginx-sg.id
}

resource "aws_security_group_rule" "inbound-bastion-ssh" {
  type                     = "ingress"
  from_port                = 22
  to_port                  = 22
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.nginx-sg.id
}

# security group for ialb, to have acces only from nginx reverser proxy server
resource "aws_security_group" "int-alb-sg" {
  name   = "int-alb-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "int-alb-sg"
    },
  )

}

resource "aws_security_group_rule" "inbound-ialb-https" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.nginx-sg.id
  security_group_id        = aws_security_group.int-alb-sg.id
}

# security group for webservers, to have access only from the internal load balancer and bastion instance
resource "aws_security_group" "webserver-sg" {
  name   = "webserver-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "webserver-sg"
    },
  )

}

resource "aws_security_group_rule" "inbound-web-https" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.int-alb-sg.id
  security_group_id        = aws_security_group.webserver-sg.id
}

resource "aws_security_group_rule" "inbound-web-ssh" {
  type                     = "ingress"
  from_port                = 22
  to_port                  = 22
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.webserver-sg.id
}

# security group for datalayer to alow traffic from websever on nfs and mysql port and bastiopn host on mysql port
resource "aws_security_group" "datalayer-sg" {
  name   = "datalayer-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "datalayer-sg"
    },
  )
}

resource "aws_security_group_rule" "inbound-nfs-port" {
  type                     = "ingress"
  from_port                = 2049
  to_port                  = 2049
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.webserver-sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}

resource "aws_security_group_rule" "inbound-mysql-bastion" {
  type                     = "ingress"
  from_port                = 3306
  to_port                  = 3306
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}

resource "aws_security_group_rule" "inbound-mysql-webserver" {
  type                     = "ingress"
  from_port                = 3306
  to_port                  = 3306
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.webserver-sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}
```

Security group for alb, to allow acess from anywhere for HTTP and HTTPS traffic.

![security group for alb](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/49535659-6b78-4eb9-9861-c6148b3f5566)

Security group for bastion, to allow access into the bastion host from your IP.

![security group for bastion](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/d8339829-e967-4bbf-8118-eb0d8a26c397)

Security group for nginx reverse proxy, to allow access only from the external load balancer and bastion instance.

![security group for nginx reverse proxy](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/8b55e4ba-b1b9-41c7-91c3-f4178983e346)

Security group for ialb, to have access only from nginx reverse proxy server.

![security group for ialb](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/b633d9ce-c08b-420b-b84b-65f0c7c033a8)

Security group for webservers, to have access only from the internal load balancer and bastion instance.

![security group for webservers](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/563bc1bf-dd46-4a09-a2cb-5fdd082b53b5)

Security group for datalayer to alow traffic from websever on nfs and mysql port and bastiopn host on mysql port.

![security group for data layer](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/f7230468-0054-4e47-ab5d-081ed7a43358)

#### STEP 2: Create Certificate From Amazon Certificate Manager.

We create a new file **`cert.tf`** and then we paste in the following code to create and validate a certificate AWS:

```
# The entire section creates a certificate, public zone, and validates the certificate using DNS method.

# Create the certificate using a wildcard for all the domains created in qbdevops.co.uk
resource "aws_acm_certificate" "qbdevops" {
  domain_name       = "*.qbdevops.co.uk"
  validation_method = "DNS"
}

# calling the hosted zone
data "aws_route53_zone" "qbdevops" {
  name         = "qbdevops.co.uk"
  private_zone = false
}

# selecting validation method
resource "aws_route53_record" "qbdevops" {
  for_each = {
    for dvo in aws_acm_certificate.qbdevops.domain_validation_options : dvo.domain_name => {
      name   = dvo.resource_record_name
      record = dvo.resource_record_value
      type   = dvo.resource_record_type
    }
  }

  allow_overwrite = true
  name            = each.value.name
  records         = [each.value.record]
  ttl             = 60
  type            = each.value.type
  zone_id         = data.aws_route53_zone.qbdevops.zone_id
}

# validate the certificate through DNS method
resource "aws_acm_certificate_validation" "qbdevops" {
  certificate_arn         = aws_acm_certificate.qbdevops.arn
  validation_record_fqdns = [for record in aws_route53_record.qbdevops : record.fqdn]
}

# create records for tooling
resource "aws_route53_record" "tooling" {
  zone_id = data.aws_route53_zone.qbdevops.zone_id
  name    = "tooling.qbdevops.co.uk"
  type    = "A"

  alias {
    name                   = aws_lb.ext-alb.dns_name
    zone_id                = aws_lb.ext-alb.zone_id
    evaluate_target_health = true
  }
}


# create records for wordpress
resource "aws_route53_record" "wordpress" {
  zone_id = data.aws_route53_zone.qbdevops.zone_id
  name    = "wordpress.qbdevops.co.uk"
  type    = "A"

  alias {
    name                   = aws_lb.ext-alb.dns_name
    zone_id                = aws_lb.ext-alb.zone_id
    evaluate_target_health = true
  }
}
```

![create certificate](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c509f643-6089-4b7e-a5ef-b5f418b29c23)

#### STEP 3: Create External Application Load Balancer.

In this step, we shall create an external (Internet facing) [Application Load Balancer (ALB)](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/application-load-balancer-getting-started.html) which serves to balance traffic for the Nginx servers.

We proceed by creating a file **`alb.tf`** and then we paste in the following code to create the external(internet-facing) load balancer, after which we create the target group and then, we create the lsitener rule:

```
# create an ALB to balance the traffic between the Instances

resource "aws_lb" "ext-alb" {
  name     = "ext-alb"
  internal = false
  security_groups = [
    aws_security_group.ext-alb-sg.id,
  ]

  subnets = [
    aws_subnet.public[0].id,
    aws_subnet.public[1].id
  ]

  tags = merge(
    var.tags,
    {
      Name = "TCS-ext-alb"
    },
  )

  ip_address_type    = "ipv4"
  load_balancer_type = "application"
}
```

To inform our ALB where to  route the traffic we need to create a [`Target Group`](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html) to point to its targets:

```
resource "aws_lb_target_group" "nginx-tgt" {
  health_check {
    interval            = 10
    path                = "/healthstatus"
    protocol            = "HTTPS"
    timeout             = 5
    healthy_threshold   = 5
    unhealthy_threshold = 2
  }
  name        = "nginx-tgt"
  port        = 443
  protocol    = "HTTPS"
  target_type = "instance"
  vpc_id      = aws_vpc.main.id
}
```

Then we will need to create a [`Listner`](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-listeners.html) for this target group:

```
resource "aws_lb_listener" "nginx-listner" {
  load_balancer_arn = aws_lb.ext-alb.arn
  port              = 443
  protocol          = "HTTPS"
  certificate_arn   = aws_acm_certificate_validation.qbdevops.certificate_arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.nginx-tgt.arn
  }
}
```

Next, we add the following outputs to **`output.tf`** to print the output values on screen:.

```
output "alb_dns_name" {
  value = aws_lb.ext-alb.dns_name
}

output "alb_target_group_arn" {
  value = aws_lb_target_group.nginx-tgt.arn
}
```

#### STEP 4: Create Internal Application Load Balancer.

Next, we create an Internal (Internal) [Application Load Balancer (ALB)](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-internal-load-balancers.html) to balance traffic between the webservers.

For the Internal Load balancer we will follow the same concepts as with the external load balancer and we add the following code snippets inside the **`alb.tf`** file:

```
# ----------------------------
#Internal Load Balancers for webservers
#---------------------------------

resource "aws_lb" "ialb" {
  name     = "ialb"
  internal = true
  security_groups = [
    aws_security_group.int-alb-sg.id,
  ]

  subnets = [
    aws_subnet.private[0].id,
    aws_subnet.private[1].id
  ]

  tags = merge(
    var.tags,
    {
      Name = "TCS-int-alb"
    },
  )

  ip_address_type    = "ipv4"
  load_balancer_type = "application"
}
```

To inform our ALB where to route the traffic, we need to create a [`Target Group`](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html) to point to its targets:

```
# --- target group  for wordpress -------

resource "aws_lb_target_group" "wordpress-tgt" {
  health_check {
    interval            = 10
    path                = "/healthstatus"
    protocol            = "HTTPS"
    timeout             = 5
    healthy_threshold   = 5
    unhealthy_threshold = 2
  }

  name        = "wordpress-tgt"
  port        = 443
  protocol    = "HTTPS"
  target_type = "instance"
  vpc_id      = aws_vpc.main.id
}

# --- target group for tooling -------

resource "aws_lb_target_group" "tooling-tgt" {
  health_check {
    interval            = 10
    path                = "/healthstatus"
    protocol            = "HTTPS"
    timeout             = 5
    healthy_threshold   = 5
    unhealthy_threshold = 2
  }

  name        = "tooling-tgt"
  port        = 443
  protocol    = "HTTPS"
  target_type = "instance"
  vpc_id      = aws_vpc.main.id
}
```

Then, we will need to create a [`Listner`](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-listeners.html) for this target Group:

```
# For this aspect a single listener was created for the wordpress which is default,
# A rule was created to route traffic to tooling when the host header changes


resource "aws_lb_listener" "web-listener" {
  load_balancer_arn = aws_lb.ialb.arn
  port              = 443
  protocol          = "HTTPS"
  certificate_arn   = aws_acm_certificate_validation.qbdevops.certificate_arn


  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.wordpress-tgt.arn
  }
}

# listener rule for tooling target

resource "aws_lb_listener_rule" "tooling-listener" {
  listener_arn = aws_lb_listener.web-listener.arn
  priority     = 99

  action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.tooling-tgt.arn
  }

  condition {
    host_header {
      values = ["tooling.qbdevops.co.uk"]
    }
  }
}
```
