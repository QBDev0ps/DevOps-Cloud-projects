## DOCUMENTATION FOR DEVOPS TOOLING WEBSITE SOLUTION

In the [previous project](https://github.com/QBDev0ps/DevOps-Cloud-projects/blob/main/Project9.md), we implemented a WordPress solution that is ready to be filled with content and can be used as a blog or a fully fledged website. Subsequently, in this project, we will be adding some more value to our solutions by implementing a comprehensive DevOps tooling website solution that will integrate various tools and technologies to create a unified platform that enhances collaboration, automation and efficiency for software development and operations teams.

### <br>Introduction to DevOps Tooling Website Solution<br/>

The goal of this project is to build a tooling website solution which will enable easy access to DevOps tools within the corporate organisation. The solution we will implement will consist of the following components:

**1.** **Infrastructure**: Amazon Web Services (AWS)

**2.** **Web Server**: Red Hat Enterprise Linux 8

**3.** **Database Server**: Ubuntu 20.04 + MySQL

**4.** **Storage Server**: Enterprise Linux 8 + NFS Server

**5.** **Programming Language**: PHP

**6.** **Code Repository**: GitHub

Based on the infrastructure to be used and the goal of this project, it shall consist of three parts:

**1.** Configure Network File Server (NFS) Server with storage infastructure.

**2.** Configure Database Server.

**3.** Configure Web Servers.

As shown in the diagram below, we shall implement a solution where three stateless Web Servers will share a common database and also access the same files using Network File Sytem (NFS) as a shared file storage. Even though the NFS server might be located on a completely separate hardware – for the Web Servers it would look like a local file system from where they can serve the same files.

![3 TIER Tooling-Website-Infrastructure](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/61a271c9-4efd-45ff-a71e-94e62340faf8)

### <br>Configure Network File Server (NFS) Server with storage infastructure<br/>

To begin our project we need to deploy and configure LVM for our Linux based NFS server. We do this by implementing the following steps:


#### <br>Step 1: Provision EC2 Instance<br/>

We begin by spinning up an EC2 Instance of Red Hat Linux that will serve as our Web Server: We launch our EC2 instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance) 

**i.** We open the AWS console and click on **"EC2"**, then we scroll up and click on **"Launch Instance"**.

![launch EC2 instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d331142c-a425-485d-9338-5e8f21d2a37d)

**ii.** Under **Name and tags**, we provide a unique name for our web server.

![name and tags](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a121b805-be6a-4e01-8be2-698e194d8909)
  
**iii.** From the **Applications and Amazon Machine Image (AMI Image)** tab, we ensure we select the free tier eligible version of Red Hat Enterprise Linux 8 (HVM).

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

![Volume Settings](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fc5ac363-e6f6-45af-9e5a-19cf3342a6ed)

**iv.** We repeat **i-iii** above twice to create two more Elastic Block Store (EBS) Volumes.

#### <br>Step 3: Attach EBS Volumes to EC2 Web Server Instance<br/>

After creating the 3 EBS volumes we proceed to attach them to our EC2 Web Server instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-attaching-volume.html)

**i.** We open the AWS console and click on **"EC2"**, then we scroll down in the navigation pane and click on **"Volumes"** under **"Elastic Block Store"**.

![volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2befe33d-7849-4677-9252-fd7d4af93b26)

**ii.** Select the Volume you wish to attach under the Volumes list then right click and select **"Attach Volume"**.

![Attach Volume](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/32ec447b-5d02-4328-8582-528573bb3667)

**iii.** In the **"Attach Volume"** page, under **"Instance"**, we select our EC2 Linux Web Server Instance, under **"Device name"** the name can be changed from **/dev/sdf** through **/dev/sdp** depending on preferences, then we click on **"Attach Volume"** at the bottom right corner of the page.

![Attach Volume 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/36039f97-ae1c-4eb8-85ae-8239b16f5e32)

**iv.** We repeat **i-iii** above twice to attach the remaining two Elastic Block Store (EBS) Volumes to our EC2 Linux Web Server Instance.

#### <br>Step 4: Connect to the Web Server via the Terminal using the SSH Client<br/>

After we have provisioned our server and we have created and attached our EBS volumes, we must next connect to the web server via an SSH client. This will enable us to subsequently be able to run commands and begin configuration on our web server. We carry this out by doing the following:

**i.** Download and Install an SSH client: Download and install [Termius](https://www.termius.com/download/windows) or Download and install [git](https://git-scm.com/downloads) (the ssh client - git bash will be packaged with the git installation)

**ii.** Establish connection with the EC2 instance: We connect to our EC2 instance via our Termius SSH client by following [these instructions:](https://dev.to/aws-builders/how-to-connect-your-ec2-linux-instance-with-termius-5209)

#### <br>Step 5: Update Webserver and Inspect Attached Block Devices<br/>

**i.** After connecting to our server we must first update all installed packages and their dependencies before commencing configuration. We do this by executing the following command: 

**`$ sudo yum update -y`**

![sudo yum update](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/67ea0933-1e20-4e1d-aede-a1e29e51fe67)

**ii.** Subsequently, we inspect what block devices are attached to the web server with the following command:

**`$ lsblk`**

As can be seen in the image below, our EBS volumes are shown using the **`nvme`** naming convention rather than **`xvdf`**. This is because our block devices are connected through an NVMe port which uses the nvme driver on Linux. It should also be noted that EBS volumes are typically exposed as NVMe block devices on instances built on the Nitro System. The device names are **/dev/nvme0n1**, **/dev/nvme1n1**, and so on. The Nitro System is a collection of hardware and software components built by AWS that enable high performance, high availability, and high security.

![lsblk](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b312e15c-f0cc-4813-9f6c-e057471f90e9)

The **`lsblk`** command reveals that **/dev/nvme0n1** is the default storage device attached to our EC2 instance and has four partitions **/dev/nvme0n1p1-4** with **/dev/nvme0n1p4** mounted as the root device. The block devices we created are listed as **/dev/nvme1n1**, **/dev/nvme2n1** and **/dev/nvme3n1** and as can be seen, they have no mount points because they are not yet mounted.

**iii.** To see all mounts and free space on our Web Server, we run the command below.

**`$ df -h`**

![df -h](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/63742891-1c19-4932-8d8a-d5f4b98c7876)

**iv.** We also proceed to check the **/dev/** directory with the following command: 

**`$ ls /dev/`**

#### <br>Step 6: Partition Disks and Install lvm2 Package<br/>
![ls dev folder](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/958f72cb-9144-4a5d-bcc2-f26e8ad15795)

As can be seeen in the above image, the executed command lists all Linux devices and we can see that our attached block devices  **/dev/nvme1n1**, **/dev/nvme2n1** and **/dev/nvme3n1** are listed.
