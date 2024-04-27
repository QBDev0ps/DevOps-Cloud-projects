## Automate Infrastructure With IaC using Terraform. Part 3 - Refactoring

Improving cloud infrastructure automtion is esential for enhancing operational efficiency, reducing manual effort and achieving greater agility in managing and scaling resources in cloud environments.

In the past two previous projects we have developed AWS Infrastructure code using Terraform and tried to run it from our local workstation.
Now it is time to introduce some more advanced concepts and enhance our code.

In this project we shall explore, alternative Terraform [modules](https://developer.hashicorp.com/terraform/language/modules) and [backends](https://www.terraform.io/docs/language/settings/backends/index.html).

### Introduction to Terraform Modules

Simply put, terraform modules serve as best practices to structure our **`.tf`** files. At this time, we can appreciate the inherent difficulties in navigating through all the Terraform code blocks if they are all written in a single long **`.tf`** file. It is imperative for DevOps engineers to produce reusable and comprehensive IaC code structure, and one of the tool that Terraform provides out of the box is [**Modules**](https://www.terraform.io/docs/language/modules/index.html).

Modules are the main way to package and reuse resource configurations with Terraform. They serve as containers that allow us to logically group Terraform codes for similar resources in the same domain (e.g., Compute, Networking, AMI, etc.). A module consists of a collection of **`.tf`** and/or **`.tf.json`** files kept together in a directory. There are primarily two types of modules depending on how they are written (root and child modules). One **root module** can call other **child modules** and insert their configurations when applying Terraform configuration. This concept makes our code structure neater, and it allows different team members to work on different parts of configuration at the same time. 

We can also create and publish our modules to [Terraform Registry](https://registry.terraform.io/browse/modules) for others to use and vice-versa use someone's modules in our projects.

We can refer to existing **child modules** from our **root module** by specifying them as a source, as shown below:

```
module "network" {
  source = "./modules/network"
}
```

Note that the path to 'network' module is set as relative to our working directory.

Alternatively, we can also directly access resource outputs from the modules, like this:

```
resource "aws_elb" "example" {
  # ...

  instances = module.servers.instance_ids
}
```

In the example above, we will have to have module 'servers' to have an output file to expose variables for this resource.

### Refactoring our project using Modules.

In continuation of [Project 17](https://github.com/QuadriBello/DevOps-Cloud/blob/main/Project17.md), we shall make use of modules to refactor our entire code. This will help simplify our codebase. In the Project 17 [repository](https://github.com/darey-devops/PBL-project-17.git), we had a single list of long file for creating all of our resources, but this solution is far from optimal as it makes our codebase vey hard to read and understand thereby making potential future changes very cumbersome to implement.

To proceed, we shall break down all our Terraform codes to have all resources in their respective modules. We shall combine resources of a similar type into directories within a 'modules' directory. 

We begin our refactoring by creating a directory **`PBL-Project18-Terraform-Modular-Architecture`** which will be the root-module. Inside the root-module, we create a directory named **`modules`**:

```
$ mkdir PBL-Project18-Terraform-Modular-Architecture 

$ cd PBL-Project18-Terraform-Modular-Architecture

$ mkdir modules
```

Next, we navigate to the modules directory, and we create the directories that will hold the diiferent resources just as shown below:

```
- modules
  - ALB
  - EFS
  - RDS
  - Autoscaling
  - compute
  - VPC
  - security
```

We ensure that each module contains the following files:

```
- main.tf (or %resource_name%.tf) file(s) with resources blocks
- outputs.tf (optional, if you need to refer outputs from any of these resources in your root module)
- variables.tf (as we learned before - it is a good practice not to hard code the values and use variables)
```

For our **`root modules`**, we create the following files outside of the **`modules`** sub directory but still within our root directory:

```
main.tf

provider.tf

terraform.tfvars

vars.tf file.
```

Then we proceed to copy and refactor the code containing the resources that were created in [Project 17](https://github.com/QuadriBello/DevOps-Cloud/blob/main/Project17.md) and we paste them into each of their relevant modules.

The modularized code for this project can be found in this [repository]().


### Complete the Terraform configuration

Complete the rest of the codes yourself, so, the resulted configuration structure in your working directory may look like this:

```
└── PBL
    ├── modules
    |   ├── ALB
    |     ├── ... (module .tf files, e.g., main.tf, outputs.tf, variables.tf)     
    |   ├── EFS
    |     ├── ... (module .tf files) 
    |   ├── RDS
    |     ├── ... (module .tf files) 
    |   ├── autoscaling
    |     ├── ... (module .tf files) 
    |   ├── compute
    |     ├── ... (module .tf files) 
    |   ├── network
    |     ├── ... (module .tf files)
    |   ├── security
    |     ├── ... (module .tf files)
    ├── main.tf
    ├── providers.tf
    ├── terraform.tfvars
    └── variables.tf
```

Now, the code is much more well-structured and can be easily read, edited and reused by our DevOps team members.

Next, we proceed to run the following commands:

```
$ terraform init

$ terraform fmt

$ terraform validate
```

After confirmation that our configuration is valid, we execute the following commands:

```
$ terraform plan

$ terraform apply
```




### Introduction to Backend on [S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html) 

Each Terraform configuration can specify a backend, which defines where and how operations are performed, where state snapshots are stored, etc.
Our state file is basically where terraform stores all the state of the infrastructure in `json` format.

So far, we have been using the default backend, which is the **`local backend`** - it requires no configuration, and the state file is stored locally. This mode can be suitable for learning purposes, but it is not a robust solution, so it is better to store it in some more reliable and durable storage.

The second problem with storing this file locally is that, in a team of multiple DevOps engineers, other engineers will not have access to a state file stored locally on our computer.

To solve this, we will need to configure a backend where the state file can be accessed remotely by other DevOps team members. There are plenty of different standard backends supported by Terraform that we can choose from but since we are already using AWS - we shall choose an [S3 bucket as a backend](https://www.terraform.io/docs/language/settings/backends/s3.html).

Another useful option that is supported by S3 backend is [State Locking](https://www.terraform.io/docs/language/state/locking.html) - it is used to lock our current state for all operations that could write and change the state. This prevents others from acquiring the lock and potentially corrupting our state. State Locking feature for S3 backend is optional and requires another AWS service - [DynamoDB](https://aws.amazon.com/dynamodb/).

To proceed, we shall need to re-initialize Terraform to use S3 backend and to this, we will be carrying out the following:

- Add S3 and DynamoDB resource blocks before deleting the local state file
  
- Update terraform block to introduce backend and locking
  
- Re-initialize terraform

- Delete the local `tfstate` file and check the one in S3 bucket
  
- Add `outputs`
  
- `terraform apply`

**Step 1:** We create a file and named **`backend.tf`** then we add the following code and replace the name of the S3 bucket we created in [Project-16](https://expert-pbl.darey.io/en/latest/project16.html).

```
# Note: The bucket name may not work for you since buckets are unique globally in AWS, so you must give it a unique name.
resource "aws_s3_bucket" "terraform_state" {
  bucket = "dev-terraform-bucket"
  # Enable versioning so we can see the full revision history of our state files
  versioning {
    enabled = true
  }
  # Enable server-side encryption by default
  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }
}
```

