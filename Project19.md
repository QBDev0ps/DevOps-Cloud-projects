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





 



