## DOCUMENTATION FOR SETTING UP A BASIC LOAD BALANCER

This project demonstrates the art of load balancing and shows how to distribute traffic efficiently across multiple servers, optimize performance and ensure high availability for web applications.

### <br>Introduction to Load Balancing and Nginx<br/>

Load balancing refers to the method of efficiently distributing network traffic equally across a pool of resources i.e a group of back end servers that support an application. Modern applications must process millions of users simultaneously and return the correct text, videos, images, and other data to each user in a fast and reliable manner.

![what-is-a-load-balancer](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/c5716f66-1dfa-4734-9d3b-9efa0e6b0c62)

Nginx is a very versatile and efficient software that can act as a web server or a reverse proxy. It can also be used as a simple yet powerful load balancer to improve resource availability and system performance. It just requires proper configuration in order to serve the appropriate use case. In this project, our use case shall be load balancing. We shall be using Nginx to set up a basic load balancer and show how it ensures the smooth running of web applications by distributing tasks affectively and efficiently.

### <br>Setting up a Basic Load Balancer<br/>

To execute our project, we will be provisioning two EC2 instances running Ubuntu 22.04 and then we will install apache web server in them. We will open port 8000 to allow traffic from anywhere and finally update the default page of the web servers to display their public IP address. Next, we shall provision another EC2 instance running Ubuntu 22.04, this time, we will install Nginx and configure it to act as a load balancer distributing traffic across the web servers. We will do these by implementing the following steps:


### <br>Step 1: Provisioning EC2 Instance<br/>

We begin by spinning up 2 EC2 Instances of Ubuntu Server: We launch our EC2 instances by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance) 

**i.** We open the AWS console and click on **"EC2"**, then we scroll up and click on **"Launch Instance"**.

![launch EC2 instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d331142c-a425-485d-9338-5e8f21d2a37d)

**ii.** Under **Name and tags**, we provide a unique name for each of our web servers.

![Name and tags](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a28dd984-04f5-4dc9-bcf2-d9fc4a5812fb)
  
**iii.** From the **Applications and Amazon Machine Image (AMI Image)** tab, we ensure we select the free tier eligible version of Ubuntu Linux Server 20.04 LTS (HVM).

![Application and OS Image](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ae844641-2121-49de-99e3-67c8621c4027)
  
**iv.** Under **Key pair**, we select an existing one. (You can create a new key pair if you do not have one and the same key pair can be used for all the instances that will be provisioned in this project.)

![Key Pair](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/65facdd1-4be3-4ec5-aac4-aadd74821653)
  
**v.** And then finally, we click on **"Launch Instance"**
  
![Launch Instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ed623db9-831f-4c86-bc46-f0e7201c18f6)


### <br>Step 2: Open Port 8000<br/>

We will be running our Web Servers on Port 8000 while the Load Balancer will be running on Port 80. We will therefore need to open Port 8000 to allow traffic from anywhere. To implement this, we need to add a rule to the Security Group of each of our Web Servers:

**i.** In the AWS  console navigation pane, we choose **Instances**.

![Instances](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/75d2208f-d030-4f44-9667-23521332607f)

**ii.** We click on our Instance ID to get the details of our EC2 instance and in the bottom half of the screen, we choose the **Security** tab. **Security groups** lists the security groups that are associated with the instance. Inbound rules displays a list of the **inbound rules** that are in effect for the instance.

![instance summary](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/401e3b80-6b6b-4fff-a754-f9fecd97852e)

**iii.** For the security group to which we will add the new rule, we choose the security group ID link to open the security group.

![security groups](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f4453010-cf80-4e64-aab5-d6ac89c2a5fc)

**iv.** On the **Inbound rules** tab, we choose **Edit inbound rules**.

![Edit Inbound Rules](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ca7e7378-eba1-455e-a439-f91dd34cc038)

**v.** On the **Edit inbound rules** page, we do the following:

+ Choose **Add rule**.

+ For **Port Range**, enter **8000** 

+ In the space with the magnifying glass under **Source**, choose **Anywhere**.

+ Click on **Save rules** at the bottom right corner of the page.

![save rules](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/c4bb985a-04dc-4463-998a-07b047e49207)


### <br>Step 3: Install Apache Web Server<br/>

After we have provisioned both of our servers and we have opened the necessary ports, we will proceed to install apache software on both servers. However, we must first connect to each of the web servers via an SSH client. This will enable us to subsequently be able to run commands on the terminal of our web servers. We carry this out by doing the following:

**i.** Download and Install an SSH client: Download and install [PuTTY](https://putty.org/) or Download and install [git](https://git-scm.com/downloads) (the ssh client - git bash will be packaged with the git installation)

**ii.** Establish connection with the EC2 instance: We connect to our EC2 instance via our SSH client by following [these instructions:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect-linux-inst-ssh.html)

+  To initiate connection to the webserver, we click on Instance ID. Then at the top of the page, we click on connect.

![Connect to Instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/044430ee-72f6-4249-8db1-1b437d703b7b)

+  Next, we click on **"SSH Client"** tab and we copy the ssh command under **"Example:"** as shown in the image below:

![copy ssh command](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e7e4fc04-b354-40a3-b467-175a85620741)

+  Next, we open a terminal on our ssh client in our local machine, then we change directory **`cd`** to the downloads folder and then we paste and execute the ssh command we copied in the previous step.

  ![cd Downloads](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f199e640-a738-45d1-94d1-eca0bdbb5d94)

+  We click on **Enter** and type **Yes** when prompted. This connects us to a terminal on our EC2 instance.

  ![connection to instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/813e8bf9-7776-4a5b-922c-a99b4228c265)

**iii.** The next thing to do is to install apache. However, before we install apache we need to firstly update our ubuntu web server. So we concatenate both the web server update and apache installation commands by executing the following: 

**`$ sudo apt update -y &&  sudo apt install apache2 -y`**

![install apache](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/bc06ed4e-319c-4f2d-956c-8b0744a07bc4)

To verify that the apache2 service is running on our server, we enter the following command:

**`$ sudo systemctl status apache2`**

![check apache status](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5264da13-fd9c-46bf-a7db-512a0e69a871)

**iv.** We repeat **ii.** and **iii.** above for the second EC2 instance as well.


### <br>Step 4: Configure Apache to Serve a Page Showing its Public IP<br/>

In this step, we will commence by configuring **Apache** web server to serve content on Port 8000 rather than on its default port which is Port 80. The next thing we'll do is to create a new **index.html** file. The file will contain code to display the public IP of the EC2 instance. We will then override Apache webserver's default html file with our new file. 

**i.** **Configuring Apache to serve content on Port 8000**

+ We use the Vi editor to open the apache ports configuration file /etc/apache2/ports.conf by entering the following command: 

**`$ sudo vi /etc/apache2/ports.conf`**

+  Add a new listen directive for Port 8000. As shown in the image below, we press **`i`** to enter insert mode and then we add the listen directive. Afterwards, on our keyboard, we press **`esc`**, type **`:wq!`** to save and quit immediately and press **`enter`** to confirm exit.

+ The next thing we do is to open the file /etc/apache2/sites-available/000-default.conf by entering the following command:

**`$ sudo vi /etc/apache2/sites-available/000-default.conf`**

+ Then, as shown in the image below, we press **`i`** to enter insert mode and change Port 80 on the virtual host to 8000. Afterwards, on our keyboard, we press **`esc`**, type **`:wq!`** to save and quit immediately and press **`enter`** to confirm exit.

+ To complete this process, we restart the Apache service with the command below:

**`$ sudo systemctl restart apache2`**

**ii.** **Creating our New HTML File**

+ We open a new index.html file by entering the following command:

**`$ sudo vi index.html`**

+ We sitch the Vi editor to insert mode by pressing **`i`** and then we paste in the following block of code. We obtain the Public IP adress of our EC2 instance by copying it from the AWS management console.

```
        <!DOCTYPE html>
        <html>
        <head>
            <title>My EC2 Instance</title>
        </head>
        <body>
            <h1>Welcome to my EC2 instance</h1>
            <p>Public IP: YOUR_PUBLIC_IP</p>
        </body>
        </html>
```

+ We change the file ownership rights of the index.html file by executing the following command:

**`$ sudo chown www-data:www-data ./index.html`**


**iii.** **Overriding the Default html file of Apache Web Server**

