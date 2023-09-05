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



### <br>Step 3: Install MySQL *Client* Software on **`mysql client`** <br/>

And then we install MySQL _**Client**_ Software on our "Client" by entering the following command:

**`$ sudo apt install mysql-client -y`**
