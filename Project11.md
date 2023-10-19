## DOCUMENTATION FOR ANSIBLE-AUTOMATE PROJECT

In this project, we will be working on automating routine IT operations with [Ansible Configuration Management](https://www.redhat.com/en/topics/automation/what-is-configuration-management#:~:text=Configuration%20management%20is%20a%20process,in%20a%20desired%2C%20consistent%20state.&text=Managing%20IT%20system%20configurations%20involves,building%20and%20maintaining%20those%20systems.). The project will show how deploying Ansible will help to simplify complex tasks and streamline IT infastructure. We will also work with Jenkins to configure and execute build jobs whilst writing code using declarative language such as **`YAML`**.
 

### <br>Introduction to Ansible Configuration Management<br/>

The goal of this project is to demonstrate Ansible's powerful automation capabilities. Ansible is an open source, command line software application that is used for automating IT operations such as deploying applications, managing configurations scaling infrastructure and other activities involving many repetitive tasks. Ansible's main strengths are simplicity and ease of use. It lets IT professionals quickly and easily deploy multi tier apps. Rather than needing to write lengthy code to automate our systems, we simply list the tasks that require automation by writing a Playbook and Ansible figures out how to get our systems to the state we need them to be in. It is also important to note Ansible has a strong focus on security and reliability and as such it has very minimal moving parts. A great exapmle of this is that Ansible is agentless. This means that the devices or infrastructure it monitors do not require any proprietary software agent to be installed on them beforehand. Ansible has two major types of files: 

**1.** The Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate.
   
**2.** Ansible Playbooks are lists of tasks that are automatically executed for the specified inventory or groups of hosts. One or more Ansible tasks can be combined to make a play — an ordered grouping of tasks mapped to specific hosts—and tasks are executed in the order in which they are written.

![0_sMSfIbPO8mH299to](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4134dac2-80be-4957-bdf5-9357889f51f5)

#### Ansible as a Jump Server

A [Jump Server](https://en.wikipedia.org/wiki/Jump_server) (sometimes also referred as [Bastion Host](https://en.wikipedia.org/wiki/Bastion_host)) is an intermediary server through which access to internal network can be provided. If you think about the current architecture we are working on, ideally, the webservers would be inside a secured network which cannot be reached directly from the Internet. That means, even DevOps engineers cannot SSH into the Web servers directly and can only access it through a Jump Server – it provides better security and reduces [attack surface](https://en.wikipedia.org/wiki/Attack_surface).

On the diagram below, the Virtual Private Network (VPC) is divided into [two subnets](https://docs.aws.amazon.com/vpc/latest/userguide/how-it-works.html) – Public subnet has public IP addresses and Private subnet is only reachable by private IP addresses.

![bastion host](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7edfe278-2217-45c4-b35d-ac05bfd8ff47)

Further on in our future projects, we will implement a proper Bastion Host. But for now, we will make do with developing Ansible scripts to simulate the use of a Jump Box/Bastion Host to access our Web Servers. Therefore, based on the infrastructure to be used and the goal of this project, it shall consist of three parts:

**1.** Install and Configure Jenkins on EC2 Instance.

**2.** Install and Configure Ansible to act as a Jump Server/Bastion Host.

**3.** Create a Simple Ansible Playbook to Automate Server Configuration.
