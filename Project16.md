Automate Infrastructure With IaC using Terraform Part 1
============================================

After we have built AWS infrastructure for 2 websites manually, it is time to automate the process using Terraform. 

Let us start building the same set up with the power of Infrastructure as Code (IaC)

![152831445-844e3865-0317-4bf4-969a-490a7c1e06ba](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/a07fce80-4415-4e3c-961e-ff5e1af744cf)

### Introducion to Infrastructure as Code with Terraform

Infrastructure as code is writing your infrastructure (servers, databases, networks etc.) to version controlled scripts or code. It is your entire environment and infrastructure represented and maintained in a configuration code base. You can use one command to build your whole infrastructure and you can also use one command to tear it all down.

Terraform is an open source infrastructure as code tool created by Hashicorp. It is agentless and it can communicate via providers with other tools such as AWS, Kubernetes, Jenkins, Docker etc. Other benefits of terraform include:

**i.** Terraform is multicloud. You can deploy on multiple cloud environments with a single configuration file.

**ii.** Terraform is stateful. It allows you to track your resource changes throughout your deployment in a state file which is the single source of truth for your environment. Terraform uses this state file to determine the changes that you're making to manage your infrastructure to make sure that it matches with your configuration.

**iii.** Terraform is version controlled. All your engineers and developers can work on a single publicly available code base.

**iv.** Terraform is declarative. You declare what the end state of your infrastructure should be like and terraform will get you there.

**v.** There is no need for manual click ups. There won't be any need to manually deploy infrastructure.

**vi.** Terraform is great for disaster recovery. It helps to save a lot of money as there might be no need to spend huge amounts on disatser recovery if you can spin up and spin down all your infrastructure in seconds.

Terraform helps to kinimize user errors.

### The secrets of writing quality Terraform code

The secret recipe of a successful Terraform projects consists of:

- Your understanding of your goal (desired AWS infrastructure end state)
  
- Your knowledge of the `IaC` technology used (in this case - Terraform)
  
- Your ability to effectively use up to date Terraform documentation [here](https://www.terraform.io/docs/configuration)

As we go along completing this project, we will get familiar with [Terraform-specific terminology](https://www.terraform.io/docs/glossary.html), such as: 

- [Attribute](https://www.terraform.io/docs/glossary.html#attribute)
- [Resource](https://www.terraform.io/docs/glossary.html#resource)
- [Interpolations](https://www.terraform.io/docs/glossary.html#interpolation)
- [Argument](https://www.terraform.io/docs/glossary.html#argument)
- [Providers](https://www.terraform.io/docs/providers/index.html)
- [Provisioners](https://www.terraform.io/docs/language/resources/provisioners/index.html)
- [Input Variables](https://www.terraform.io/docs/glossary.html#variables)
- [Output Variables](https://www.terraform.io/docs/glossary.html#output-values)
- [Module](https://www.terraform.io/docs/glossary.html#module)
- [Data Source](https://www.terraform.io/docs/glossary.html#data-source)
- [Local Values](https://www.terraform.io/docs/configuration-0-11/locals.html)
- [Backend](https://www.terraform.io/docs/glossary.html#backend)  

We shall make sure to understand them and know when to use each of them. 

Another important concept is [`data type`](https://en.wikipedia.org/wiki/Data_type). This is a general programing concept, it refers to how data is represented in a programming language and defines how a compiler or interpreter can use the data. Common data types are:

- Integer
- Float
- String
- Boolean, etc.

#### Best practices

* Ensure that every resource is tagged using multiple key-value pairs. We will see this in action as we go along.
* Try to write reusable code, avoid hard coding values wherever possible. (For learning purpose, we will start by hard coding, but gradually refactor our work to follow best practices).

### Project Prerequisites 

These are important prerequisites before we begin writing Terraform code.

**i.** Create an IAM user, name it `terraform` (*ensure that the user has only programatic access to your AWS account*) and grant this user `AdministratorAccess` permissions.

**ii.** Copy the secret access key and access key ID. Save them in a notepad temporarily.

**iii.** Configure programmatic access from your workstation to connect to AWS using the access keys copied above and a [Python SDK (boto3)](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html). Workstation must have [Python 3.6](https://www.python.org/downloads/) or higher.

For Windows, use `gitbash`, For Mac, simply open a `terminal`. Read [here](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html) to configure the Python SDK properly. 

For easier authentication configuration - use [AWS CLI](https://aws.amazon.com/cli/) with `aws configure` command.

**iv.** Create an [S3 bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html) to store Terraform state file. It can be named like `<yourname>-dev-terraform-bucket` (*Note: S3 bucket names must be unique unique within a region partition, you can read about S3 bucken naming [in this article](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucketnamingrules.html)*). We will use this bucket from Project-17 onwards.

![terraform s3 bucket](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/1aa020a6-155e-4901-87bc-ea87c58674e5)

**v.** When we have configured authentication and installed `boto3`, we make sure we can programmatically access our AWS account by running following commands in `>python`:

```
import boto3
s3 = boto3.resource('s3')
for bucket in s3.buckets.all():
    print(bucket.name)
```

We shall see our previously created S3 bucket name - `<yourname>-dev-terraform-bucket`

![access to s3 bucket confirmed aws cli setup confirmed](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/51f10785-24a0-48e9-b745-4cf5ccbb13b0)

### Base Infrastructure Automation

We shall now to make use of terraform to set up our infrastructure.

#### VPC | Subnets | Security Groups

We proceed by creating a directory structure. We open Visual Studio Code and do the following:

**i.** Create a folder called `PBL`.
  
**ii.** Create a file in the folder, name it `main.tf`.

After doing this, the setup looks as shown in the image below.

![maintf](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/245fd673-534a-472a-ac8e-0d412bb7527a)

#### Provider and VPC resource section

We set up Terraform CLI as per [this instruction](https://learn.hashicorp.com/tutorials/terraform/install-cli).

**i.** Add `AWS` as a provider, and a resource to create a VPC in the `main.tf` file.
  
**ii.** Provider block informs Terraform that we intend to build infrastructure within AWS.
  
**iii.** Resource block will create a VPC.

```
provider "aws" {
  region = "eu-central-1"
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = "172.16.0.0/16"
  enable_dns_support             = "true"
  enable_dns_hostnames           = "true"
  enable_classiclink             = "false"
  enable_classiclink_dns_support = "false"
}
```

**Note:** We can change the configuration above to create our VPC in another region that is closer to us. The same applies to all configuration snippets that will follow.

![create vpc](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/722f83b1-8238-4bbe-8551-7906c2692f9b)

**iii.** The next thing we need to do, is to download necessary plugins for Terraform to work. These plugins are used by `providers` and `provisioners`. At this stage, we only have `provider` in our `main.tf` file. So, Terraform will just download plugin for AWS provider.
  
**iv.** Lets accomplish this with `terraform init` command as seen in the below image.

![terraform init](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/5c6173d3-1a9a-4db6-b908-795b6c35daef)

***Observation***: 

- We notice that a new directory has been created: `.terraform\...`. This is where Terraform keeps plugins. Generally, it is safe to delete this folder. It just means that we must execute `terraform init` again, to download them.

**v.** Moving on, let us create the only resource we just defined. `aws_vpc`. But before we do that, we should check to see what terraform intends to create before we tell it to go ahead and create it.

* We run `terraform plan`
  
* Then, since we are happy with changes planned, we execute `terraform apply`

![terraform plan apply](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/748473c7-a589-48e4-9958-80c16187db10)

***Observations***: 

- A new file is created `terraform.tfstate` This is how Terraform keeps itself up to date with the exact state of the infrastructure. It reads this file to know what already exists, what should be added, or destroyed based on the entire terraform code that is being developed.
   
- When we observed closely, we realised that another file gets created during planning and apply. But this file gets deleted immediately. `terraform.tfstate.lock.info` This is what Terraform uses to track, who is running its code against the infrastructure at any point in time. This is very important for teams working on the same Terraform repository at the same time. The lock prevents a user from executing Terraform configuration against the same infrastructure when another user is doing the same - it allows to avoid duplicates and conflicts.
   
Its content is usually like this.

```
    {
        "ID":"e5e5ad0e-9cc5-7af1-3547-77bb3ee0958b",
        "Operation":"OperationTypePlan","Info":"",
        "Who":"dare@Dare","Version":"0.13.4",
        "Created":"2020-10-28T19:19:28.261312Z",
        "Path":"terraform.tfstate"
    }
```

It is a `json` format file that stores information about a user: user's `ID`, what operation he/she is doing, timestamp, and location of the `state` file.
