# DOCUMENTATION FOR LAMP STACK IMPLEMENTATION PROJECT

## This project shows the implementation of the LAMP Stack. It covers essential aspects of the LAMP stack implementation such as the process of setting up a Linux Environment, configuring the Apache Web server, managing MySQL databases, and writing PHP code for server side functionality.

## Pre Installations and Dependencies

In order to execute this project successfully we need to first of all complete the following prerequisites:

1. Setup an AWS Account : [Create a free tier AWS Account](https://aws.amazon.com/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all) by following these [steps](https://repost.aws/knowledge-center/create-and-activate-aws-account).

2. Spin up an EC2 Instance with Ubuntu Server: Launch an EC2 instance by following [these steps.](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance) __IMPORTANT: From the Amazon Machine Image (AMI Image) tab, be sure to select the free tier eligible version of Ubuntu Linux Server 20.04 LTS (HVM) rather than the HVM version of Amazon Linux 2 Server as directed in the link.__

3. Download and Install an SSH client: Download and install [PuTTY](https://putty.org/) or Download and install [git](https://git-scm.com/downloads) (the ssh client - git bash will be packaged with the git installation)

4. Establish connection with your EC2 instance: Connect to your EC2 instance via your SSH client by following [these instructions.](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect-linux-inst-ssh.html)

*After completing the necessary prerequisites, we proceed to implement the following steps to fully implement our LAMP Stack and deploy it in AWS Cloud:*

## STEP 1: Installing Apache and Updating the Firewall

The apache web server is the most popular and widely used web server in the world. It is fast, reliable, secure, and can be customized to meet the needs of different environments with the use of modules and extensions. However, before we install apache we need to firstly update our ubuntu web server by running the command below: 

`$ sudo apt update`

![sudo apt update](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/84b1b3b6-bb08-454b-adf9-3df0ad2ae3f0)

After completing the update process we can now proceed to install the apache web server, To do this, we execute the following command:

`$ sudo apt install apache2`

![apache installation](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/05c63919-b424-479c-b075-377091ff5118)

To verify that the apache2 service is running on our ubuntu server, we enter the following command:

`$ sudo systemctl status apache2`

![apache status](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9defa108-ca6d-495a-80ba-d837eda3fcd6)

As can be seen in the status image above, apache is fully active and running.

The next step is to open TCP Port 80 on our machine. Port 80 is the defualt port used by web browsers to access web pages. So we implement this by adding a rule to our EC2 configuration to allow http traffic via port 80. The steps are listed below:

1. Open the Amazon EC2 console at [https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/).

2. From the top navigation bar, select a Region for the security group. Security groups are specific to a Region, so you should select the same Region in which you created your instance.

3. In the navigation pane, choose **Instances**.

4. Select your instance and, in bottom half of the screen, choose the **Security** tab. **Security groups** lists the security groups that are associated with the instance. Inbound rules displays a list of the **inbound rules** that are in effect for the instance.

5. For the security group to which you'll add the new rule, choose the security group ID link to open the security group.

6. On the **Inbound rules** tab, choose **Edit inbound rules**.

7. On the **Edit inbound rules** page, do the following:

a. Choose **Add rule**.

b. For **Type**, choose **HTTP**.

c. Under **Source**, leave it at **Custom** and select **0.0.0.0/0** in the space with the magnifying glass.

d. Choose **Save rules** at the bottom right corner of the page

![edit inbound rules](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9b290578-acd5-48ea-a297-a589ac127e6b)

Selecting the **Source** setting as **0.0.0.0/0** means we can access our server from any IP address. i.e. both locally and from the internet.

To verify that the apache2 webpage is accessible locally from our ubuntu machine, we run the following command:

`$ curl http://localhost:80`   or   `$ curl http://127.0.0.1:80

![verify apache access](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/3e1685db-4835-4ec1-b961-546ac37976d9)

In the output image above, we can see the code of our Apache Service web page displayed on our terminal.

Next we need to test to ensure that our web server is responding to requests from the internet. we open a web page and enter the following url in the adress bar:


