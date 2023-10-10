## DOCUMENTATION FOR DEVOPS TOOLING WEBSITE SOLUTION

In the [previous project](https://github.com/QBDev0ps/DevOps-Cloud-projects/blob/main/Project9.md), we implemented a WordPress solution that is ready to be filled with content and can be used as a full fledged website or blog. Subsequently, in this project, we will be adding some more value to our solutions by implementing a comprehensive DevOps tooling website solution that will integrate various tools and technologies to create a unified platform that enhances collaboration, automation and efficiency for software development and operations teams.

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

**2.** Configure Backend Database Server.

**3.** Configure Web Servers.

As shown in the diagram below, we shall implement a solution where three stateless Web Servers will share a common database and also access the same files using Network File Sytem (NFS) as a shared file storage. Even though the NFS server might be located on a completely separate hardware – for the Web Servers it would look like a local file system from where they can serve the same files.

![3 TIER Tooling-Website-Infrastructure](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/61a271c9-4efd-45ff-a71e-94e62340faf8)

### <br>Configure Network File Server (NFS) Server with storage infastructure<br/>

To begin our project we need to deploy and configure LVM for our Linux based NFS server. We do this by implementing the following steps:


#### <br>Step 1: Provision EC2 Instance<br/>

We begin by spinning up an EC2 Instance of Red Hat Linux that will serve as our NFS Server: We launch our EC2 instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance) 

**i.** We open the AWS console and click on **"EC2"**, then we scroll up and click on **"Launch Instance"**.

![launch EC2 instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d331142c-a425-485d-9338-5e8f21d2a37d)

**ii.** Under **Name and tags**, we provide a unique name for our server.

![name and tags](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/04cf9f94-d41c-44ca-a108-346911a7a646)
  
**iii.** From the **Applications and Amazon Machine Image (AMI Image)** tab, we ensure we select the free tier eligible version of Red Hat Enterprise Linux 8 (HVM).

![application and OS images](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2beaaf89-c746-4ed5-a434-f0989bfb3db1)

**iv.** Under **Key pair**, we select an existing one. (You can create a new key pair if you do not have one and the same key pair can be used for all the instances that will be provisioned in this project.)

![Key Pair](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/65facdd1-4be3-4ec5-aac4-aadd74821653)
  
**v.** And then finally, we click on **"Launch Instance"**
  
![Launch Instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ed623db9-831f-4c86-bc46-f0e7201c18f6)

#### <br>Step 2: Create EBS Volumes<br/>

The next course of action is to create 3 EBS volumes of 10GB each in the same Availability Zone as our EC2 Linux NFS Server instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-creating-volume.html) 

**i.** We open the AWS console and click on **"EC2"**, then we scroll down in the navigation pane and click on **"Volumes"** under **"Elastic Block Store"**.

![volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2befe33d-7849-4677-9252-fd7d4af93b26)

**ii.** Click on **"Create Volume"** at the top right hand corner of the **"Volumes"** page.

![Create Volume](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9e710712-916e-4543-8b37-b2dc3a357c12)

**iii.** On the **"Volume Settings"** page, we select the **"Volume Type"**, set **"Size"** as 10GB, we select the **"Availability Zone"** of our EC2 Web Server and the we click on **"Create Volume"** at the bottom right corner of the page. 

![Volume Settings](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fc5ac363-e6f6-45af-9e5a-19cf3342a6ed)

**iv.** We repeat **i-iii** above twice to create two more Elastic Block Store (EBS) Volumes.

#### <br>Step 3: Attach EBS Volumes to EC2 NFS Server Instance<br/>

After creating the 3 EBS volumes we proceed to attach them to our EC2 NFS Server instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-attaching-volume.html)

**i.** We open the AWS console and click on **"EC2"**, then we scroll down in the navigation pane and click on **"Volumes"** under **"Elastic Block Store"**.

![volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2befe33d-7849-4677-9252-fd7d4af93b26)

**ii.** Select the Volume you wish to attach under the Volumes list then right click and select **"Attach Volume"**.

![Attach Volume](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/32ec447b-5d02-4328-8582-528573bb3667)

**iii.** In the **"Attach Volume"** page, under **"Instance"**, we select our EC2 Linux NFS Server Instance, under **"Device name"** the name can be changed from **/dev/sdf** through **/dev/sdp** depending on preferences, then we click on **"Attach Volume"** at the bottom right corner of the page.

![Attach Volume 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/36039f97-ae1c-4eb8-85ae-8239b16f5e32)

**iv.** We repeat **i-iii** above twice to attach the remaining two Elastic Block Store (EBS) Volumes to our EC2 Linux NFS Server Instance.

#### <br>Step 4: Connect to the NFS Server via the Terminal using the SSH Client<br/>

After we have provisioned our server and we have created and attached our EBS volumes, we must next connect to the NFS server via an SSH client. This will enable us to subsequently be able to run commands and begin configuration on our NFS server. We carry this out by doing the following:

**i.** Download and Install an SSH client: Download and install [Termius](https://www.termius.com/download/windows) or Download and install [git](https://git-scm.com/downloads) (the ssh client - git bash will be packaged with the git installation)

**ii.** Establish connection with the EC2 instance: We connect to our EC2 instance via our Termius SSH client by following [these instructions:](https://dev.to/aws-builders/how-to-connect-your-ec2-linux-instance-with-termius-5209)

#### <br>Step 5: Update NFS Server and Inspect Attached Block Devices<br/>

**i.** After connecting to our server we must first update all installed packages and their dependencies before commencing configuration. We do this by executing the following command: 

**`$ sudo yum update -y`**

![Sudo yum update -y](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/c5f2681a-141f-4dc7-8d8a-ac527184a9e3)

**ii.** Subsequently, we inspect what block devices are attached to the web server with the following command:

**`$ lsblk`**

As can be seen in the image below, our EBS volumes are shown using the **`nvme`** naming convention rather than **`xvdf`**. This is because our block devices are connected through an NVMe port which uses the nvme driver on Linux. It should also be noted that EBS volumes are typically exposed as NVMe block devices on instances built on the Nitro System. The device names are **/dev/nvme0n1**, **/dev/nvme1n1**, and so on. The Nitro System is a collection of hardware and software components built by AWS that enable high performance, high availability, and high security.

![lsblk](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e3f43630-958c-4c6d-b4a5-eebf08ab4e1c)

The **`lsblk`** command reveals that **/dev/nvme0n1** is the default storage device attached to our EC2 instance and has four partitions **/dev/nvme0n1p1-4** with **/dev/nvme0n1p4** mounted as the root device. The block devices we created are listed as **/dev/nvme1n1**, **/dev/nvme2n1** and **/dev/nvme3n1** and as can be seen, they have no mount points because they are not yet mounted.

**iii.** To see all mounts and free space on our Web Server, we run the command below.

**`$ df -h`**

![df -h](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2cfe8778-107c-48b8-a68e-878eed860816)

**iv.** We also proceed to check the **/dev/** directory with the following command: 

**`$ ls /dev/`**

![ls dev](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/079bef77-e12b-49db-b0a5-58c308771c2b)

As can be seeen in the above image, the executed command lists all Linux devices and we can see that our attached block devices  **/dev/nvme1n1**, **/dev/nvme2n1** and **/dev/nvme3n1** are listed.

#### <br>Step 6: Partition Disks and Install lvm2 Package<br/>

**i.** In this step, we proceed to create a single partition on each of the three (3) Disks using the **`gdisk`** utility. We partition **/dev/nvme1n1** by executing the following command: 

**`$ sudo gdisk /dev/nvme1n1`**

As shown in the output image below, we enter **`?`** to list out all the available commands in the gdisk console, we enter the **`p`** command to provide information about available space in hard disk to create a new partition. Subsequently we enter the **`n`** command to create a new partition, we follow through with the prompts and then we enter **`w`** to write the partition table to disk and exit the gdisk console. When the system requests for input with **`Do you want to proceed? (Y/N):`**,  we enter **`y`** to confirm our earlier operation.

![sudo gdisk](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a5b37c83-2f90-4748-a090-abd71d005a33)

We repeat the same process above to create a single partition on **/dev/nvme2n1** and **/dev/nvme3n1**.

**ii.** Afterwards, we run the **`lsblk`** utility to view the newly configured partition on each of the three (3) disks.

![lsblk 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/23676904-26bb-4cb3-84d2-2bd6e51bf49f)

As can be seen in the image above, we have our newly configured partitions on each of the three (3) disks listed as **`nvme1n1p1`**, **`nvme2n1p1`** and **`nvme3n1p1`**

**iii.** The next course of action is to install the **`lvm2`** package using the following command:

**`$ sudo yum install lvm2 -y`**

![install lvm2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/450cf513-5859-42b5-8bbc-636d9b2560ae)

**iv.** Then we run the command below to check for available partitions:

**`$ sudo lvmdiskscan`**

![sudo lvmdiskscan](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e40db3a8-5549-483e-bc44-401aefb6143e)

#### <br>Step 7: Create Physical and Logical Volumes<br/>

**i.** For the next step, we use the **`pvcreate`** utility to mark each of our three (3) partitioned disks as physical volumes (PVs) to be used by LVM:

**`$ sudo pvcreate /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1`**

![create physical volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/71e43f90-ba1b-42e0-9908-b8b3b02b4438)

**ii.** Then afterwards, we verify that the physical volumes (PVs) have been created by executing the command below:

**`$ sudo pvs`**

![sudo pvs](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/177f5cfa-aff0-4d49-81ba-2cf42db3044c)

**iii.** After this, we proceed to use the **`vgcreate`** utility to add all three (3) physical volumes (PVs) to a volume group (VG) that we will be naming **nfsdata-vg**

**`$ sudo vgcreate nfsdata-vg /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1`**

![create volume group](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/778a3827-cd9b-45d6-8dca-d3f99490b46d)

**iv.** And then we confirm that our volume group (VG) has been created by successfully executing the following command:

**`$ sudo vgs`**

![sudo vgs](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/abded900-be1e-47a7-bb1c-7759b9a2a03c)

**v.** The next step is to create three (3) logical volumes (LVs) **lv-opt**, **lv-apps** and **lv-logs** using the **`lvcreate`** utility. 

```
$ sudo lvcreate -n lv-opt -L 9G nfsdata-vg
$ sudo lvcreate -n lv-apps -L 9G nfsdata-vg
$ sudo lvcreate -n lv-logs -L 9G nfsdata-vg
```

![create logical volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f50679a1-39b3-42af-b858-8023d16d183b)

**vi.** And then we confirm that our logical volumes have been created by successfully executing the following command:

**`$ sudo lvs`**

![sudo lvs](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a51722e4-114c-4993-80c3-370427b0a757)

**vii.** Then we verify our entire setup of Volume Group (VG), Physical Volumes (PV) and Logical Volumes (LV) with the following commands:

**`$ sudo vgdisplay -v`**

![sudo vgdisplay](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ffbdda15-513c-4bb9-8dc7-b4c9afc56b4a)

**`$ sudo lsblk`**

![sudo lsblk set up verification](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/04b9bd0e-7781-4f52-80b3-b68887058302)

**viii.** To complete the process, we use **`mkfs.xfs`** to format the logical volumes (LVs) with **xfs** filesystem.

**`$ sudo mkfs -t xfs /dev/nfsdata-vg/lv-opt && sudo mkfs -t xfs /dev/nfsdata-vg/lv-apps && sudo mkfs -t xfs /dev/nfsdata-vg/lv-logs`**

![format logical volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5ab3f85b-9e84-4386-b336-246300626b71)

#### <br>Step 8: Create Directories and Mount on Logical Volumes<br/>

In this step, we need to create the directory to hold our application files and then another directory to store backup of log data and then a directory we will use for Jenkins in the future. After creating the directories we will respectively mount them  on the created logical volumes **lv-apps**, **lv-logs** and **lv-opt**.

**i.** We use the following command to create the **/mnt/apps** directory to store our application files:

**`$ sudo mkdir -p /mnt/apps`**

**ii.** We use the command below to create the **/mnt/logs** directory to store backup of log data:

**`$ sudo mkdir -p /mnt/logs`**

**iii.** We use the following command to create the **/mnt/opt** directory to to be used by Jenkins Server in a future project:

**`$ sudo mkdir -p /mnt/opt`**

**iv.** Then we execute the following commands to mount the logical volumes on to the directories we just created:

```
$ sudo mount /dev/nfsdata-vg/lv-apps /mnt/apps
$ sudo mount /dev/nfsdata-vg/lv-logs /mnt/logs
$ sudo mount /dev/nfsdata-vg/lv-opt /mnt/opt
```

![mount logical volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/203269d2-5dd7-4e5f-ad2e-894e2a7e9647)

**vii.** The next step is to use the universally unique identifier (UUID) of the device to update the **/etc/fstab** file so that the mount configuration will persist after the restart of the server. We check the UUID of the device by entering the command below:

**`$ sudo blkid`**

![sudo blkid](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ffa0853d-6ebe-409f-b3d1-f3ee5bbf4202)

**viii.** We copy the UUID as shown in the above image and we open the **/etc/fstab** file with the following command:

**`$ sudo vi /etc/fstab`**

**ix.** We paste in the copied UUID whilst removing the leading and ending quotes and update the **/etc/fstab** file as shown in the image below:

![mounts for nfs server](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/6f1c76a0-27ce-42de-a613-35d6bcfd50f7)

**x.** Afterwards, on our keyboard, we press **`esc`**, type **`:wq!`** to save and quit immediately and press **`enter`** to confirm exit.

**xi.** We test our mount configuration with the following command:

**`$ sudo mount -a`**

**xii.** Then we reload the daemon with the command below:

**`$ sudo systemctl daemon-reload`**

**xiii.** To complete our configuration process, we verify our entire setup by executing the following command:

**`$ df -h`**

![df -h final output](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5650411d-f55e-4a4f-8e8c-827c19e1ca5d)

The output must look like what we have in the image above.

#### <br>Step 9: Install NFS Server<br/>

**i** Firstly, we update the server machine if we have not already done so:

**`$ sudo yum -y update`**

![Sudo yum update -y](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/c5f2681a-141f-4dc7-8d8a-ac527184a9e3)

**ii.** The next course of action is to install the NFS server using the nfs-utils package as shown in the command below:

**`$ sudo yum install -y nfs-utils`**

![install NFS server](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/605821d1-650a-4ca0-9df2-a547f414208c)

**iii.** After the installation is complete, we start the NFS service with the following command:

**`$ sudo systemctl start nfs-server.service`**

**iv.** Next, we execute the command below to configure NFS to always start on system reboot:

**`$ sudo systemctl enable nfs-server.service`**

![enabls nfs](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2baa542e-058d-49df-9def-207cb9f2251b)

**v.** And then to conclude, we check the status of the NFS service to verify that it is active and enabled:

 **`$ sudo systemctl status nfs-server.service`**

 ![status NFS service](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7ca6c7e3-b832-4913-846f-250ff332472f)
