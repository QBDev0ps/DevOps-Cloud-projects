# Automate Infrastructure With IaC using Terraform. Part 4 - Terraform Cloud

In [Project 18](https://github.com/QuadriBello/DevOps-Cloud/blob/main/Project18.md), we refactored our terraform codes into modules to make our code base less cumbersome and more reusable and then subsequently we introduced the concept of backends and we migrated our `.tf ` state file to an AWS S3 bucket to enable easy collaboration with other members of the DevOps team. In this project we shall incorporate more new concepts into our deployment and ensure that our overall infrastructure is working by the time we are done.

Up till now, we have had to install Terraform ourselves and we have been running Terraform codes from a command line on our local computer. However, in the Cloud world it is quite common to provide a managed version of an open-source software like Terraform. Managed means that we do not have to install, configure and maintain it ourselves - we just have to create an account and use it "as A Service".

### Introduction to Terraform Cloud

[Terraform Cloud](https://www.hashicorp.com/products/terraform) is a managed IaC service that provides you with Terraform CLI to provision infrastructure, either on demand or in response to various events. It helps to enhance collaboration between developers and DevOps engineers, simplify workflow and improve security overall around the product.

By default, Terraform CLI performs operations on the server when it is invoked, it is perfectly fine if you have a dedicated role who can launch it, but if you have a team who works with Terraform - you need a consistent remote environment with remote workflow and shared state to run Terraform commands. This is where Terraform Cloud comes in. It runs in a consistent and reliable environment, and includes easy access to shared state and secret data, access controls for approving changes to infrastructure, a private registry for sharing Terraform modules, detailed policy controls for governing the contents of Terraform configurations and more. Teams can connect Terraform to version control, share variables, run Terraform in a stable remote environment, and securely store remote state.

Terraform Cloud executes Terraform commands on disposable virtual machines, this remote execution is also called [remote operations](https://developer.hashicorp.com/terraform/cloud-docs/run/remote-operations).

In this project, in addition to terraform cloud, we shall also be utilizing the combined powers of Packer and Ansible to automate infrastructure provisioning in AWS.

* [Packer](https://www.packer.io/) is an open source tool for creating identical machine images for multiple platforms from a single source configuration. Packer is lightweight, runs on every major operating system, and is highly performant, creating machine images for multiple platforms in parallel.

* [Ansible](https://www.ansible.com/) is an open-source automation tool, or platform, used for IT tasks such as configuration management, application deployment, intraservice orchestration, and provisioning.

By combining these tools, we shall automate the provisioning process and ensure consistent, reproducible infrastructure deployments.

### Build AMIs Using Packer

As stated before, we shall use packer in this project to create the AMIs for the launch templates used by the Auto Scaling Group. We have the Packer codes we used for this purpose stored in this repository. Packer creates the instances, provision the instances using shell scripts and creates the AMIs. It also deletes the instances after the completion of the AMIs creation process.

**1.** Run the Packer command to build AMIs

To carry out our AMi builds, we run the packer build command as shown below:

**i.** We build the bastion ami using the following command:

**`$ packer build bastion.pkr.hcl`**

![packer build bastion](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/84bf2053-b2e3-45a9-92a6-3fc8dbf328c0)

**ii.** We build the nginx ami using the following command:

**`$ packer build nginx.pkr.hcl`**

![packer build nginx](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/6145565c-9683-4c4b-bcc2-372ee36d8eb7)

**iii.** We build the ubuntu ami using the following command:

**`$ packer build ubuntu.pkr.hcl`**

![packer build ubuntu](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/20255b10-02d8-448d-b1d7-2ac7d47698c2)

**iv.** We build the web ami using the following command:

**`$ packer build web.pkr.hcl`**

![packer build web](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/b45cbebd-7cdd-4ade-aab1-1080306de1eb)

As shown in the image below we can verify in our AWS console that the AMIs were successfully built.

![ami images created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/6c9e6922-f930-4a75-b99d-1e68c09e4be4)

**2.** Update Terraform script with AMI IDs generated from Packer build.

We proceed to update the **`terraform.auto.tfvars`** with the AMI IDs.

![update autotfvars with ami ids](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/77ce78e2-73c1-4519-a282-89d9b8e5cded)

### Migrate `.tf` codes to Terraform Cloud

In this step, we shall be migrating our codes to Terraform Cloud and manage our AWS infrastructure from there:

**1.** Create a Terraform Cloud account

We navigate to the [terraform cloud homepage](https://app.terraform.io/signup/account), create a new account and verify our email address.

![terraform cloud homepage](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/44d3a030-328b-4722-8833-b44b03232084)

**2.** Create an organization

We select "Create Organization", we choose a name for our organization and create it.

![create organization](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/63cef564-1e90-4bb7-bcb4-570a7062b72c)

**3.** Configure a workspace

Here, we decide to use **`version control workflow`** as it is the most common and recommended way to run Terraform commands triggered from our git repository.

![create workspace](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/3e181484-a542-493b-94bf-1ebf554b8eff)

**i.** We create a new repository in our GitHub and we call it **`qb-terraform-cloud`**, then we push our Terraform codes developed in the previous projects to the repository.

![create new repo and push terraform code](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c894a7d7-2e73-486c-add9-6210575fd819)

![create new repo and push terraform code 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/ae0af401-b2b7-4d92-b32b-f7876a4bc620)

![repository for terraform cloud](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/9585a6c8-da5c-48a2-bdfb-9f9b91396230)

**ii.** We choose `version control workflow` and we are subsequently prompted to connect our version control system (VCS) which is our GitHub account to our workspace - we follow the prompt and add our newly created repository to the workspace.

![connect to VCS](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/ea3977ad-9d0a-4dd1-bd61-e2e15c3c15a6)

![choose repository](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/ab1bdaae-4561-4ccc-a6d9-d0df4bfeb2d4)

**iii.** We move on to "Configure settings", provide a description for our workspace and we leave all the rest settings as default, then we click "Create workspace".

![complete create workspace](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/253196d6-db4f-4adb-b7b9-55f09b6309da)

**4.** Configure variables

Terraform Cloud supports two types of workspace variables: Environment variables and Terraform variables. Either type can be marked as sensitive, which prevents them from being displayed in the Terraform Cloud web UI and makes them write-only.

![workspace variables](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/9cbb95b4-3763-4b0e-8882-a63f2534eb2e)

**i.** Terraform variables are the variables we have declared in our configuration. We however do not need to specify these variables in the Terraform Cloud UI since Terraform cloud can also load the default values from our **`.auto.tfvars`** file in the configuration.

![terraform autovars](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/9b70982b-bb09-475c-95dd-e26d1b560a80)

**ii.** Environment variables are variables we need to set to be used in the Terraform runtime environment. Here we set two environment variables: **`AWS_ACCESS_KEY_ID`** and **`AWS_SECRET_ACCESS_KEY`**, and we set the values that we used in [Project 16](https://expert-pbl.darey.io/en/latest/project16.html). These credentials will be used to provision our AWS infrastructure by Terraform Cloud.

![aws access key and secret key environment variable](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/5160f29e-8be6-44ce-8c6d-769e0d7ea770)

After setting these 2 environment variables as shown in the image above, our Terraform Cloud is all set to apply the codes from GitHub and create all necessary AWS resources. However, we make sure to update the code in **`backend.tf`** with the name of our Terraform Cloud organization and workspace.

![backend terraform cloud organisation and workspace names](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/bf189825-65de-4b64-85dc-4f8d858e63b3)

**5.** Push changes to remote repository and run Terraform Script

Next, we push the changes we made in the terraform code on our local machine to the github remote repository we created earlier for terraform cloud.

![push changes to remote](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/51612c6b-b8d7-450a-beff-a97071763af4)

Whenever we push updated code from our local machine to our github repository **`qb-terraform-cloud`**, the version control functionalities of github kicks in and triggers terraform cloud to automatically create a plan as shown below:



We proceed to click on Confim and apply. Then a dialogue comes up which asks us to add a comment to explain the action. We type in a comment and then we click on Confirm plan.




Subsequently terraform cloud starts the apply process and creates our infrastructure in AWS.


On completion of the apply process as shown in the image above we navigate to the AWS console to check on our Target groups.


Here, as seen in the image above, we notice that the instances in the target groups are unhealthy. This is because we are yet to have the instances properly configured.

To fix this, since it is the listeners that route traffic to the target groups which contain our instances, we navigate back to our terraform code and we comment out the listeners in the **`alb.tf`** file. We choose to do this since we will be running into a lot of errors if we attempt to configure the instances with Ansible.


To ensure that the autoscaling group does not spin up instances to the load balancer, we also need to comment it out. So we navigate to the **`asg-bastion-nginx.tf`** and **`asg-webserver.tf`** files to implement this.

Next, we push the changes to GitHub and then we apply the plan in Terraform Cloud.

As we can see in the image above the apply run was successfully completed. We proceed to navigate to our AWS console and as we can see in the image below, there are no more targets registered to the target groups.

And also when we check our Loadbalancers we can see that there are no listeners attached.




**6.** update ansible script with values from terraform output

To do this, we do not update the ansible script on our local machine. Rather, we SSH into the Bastion host and clone down the ansible script from our repository then we go ahead to make our changes.

But before we proceed, we will need to set up SSH Agent on our windowss machine. We proceed to do this with the help of the [official windows openssh key management documentation](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement) and we execute the following commands on Windows Powershell.

```
# By default the ssh-agent service is disabled. Configure it to start automatically.
# Make sure you're running as an Administrator.
Get-Service ssh-agent | Set-Service -StartupType Automatic

# Start the service
Start-Service ssh-agent

# This should return a status of Running
Get-Service ssh-agent

# Now load your key files into ssh-agent
ssh-add <user-key>

# Ensure Key has been added
ssh-add -l
```


update dns name for ialb in nginx reverse proxy
copy rds endpoint and update for wordpress and tooling
Update password and username for wordpress and tooling
copy and update efs access point ID and file system ID for wordpress and tooling
update roles path in ansible.cfg file export ANSIBLE_CONFIG=/home/ec2-user/ansible-deploy-pbl-19/roles


**7.** Run ansible Playbook

ansible-playbook -i inventory/aws_ec2.yml playbooks/site.yml


check the website




 


