## DOCUMENTATION FOR IMPLEMENTING WORDPRESS WEBSITE WITH LVM STORAGE MANAGEMENT

This project demonstrates how to build and manage a scalable WordPress website using AWS EC2 and LVM (Logical Volume Management) storage. The project will delve into the intricacies of LVM storage management on Ubuntu such as the processes of creating logical volumes, managing disk space and accomodating changing storage requirements. And it will also cover performance optimization techniques and best practices for managing and securing WordPress installation in the AWS cloud.

### <br>Introduction to Implementing WordPress Website with LVM Storage Management<br/>

This project consists of two parts:

1. Configure storage infastructure on two Linux OS based (Web and Database) servers.
   
2. Deploy the Web and Database tiers of a basic web solution by installing WordPress and connecting it to a remote MySQL database server.

WordPress is a free and open-source content management system written in **PHP** and paired with **MySQL** or **MariaDB** as its backend Relational Database Management System (RDBMS)

In terms of deploying tiers of a web solution, the **Three-tier Architecture** is generally used when implementing a web or mobile solution.

The **Three-tier Architecture** is a client-server architecture pattern that consists of three seperate layers.

![3 tier](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/1ec51fa2-569f-4155-8310-1a24f9e80ef6)

**1. Presentation Layer (PL)**: This is the user interface and communication layer of the application, where the end user interacts with the application. This can for instance be the client server or browser on your machine. Its main purpose is to display information to and collect information from the user.

**2. Business Layer (BL)**: Also known as the Application Layer or the Logic Layer. This is the heart of the application. In this layer, information collected in the presentation tier is processed - sometimes against other information in the data layer - using business logic, a specific set of business rules.

**3. Data Access Layer (DAL)**: Also known as the Management Layeror Database Layer. This is where the information processed by the application is stored and managed. This can be a Database server such as PostgreSQL, MySQL, MariaDB etc. or  a File System Server such as FTP Server, NFS Server etc.

In the execution of this project, our three tier architecture shall be:

1. A Laptop or PC to serve as a Client.
   
2. An EC2 Linux Server to serve as a Web Server that will host our WordPress website.
   
3. An EC2 Linux Server to serve as our Database Server.

### <br>Implementing LVM on Linux Servers (Web and Database Servers)<br/>

To begin our project we need to deploy and configure LVM for our Linux based Web and Database servers. We do this by implementing the following steps:

#### <br>Step 1: Provision EC2 Instance<br/>

We begin by spinning up an EC2 Instance of Red Hat Linux that will serve as our Web Server: We launch our EC2 instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance) 

**i.** We open the AWS console and click on **"EC2"**, then we scroll up and click on **"Launch Instance"**.

![launch EC2 instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d331142c-a425-485d-9338-5e8f21d2a37d)

**ii.** Under **Name and tags**, we provide a unique name for our web server.

![name and tags](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a121b805-be6a-4e01-8be2-698e194d8909)
  
**iii.** From the **Applications and Amazon Machine Image (AMI Image)** tab, we ensure we select the free tier eligible version of Red Hat Enterprise Linux 9 (HVM).

![Application and OS Images](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/86ef4789-366a-4319-b4f2-709f305fa7f1)

**iv.** Under **Key pair**, we select an existing one. (You can create a new key pair if you do not have one and the same key pair can be used for all the instances that will be provisioned in this project.)

![Key Pair](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/65facdd1-4be3-4ec5-aac4-aadd74821653)
  
**v.** And then finally, we click on **"Launch Instance"**
  
![Launch Instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ed623db9-831f-4c86-bc46-f0e7201c18f6)

#### <br>Step 2: Create EBS Volumes<br/>

The next course of action is to create 3 EBS volumes of 10GB each in the same Availability Zone as our EC2 Linux Web Server instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-creating-volume.html) 

**i.** We open the AWS console and click on **"EC2"**, then we scroll down in the navigation pane and click on **"Volumes"** under **"Elastic Block Store"**.

![volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2befe33d-7849-4677-9252-fd7d4af93b26)

**ii.** Click on **"Create Volume"** at the top right hand corner of the **"Volumes"** page.

![Create Volume](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9e710712-916e-4543-8b37-b2dc3a357c12)

**iii.** On the **"Volume Settings"** page, we select the **"Volume Type"**, set **"Size"** as 10GB, we select the **"Availability Zone"** of our EC2 Web Server and the we click on **"Create Volume"** at the bottom right corner of the page. 

![Create Volume](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b37c54cf-22c0-4a45-827a-e58b4b7651b8)

**iv.** We repeat **i-iii** above twice to create two more Elastic Block Store (EBS) Volumes.

#### <br>Step 3: Attach EBS Volumes to EC2 Web Server Instance<br/>

After creating the 3 EBS volumes we proceed to attach them to our EC2 Web Server instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-attaching-volume.html)

**i.** We open the AWS console and click on **"EC2"**, then we scroll down in the navigation pane and click on **"Volumes"** under **"Elastic Block Store"**.

![volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2befe33d-7849-4677-9252-fd7d4af93b26)

**ii.** Select the Volume you wish to attach under the Volumes list then right click and select **"Attach Volume"**.

![Attach Volume](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/32ec447b-5d02-4328-8582-528573bb3667)

**iii.** In the **"Attach Volume"** page, under **"Instance"**, we select our EC2 Linux Web Server Instance, under **"Device name"** the name can be changed from /dev/sdf through /dev/sdp depending on preferences, then we click on **"Attach Volume"** at the bottom right corner of the page.

![Attach Volume 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/36039f97-ae1c-4eb8-85ae-8239b16f5e32)

**iv.** We repeat **i-iii** above twice to attach the remaining two Elastic Block Store (EBS) Volumes to our EC2 Linux Web Server Instance.

#### <br>Step 4: Connect to the Webserver via the Terminal using the SSH Client<br/>

After we have provisioned our server and we have created and attached our EBS volumes, we must next connect to the web server via an SSH client. This will enable us to subsequently be able to run commands and begin configuration on our web server. We carry this out by doing the following:

**i.** Download and Install an SSH client: Download and install [Termius](https://www.termius.com/download/windows) or Download and install [git](https://git-scm.com/downloads) (the ssh client - git bash will be packaged with the git installation)

**ii.** Establish connection with the EC2 instance: We connect to our EC2 instance via our Termius SSH client by following [these instructions:](https://dev.to/aws-builders/how-to-connect-your-ec2-linux-instance-with-termius-5209)

#### <br>Step 5:Update Webserver and Inspect Attached Block Devices<br/>

After connecting to our server we must first update all installed packages and their dependencies before commecing configuration. We do this by executing the following command: 

**`$ sudo yum update -y`**

![sudo yum update](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/67ea0933-1e20-4e1d-aede-a1e29e51fe67)

Subsequently, we inspect what block devices are attached to the web server with the following command:

**`$ lsblk`**

As can be seen in the image below, our EBS volumes are shown using the **`nvme`** naming convention rather than **`xvdf`**. This is because our block devices are connected through an NVMe port which uses the nvme driver on Linux. It should also be noted that EBS volumes are typically exposed as NVMe block devices on instances built on the Nitro System. The device names are /dev/nvme0n1, /dev/nvme1n1, and so on. The Nitro System is a collection of hardware and software components built by AWS that enable high performance, high availability, and high security.

![lsblk](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b312e15c-f0cc-4813-9f6c-e057471f90e9)

The **`lsblk`** command reveals that **/dev/nvme0n1** is the default storage devivce attached to our EC2 instance and has four partitions **/dev/nvme0n1p1-4** with **/dev/nvme0n1p4** mounted as the root device. The block devices we created are listed as **/dev/nvme1n1**, **/dev/nvme2n1** and **/dev/nvme3n1** and as can be seen, they have no mount points because they are not yet mounted.

To see all mounts and free space on our Web Server, we run the command below.

**`$ df -h`**

![df -h](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/63742891-1c19-4932-8d8a-d5f4b98c7876)

We also proceed to check the **/dev/** directory with the following command: 

**`$ ls /dev/`**

![ls dev folder](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/958f72cb-9144-4a5d-bcc2-f26e8ad15795)

As can be seeen in the above image, the executed command lists all Linux devices and we can see that our attached block devices  **/dev/nvme1n1**, **/dev/nvme2n1** and **/dev/nvme3n1** are listed.

We also obtain more important information such as name, serial number, size and LBA format about all the NVMe devices attached to our machine. However, a prerequisite for this is to install the NVMe command line package, **`nvme-cli`** by executing the following command:

**`$ sudo dnf install nvme-cli -y`**

![nvme cli](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/48eb3eb1-8794-4c45-84ec-97364bfd9089)

Then we subsequently run the command below to see additional information about the EBS volumes attached to our Linux Machine:

**`$ sudo nvme list`**

![nvme list](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/633d4425-ed2c-4d0f-870f-b88f77e61023)

#### <br>Step 6:Partition Disks and Install lvm2 Package<br/>

In this step, we proceed to create a single partition on each of the 3 Disks using the **`gdisk`** utility. We partition **/dev/nvme1n1** by executing the following command: 

**`$ sudo gdisk /dev/nvme1n1`**

As shown in the output image below, we enter **`?`** to list out all the available commands in the gdisk console, we enter the **`p`** command to provide information about available space in hard disk to create a new partition. Subsequently we enter the **`n`** command to create a new partition, we follow through with the prompts and then we enter **`w`** to write the partition table to disk and exit the gdisk console. When the system requests for input with **`Do you want to proceed? (Y/N):`**,  we enter **`y`** to confirm our earlier operation.

![gdisk commands](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a3aa2185-2005-4604-9af3-7ff03a426b5f)

We repeat the same process above to create a single partition on **/dev/nvme2n1** and **/dev/nvme3n1**.

Afterwards, we run the **`lsblk`** utility to view the newly configured partition on each of the 3 disks.

![gdisk partition](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/47df48a1-9c8b-4dbc-b1d9-1d0a8a2ff9a0)

As can be seen in the image above, we have our newly created partitions listed as **`nvme1n1p1`**, **`nvme2n1p1`** and **`nvme3n1p1`**
