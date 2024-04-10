Automate Infrastructure With IaC using Terraform Part 1
============================================

After we have built AWS infrastructure for 2 websites manually, it is time to automate the process using Terraform. 

Let us start building the same set up with the power of Infrastructure as Code (IaC)

![152831445-844e3865-0317-4bf4-969a-490a7c1e06ba](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/a07fce80-4415-4e3c-961e-ff5e1af744cf)

### Introducion to Infrastructure as Code with Terraform


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

**v.** When we have configured authentication and installed `boto3`, we make sure we can programmatically access our AWS account by running following commands in `>python`:

```
import boto3
s3 = boto3.resource('s3')
for bucket in s3.buckets.all():
    print(bucket.name)
```
We shall see our previously created S3 bucket name - `<yourname>-dev-terraform-bucket`


### Base Infrastructure Automation

#### VPC | Subnets | Security Groups

We proceed by creating a directory structure. We open Visual Studio Code and do the following:

* Create a folder called `PBL`.
  
* Create a file in the folder, name it `main.tf`
