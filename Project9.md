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

### <br>Implementing LVM on Linux Web Server<br/>

To begin our project we need to deploy and configure LVM for our Linux based Web server. We do this by implementing the following steps:

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

![Volume Settings](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fc5ac363-e6f6-45af-9e5a-19cf3342a6ed)

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

As can be seen in the image below, our EBS volumes are shown using the **`nvme`** naming convention rather than **`xvdf`**. This is because our block devices are connected through an NVMe port which uses the nvme driver on Linux. It should also be noted that EBS volumes are typically exposed as NVMe block devices on instances built on the Nitro System. The device names are /dev/nvme0n1, /dev/nvme1n1, and so on. The Nitro System is a collection of hardware and software components built by AWS that enable high performance, high availability, and high security.

![lsblk](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b312e15c-f0cc-4813-9f6c-e057471f90e9)

The **`lsblk`** command reveals that **/dev/nvme0n1** is the default storage device attached to our EC2 instance and has four partitions **/dev/nvme0n1p1-4** with **/dev/nvme0n1p4** mounted as the root device. The block devices we created are listed as **/dev/nvme1n1**, **/dev/nvme2n1** and **/dev/nvme3n1** and as can be seen, they have no mount points because they are not yet mounted.

**iii.** To see all mounts and free space on our Web Server, we run the command below.

**`$ df -h`**

![df -h](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/63742891-1c19-4932-8d8a-d5f4b98c7876)

**iv.** We also proceed to check the **/dev/** directory with the following command: 

**`$ ls /dev/`**

![ls dev folder](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/958f72cb-9144-4a5d-bcc2-f26e8ad15795)

As can be seeen in the above image, the executed command lists all Linux devices and we can see that our attached block devices  **/dev/nvme1n1**, **/dev/nvme2n1** and **/dev/nvme3n1** are listed.

**v.** We can also obtain more important information such as name, serial number, size and LBA format about all the NVMe devices attached to our machine. However, a prerequisite for this is to install the NVMe command line package, **`nvme-cli`** by executing the following command:

**`$ sudo dnf install nvme-cli -y`**

![nvme cli](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/48eb3eb1-8794-4c45-84ec-97364bfd9089)

**vi.** Then we subsequently run the command below to see additional information about the EBS volumes attached to our Linux Machine:

**`$ sudo nvme list`**

![nvme list](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/633d4425-ed2c-4d0f-870f-b88f77e61023)

#### <br>Step 6: Partition Disks and Install lvm2 Package<br/>

**i.** In this step, we proceed to create a single partition on each of the three (3) Disks using the **`gdisk`** utility. We partition **/dev/nvme1n1** by executing the following command: 

**`$ sudo gdisk /dev/nvme1n1`**

As shown in the output image below, we enter **`?`** to list out all the available commands in the gdisk console, we enter the **`p`** command to provide information about available space in hard disk to create a new partition. Subsequently we enter the **`n`** command to create a new partition, we follow through with the prompts and then we enter **`w`** to write the partition table to disk and exit the gdisk console. When the system requests for input with **`Do you want to proceed? (Y/N):`**,  we enter **`y`** to confirm our earlier operation.

![gdisk commands](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a3aa2185-2005-4604-9af3-7ff03a426b5f)

We repeat the same process above to create a single partition on **/dev/nvme2n1** and **/dev/nvme3n1**.

**ii.** Afterwards, we run the **`lsblk`** utility to view the newly configured partition on each of the three (3) disks.

![gdisk partition](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/47df48a1-9c8b-4dbc-b1d9-1d0a8a2ff9a0)

As can be seen in the image above, we have our newly configured partitions on each of the three (3) disks listed as **`nvme1n1p1`**, **`nvme2n1p1`** and **`nvme3n1p1`**

**iii.** The next course of action is to install the **`lvm2`** package using the following command:

**`$ sudo yum install lvm2 -y`**

![lvm2 installation](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/cc915321-9fab-4aaa-9d95-66c5b13dbdcd)

**iv.** Then we run the command below to check for available partitions:

**`$ sudo lvmdiskscan`**

![sudo lvmdiskscan](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/caa1ef9d-5f06-42d2-8cc8-815dbad6037b)

#### <br>Step 7: Create Physical and Logical Volumes<br/>

**i.** For the next step, we use the **`pvcreate`** utility to mark each of our three (3) partitioned disks as physical volumes (PVs) to be used by LVM:

```
$ sudo pvcreate /dev/nvme1n1p1
$ sudo pvcreate /dev/nvme2n1p1
$ sudo pvcreate /dev/nvme3n1p1
```

![creating physical volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4071f3c7-0ecf-454d-83e2-7840f50bd764)

**ii.** Then afterwards, we verify theat the physical volumes (PVs) have been created by executing the command below:

**`$ sudo pvs`**

![sudo pvs](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/cc0aacd8-ac5f-415f-acfb-9a4d96c290eb)

**iii.** After this, we proceed to use the **`vgcreate`** utility to add all three (3) physical volumes (PVs) to a volume group (VG) that we will be naming **webdata-vg**

**`$ sudo vgcreate webdata-vg /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1`**

![create volume group](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/658856e7-46dc-4644-829a-43ad07cd3296)

**iv.** And then we confirm that our volume group (VG) has been created by successfully executing the following command:

**`$ sudo vgs`**

![sudo vgs](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7c5a00c5-b2ce-477c-b8f2-c9cfd963f6ab)

**v.** The next step is to create two (2) logical volumes (LVs) **apps-lv** and **logs-lv** using the **`lvcreate`** utility. For **apps-lv**, we will be using half of the PV size and it will be used to store data for the website while for **logs-lv** we will be using the remaining space left of the PV size and it will be used to store data for logs.

```
$ sudo lvcreate -n apps-lv -L 14G webdata-vg
$ sudo lvcreate -n logs-lv -L 14G webdata-vg
```

![create logical volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/1f653f4e-1f3b-464c-83be-d0526683f467)

**vi.** And then we confirm that our logical volumes have been created by successfully executing the following command:

**`$ sudo lvs`**

![sudo lvs](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d14ad131-29a7-42b9-950c-e9dda18377e8)

**vii.** Then we verify our entire setup of Volume Group (VG), Physical Volumes (PV) and Logical Volumes (LV) with the following commands:

**`$ sudo vgdisplay -v`**

![sudo vgdisplay](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5168c6ad-e71b-4d9f-868c-ec2728ec95ed)

**`$ sudo lsblk`**

![verify entire setup](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/03a9e4c8-6ac9-486a-a81c-6626f91e5bbc)

**viii.** To complete the process, we use **`mkfs.ext4`** to format the logical volumes (LVs) with **ext4** filesystem.

```
$ sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
$ sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```

![format logical volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/dfc6a0a3-51e4-4187-80c3-acec38b92412)

#### <br>Step 8: Create Directories and Mount on Logical Volumes<br/>

In this step, we need to create the directory to hold our website files and then another directory to store backup of log data after which we will respectively mount these directories on the created logical volumes **apps-lv** and **logs-lv**.

**i.** We use the following command to create the **/var/www/html** directory to store our website application files:

**`$ sudo mkdir -p /var/www/html`**

**ii.** We use the command below to create the **/home/recovery/logs** directory to store backup of log data:

**`$ sudo mkdir -p /home/recovery/logs`**

**iii.** Then we execute the following command to mount **/var/www/html/** on **apps-lv** logical volume:

**`$ sudo mount /dev/webdata-vg/apps-lv /var/www/html/**

**iv.** **/var/log** is the default directory where Linux stores all log files. This is the directory that we need to mount on our **logs-lv** volume. However, mounting this directory will delete all the files contained in it so before we carry out this action, we need to use the **`rsync`** utility to backup all the files in the log directory **/var/log** into the **/home/recovery/logs** directory we created. We do this by executing the command below:

**`$ sudo rsync -av /var/log/. /home/recovery/logs/`**

![backup log files](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/1c8fabac-607a-49fd-a538-8abdb294d780)

**v.** Then we enter the command below to mount **/var/log** on **apps-lv** logical volume:

**`$ sudo mount /dev/webdata-vg/logs-lv /var/log`**

**vi.** Afterwards, we restore the log files back into the **/var/log** directory.

**`$ sudo rsync -av /home/recovery/logs/. /var/log`**

![restore log files](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/da7a0e22-0a8f-4729-8621-f9665e9d6167)

**vii.** The next step is to use the universally unique identifier (UUID) of the device to update the **/etc/fstab** file so that the mount configuration will persist after the restart of the server. We check the UUID of the device by entering the command below:

**`$ sudo blkid`**

![uuid](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e8ce173d-4db6-4f0c-b7bb-b0b56349af4d)

**viii.** We copy the UUID as shown in the above image and we open the **/etc/fstab** file with the following command:

**`$ sudo vi /etc/fstab`**

**ix.** We paste in the copied UUID whilst removing the leading and ending quotes and update the **/etc/fstab** file as shown in the image below:

![mounts for wordpress server](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fc378217-afc5-41ea-ac3a-306d2390f186)

**x.** We test our mount configuration with the following command:

**`$ sudo mount -a`**

**xi.** Then we reload the daemon with the command below:

**`$ sudo systemctl daemon-reload`**

**xii.** To complete our configuration process, we verify our entire setup by executing the following command:

**`$ df -h`**

![df -h final output](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5ca686a1-58f5-47e7-a41f-cffcab726981)

The output must look like what we have in the image above.

### <br>Implementing LVM on Linux Database Server<br/>

The next phase of this project involves preparing the database server. We shall launch a second RedHat EC2 instance that will have a role as our **‘DB Server’**. We shall be repeating the same steps as for the Web Server, but instead of **`apps-lv`** we will create **`db-lv`** and mount it to **`/db`** directory instead of **`/var/www/html/`**.

#### <br>Step 1: Provision EC2 Instance<br/>

We begin by spinning up an EC2 Instance of Red Hat Linux that will serve as our Database Server: We launch our EC2 instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance) 

**i.** We open the AWS console and click on **"EC2"**, then we scroll up and click on **"Launch Instance"**.

![launch EC2 instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d331142c-a425-485d-9338-5e8f21d2a37d)

**ii.** Under **Name and tags**, we provide a unique name for our Database server.

![Name and tags DB server](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4a139fcb-1ace-478f-abb9-7e27c3e48704)
  
**iii.** From the **Applications and Amazon Machine Image (AMI Image)** tab, we ensure we select the free tier eligible version of Red Hat Enterprise Linux 9 (HVM).

![Application and OS Images](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/86ef4789-366a-4319-b4f2-709f305fa7f1)

**iv.** Under **Key pair**, we select an existing one. (You can create a new key pair if you do not have one and the same key pair can be used for all the instances that will be provisioned in this project.)

![Key Pair](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/65facdd1-4be3-4ec5-aac4-aadd74821653)
  
**v.** And then finally, we click on **"Launch Instance"**
  
![Launch Instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ed623db9-831f-4c86-bc46-f0e7201c18f6)

#### <br>Step 2: Create EBS Volumes<br/>

The next course of action is to create 3 EBS volumes of 10GB each in the same Availability Zone as our EC2 Linux Database Server instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-creating-volume.html) 

**i.** We open the AWS console and click on **"EC2"**, then we scroll down in the navigation pane and click on **"Volumes"** under **"Elastic Block Store"**.

![volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2befe33d-7849-4677-9252-fd7d4af93b26)

**ii.** Click on **"Create Volume"** at the top right hand corner of the **"Volumes"** page.

![Create Volume](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9e710712-916e-4543-8b37-b2dc3a357c12)

**iii.** On the **"Volume Settings"** page, we select the **"Volume Type"**, set **"Size"** as 10GB, we select the **"Availability Zone"** of our EC2 Database Server and the we click on **"Create Volume"** at the bottom right corner of the page. 

![Volume Settings](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fc5ac363-e6f6-45af-9e5a-19cf3342a6ed)

**iv.** We repeat **i-iii** above twice to create two more Elastic Block Store (EBS) Volumes.

#### <br>Step 3: Attach EBS Volumes to EC2 Database Server Instance<br/>

After creating the 3 EBS volumes we proceed to attach them to our EC2 Database Server instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-attaching-volume.html)

**i.** We open the AWS console and click on **"EC2"**, then we scroll down in the navigation pane and click on **"Volumes"** under **"Elastic Block Store"**.

![volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2befe33d-7849-4677-9252-fd7d4af93b26)

**ii.** Select the Volume you wish to attach under the Volumes list then right click and select **"Attach Volume"**.

![attach volume db server](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/391a7639-23ef-41f4-8689-4eba74456031)

**iii.** In the **"Attach Volume"** page, under **"Instance"**, we select our EC2 Linux Database Server Instance, under **"Device name"** the name can be changed from /dev/sdf through /dev/sdp depending on preferences, then we click on **"Attach Volume"** at the bottom right corner of the page.

![attach volume 2 DB server](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/6156c4d1-065d-4304-99b0-71b8f68f40ce)

**iv.** We repeat **i-iii** above twice to attach the remaining two Elastic Block Store (EBS) Volumes to our EC2 Linux Database Server Instance.

#### <br>Step 4: Connect to the Database Server via the Terminal using the SSH Client<br/>

After we have provisioned our server and we have created and attached our EBS volumes, we must next connect to the Database server via an SSH client. This will enable us to subsequently be able to run commands and begin configuration on our Database server. We carry this out by doing the following:

**i.** Download and Install an SSH client: Download and install [Termius](https://www.termius.com/download/windows) or Download and install [git](https://git-scm.com/downloads) (the ssh client - git bash will be packaged with the git installation)

**ii.** Establish connection with the EC2 instance: We connect to our EC2 instance via our Termius SSH client by following [these instructions:](https://dev.to/aws-builders/how-to-connect-your-ec2-linux-instance-with-termius-5209)

#### <br>Step 5: Update Database Server and Inspect Attached Block Devices<br/>

**i.** After connecting to our server we must first update all installed packages and their dependencies before commencing configuration. We do this by executing the following command: 

**`$ sudo yum update -y`**

![sudo yum update db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/6d9eed39-54cb-4243-b7bb-2a7bb08727ca)

**ii.** Subsequently, we inspect what block devices are attached to the Database server with the following command:

**`$ lsblk`**

As can be seen in the image below, our EBS volumes are shown using the **`nvme`** naming convention rather than **`xvdf`**. This is because our block devices are connected through an NVMe port which uses the nvme driver on Linux. It should also be noted that EBS volumes are typically exposed as NVMe block devices on instances built on the Nitro System. The device names are /dev/nvme0n1, /dev/nvme1n1, and so on. The Nitro System is a collection of hardware and software components built by AWS that enable high performance, high availability, and high security.

![lsblk db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/91272424-2c1f-47c5-9041-4f89a76e0ea1)

The **`lsblk`** command reveals that **/dev/nvme0n1** is the default storage device attached to our EC2 instance and has four partitions **/dev/nvme0n1p1-4** with **/dev/nvme0n1p4** mounted as the root device. The block devices we created are listed as **/dev/nvme1n1**, **/dev/nvme2n1** and **/dev/nvme3n1** and as can be seen, they have no mount points because they are not yet mounted.

**iii.** To see all mounts and free space on our Database Server, we run the command below.

**`$ df -h`**

![df-h db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2302ce7d-a7bd-45cd-b600-95cc1804436c)

**iv.** We also proceed to check the **/dev/** directory with the following command: 

**`$ ls /dev/`**

![ls dev folder db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/55cde77f-f5a7-4581-be52-1c3b61ce979f)

As can be seen in the above image, the executed command lists all Linux devices and we can see that our attached block devices  **/dev/nvme1n1**, **/dev/nvme2n1** and **/dev/nvme3n1** are listed.

**v.** We can also obtain more important information such as name, serial number, size and LBA format about all the NVMe devices attached to our machine. However, a prerequisite for this is to install the NVMe command line package, **`nvme-cli`** by executing the following command:

**`$ sudo dnf install nvme-cli -y`**

![sudo dnf install db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/874557d2-cd22-4753-905f-41d7780cc608)

**vi.** Then we subsequently run the command below to see additional information about the EBS volumes attached to our Linux Machine:

**`$ sudo nvme list`**

![sudo nvme list db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2a6be523-3dab-4d1f-848b-7a78e200b5fe)

#### <br>Step 6: Partition Disks and Install lvm2 Package<br/>

**i.** In this step, we proceed to create a single partition on each of the three (3) Disks using the **`gdisk`** utility. We partition **/dev/nvme1n1** by executing the following command: 

**`$ sudo gdisk /dev/nvme1n1`**

As shown in the output image below, we enter **`?`** to list out all the available commands in the gdisk console, we enter the **`p`** command to provide information about available space in hard disk to create a new partition. Subsequently we enter the **`n`** command to create a new partition, we follow through with the prompts and then we enter **`w`** to write the partition table to disk and exit the gdisk console. When the system requests for input with **`Do you want to proceed? (Y/N):`**,  we enter **`y`** to confirm our earlier operation.

![gdisk commands db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/161cbb61-3082-4c8b-98c6-e4394c1af797)

We repeat the same process above to create a single partition on **/dev/nvme2n1** and **/dev/nvme3n1**.

**ii.** Afterwards, we run the **`lsblk`** utility to view the newly configured partition on each of the three (3) disks.

![gdisk partition db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/0c080c79-3d60-4bfc-bf58-14702b18224f)

As can be seen in the image above, we have our newly configured partitions on each of the three (3) disks listed as **`nvme1n1p1`**, **`nvme2n1p1`** and **`nvme3n1p1`**

**iii.** The next course of action is to install the **`lvm2`** package using the following command:

**`$ sudo yum install lvm2 -y`**

![lvm2 installation db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b78d8cec-9596-48a9-903e-1d90269618b2)

**iv.** Then we run the command below to check for available partitions:

**`$ sudo lvmdiskscan`**

![sudo lvmdiskscan db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/578ac557-94ef-4091-ba56-5c6f1061557c)

#### <br>Step 7: Create Physical and Logical Volumes<br/>

**i.** For the next step, we use the **`pvcreate`** utility to mark each of our three (3) partitioned disks as physical volumes (PVs) to be used by LVM:

**`$ sudo pvcreate /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1`**

![creating physical volumes db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/846dab7e-6029-46b9-8c24-181c75609f91)

**ii.** Then afterwards, we verify theat the physical volumes (PVs) have been created by executing the command below:

**`$ sudo pvs`**

![sudo pvs db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/eb86fd81-9823-4244-aa23-0776c7f72a56)

**iii.** After this, we proceed to use the **`vgcreate`** utility to add all three (3) physical volumes (PVs) to a volume group (VG) that we will be naming **dbdata-vg**

**`$ sudo vgcreate dbdata-vg /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1`**

![create volume group db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/55905165-0d23-457c-9ef1-01aeb1c3e50c)

**iv.** And then we confirm that our volume group (VG) has been created by successfully executing the following command:

**`$ sudo vgs`**

![sudo vgs db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5214ffc4-682a-4689-a772-ee96158c0c83)

**v.** The next step is to create two (2) logical volumes (LVs) **db-lv** and **logs-lv** using the **`lvcreate`** utility. For **db-lv**, we will be using half of the PV size and it will be used for database storage while for **logs-lv** we will be using the remaining space left of the PV size and it will be used to store data for logs.

```
$ sudo lvcreate -n db-lv -L 14G dbdata-vg
$ sudo lvcreate -n logs-lv -L 14G dbdata-vg
```

![create logical volumes db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/131a20e5-2b3b-407b-b59c-537e1888642d)

**vi.** And then we confirm that our logical volumes have been created by successfully executing the following command:

**`$ sudo lvs`**

![sudo lvs db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/56ba9b5e-4487-465e-92bd-68d3251524e0)

**vii.** Then we verify our entire setup of Volume Group (VG), Physical Volumes (PV) and Logical Volumes (LV) with the following commands:

**`$ sudo vgdisplay -v`**

![sudo vgdisplay db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/55f8718d-2de8-44c2-a57d-afdf68310a9d)

**`$ sudo lsblk`**

![verify entire setup db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/257574ab-ca72-4cf9-a17a-82f252bedd05)

**viii.** To complete the process, we use **`mkfs.ext4`** to format the logical volumes (LVs) with **ext4** filesystem.

**`$ sudo mkfs -t ext4 /dev/dbdata-vg/db-lv && sudo mkfs -t ext4 /dev/dbdata-vg/logs-lv`**

![format logical volumes db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/33c81e0f-b165-4d2a-8d8f-05da70479146)

#### <br>Step 8: Create Directories and Mount on Logical Volumes<br/>

In this step, we need to create the directory to hold our database files and then another directory to store backup of log data after which we will respectively mount these directories on the created logical volumes **db-lv** and **logs-lv**.

**i.** We use the following command to create the **/db** directory to store our database files:

**`$ sudo mkdir -p /db`**

**ii.** We use the command below to create the **/home/recovery/logs** directory to store backup of log data:

**`$ sudo mkdir -p /home/recovery/logs`**

**iii.** Then we execute the following command to mount **/db** on **db-lv** logical volume:

**`$ sudo mount /dev/dbdata-vg/db-lv /db**

**iv.** **/var/log** is the default directory where Linux stores all log files. This is the directory that we need to mount on our **logs-lv** volume. However, mounting this directory will delete all the files contained in it so before we carry out this action, we need to use the **`rsync`** utility to backup all the files in the log directory **/var/log** into the **/home/recovery/logs** directory we created. We do this by executing the command below:

**`$ sudo rsync -av /var/log/. /home/recovery/logs/`**

![backup log files](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/1c8fabac-607a-49fd-a538-8abdb294d780)

**v.** Then we enter the command below to mount **/var/log** on **apps-lv** logical volume:

**`$ sudo mount /dev/dbdata-vg/logs-lv /var/log`**

**vi.** Afterwards, we restore the log files back into the **/var/log** directory.

**`$ sudo rsync -av /home/recovery/logs/. /var/log`**

![restore log files](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/da7a0e22-0a8f-4729-8621-f9665e9d6167)

**vii.** The next step is to use the universally unique identifier (UUID) of the device to update the **/etc/fstab** file so that the mount configuration will persist after the restart of the server. We check the UUID of the device by entering the command below:

**`$ sudo blkid`**

![uuid](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e8ce173d-4db6-4f0c-b7bb-b0b56349af4d)

**viii.** We copy the UUID as shown in the above image and we open the **/etc/fstab** file with the following command:

**`$ sudo vi /etc/fstab`**

**ix.** We paste in the copied UUID whilst removing the leading and ending quotes and update the **/etc/fstab** file as shown in the image below:

![mounts for wordpress server](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fc378217-afc5-41ea-ac3a-306d2390f186)

**x.** We test our mount configuration with the following command:

**`$ sudo mount -a`**

**xi.** Then we reload the daemon with the command below:

**`$ sudo systemctl daemon-reload`**

**xii.** To complete our configuration process, we verify our entire setup by executing the following command:

**`$ df -h`**

![df -h final output](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5ca686a1-58f5-47e7-a41f-cffcab726981)

The output must look like what we have in the image above.
