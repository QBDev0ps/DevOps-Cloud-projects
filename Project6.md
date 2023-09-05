## DOCUMENTATION FOR IMPLEMENTING CLIENT SERVER ARCHITECTURE USING MySQL DATABASE MANAGEMENT SYSTEM (DBMS)

This project demonstrates the implementation of a basic client-server setup using MySQL RDBMS. Client-Server refers to a model of computing where the client requests a service and the server provides it. It is an architecture in which two or more computers (i.e. seperate entities of client which is the computer sending the request and server which is the computer responding to the request) communicate through the internet over a network to send and receive requests between one another.

We shall proceed with the by utilizing the following steps:


### <br>Step 1: Create and Configure two Linux Based Virtual Servers<br/>

**i.** We launch two EC2 instances of Ubuntu Linux by following [these steps](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance) and we respectively name them as **`mysql server`** and **`mysql client`**.  __IMPORTANT: From the Amazon Machine Image (AMI Image) tab, be sure to select the free tier eligible version of Ubuntu Linux Server 20.04 LTS (HVM) rather than the HVM version of Amazon Linux 2 Server as directed in the link.__

**ii.** It is especially important and good practice to keep software up to date for security purposes. Packages in a Linux distribution are updated frequently to fix bugs, add features, and protect against security exploits. So after launching our servers we run the following command on both servers to update them:

**`$ sudo apt update`**

![sudo apt update](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/041248b2-4d11-482b-814f-26d31fcac19b)


### <br>Step 2: Install MySQL *Server* Software on **`mysql server`** <br/>

We install MySQL _**Server**_ Software on our "Server" by entering the following command:

**`$ sudo apt install mysql-server -y`**

![mysql-server installation](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/21156ca6-0d05-4df8-82c2-6cde82ccf4dc)


### <br>Step 3: Install MySQL *Client* Software on **`mysql client`** <br/>

And then we install MySQL _**Client**_ Software on our "Client" by entering the following command:

**`$ sudo apt install mysql-client -y`**

![mysql-client installation](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/072942eb-73e3-4575-8a2a-8d065e2d4ec5)


### <br>Step 4: Edit Inbound rules on **`mysql server`** to enable access to **`mysql client`** traffic. <br/>

By default, both of our running EC2 virtual servers are located in the same local virtual network, so they can communicate with each other via local IP addresses. We use the  local IP address of MySQL _**Server**_ to connect from  MySQL _**Client**_ . MySQL _**Server**_ uses TCP port **3306** by default, so we will have to open it by creating a new entry in **"Inbound rules"** in MySQL _**Server**_ Security Groups. For extra security, we shall not allow all IP addresses to reach **`mysql server`** we shall allow access only to the specific local IP address of **`mysql client`** . To execute this we do the following: 

**i.** In the AWS navigation pane, we choose **Instances**.

**ii.** We Select our **`mysql server`** instance and in bottom half of the screen, choose the **Security** tab. **Security groups** lists the security groups that are associated with the instance. Inbound rules displays a list of the **inbound rules** that are in effect for the instance.

**iii.** For the security group to which we will add the new rule, choose the security group ID link to open the security group.

**iv.** On the **Inbound rules** tab, we choose **Edit inbound rules**.

**v.** On the **Edit inbound rules** page, we do the following:

a. Choose **Add rule**.

b. For **Type**, choose **HTTP**.

c. Under **Source**, leave it at **Custom** and select **0.0.0.0/0** in the space with the magnifying glass.

d. Choose **Save rules** at the bottom right corner of the page
