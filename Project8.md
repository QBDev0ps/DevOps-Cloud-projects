## DOCUMENTATION FOR AUTOMATING LOAD BALANCER CONFIGURATION WITH SHELL SCRIPTING

This project demonstrates how to use shell scripting to automate the set up and maintenance of a load balancer to effectively enhance efficiency and reduce manual effort to the barest minimum.

### <br>Introduction to Automating Load Balancer Configuration with Shell Scripting<br/>

Load balancing refers to the method of efficiently distributing network traffic equally across a pool of resources i.e a group of back end servers that support an application. Modern applications must process millions of users simultaneously and return the correct text, videos, images, and other data to each user in a fast and reliable manner.

![what-is-a-load-balancer](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/194dc498-8204-45ec-9c86-b9455879adb5)

A [Shell Script](https://en.wikipedia.org/wiki/Shell_script) is a collection of commands and instructions that are executed sequentially in a shell. Essentially, shell scripting helps us automate repetitive tasks.

In this project, we will be automating the entire process of deploying two back end webservers with a load balancer distributing traffic across both servers. But rather than implementing these tasks manually we will be executing a shell script that will carry out the tasks automatically therefore demonstrating how we can speed up the deployment of services and reduce the chance of making errors in our day to day activities.

### <br>Automating the Deployment and Configuration of Web Servers<br/>

To begin our project we need to deploy and configure two Apache Web Servers. To do this, we will provision an EC2 instance running Ubuntu 22.04 then we will execute a script that will update our EC2 instance and afterwards, the script will install apache web server on it. Subsequently, the script will configure apache to listen on Port 8000 and finally update the default page of the web servers to display their public IP address. Our script will pass one argument which will be the Public IP adress of the EC2 instance that is being provisioned. To run our script, we will implement the following steps:

#### <br>Step 1: Provisioning EC2 Instance<br/>

We begin by spinning up an EC2 Instance of Ubuntu Server: We launch our EC2 instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance) 

**i.** We open the AWS console and click on **"EC2"**, then we scroll up and click on **"Launch Instance"**.

![launch EC2 instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d331142c-a425-485d-9338-5e8f21d2a37d)

**ii.** Under **Name and tags**, we provide a unique name for each of our web servers.

![Name and tags](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a28dd984-04f5-4dc9-bcf2-d9fc4a5812fb)
  
**iii.** From the **Applications and Amazon Machine Image (AMI Image)** tab, we ensure we select the free tier eligible version of Ubuntu Linux Server 22.04 LTS (HVM).

![Application and OS Image](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ae844641-2121-49de-99e3-67c8621c4027)

**iv.** Under **Key pair**, we select an existing one. (You can create a new key pair if you do not have one and the same key pair can be used for all the instances that will be provisioned in this project.)

![Key Pair](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/65facdd1-4be3-4ec5-aac4-aadd74821653)
  
**v.** And then finally, we click on **"Launch Instance"**
  
![Launch Instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ed623db9-831f-4c86-bc46-f0e7201c18f6)

#### <br>Step 2: Open Port 8000<br/>

We will be running our Web Server on Port 8000 while the Load Balancer will be running on Port 80. We will therefore need to open Port 8000 to allow traffic from anywhere. To implement this, we need to add a rule to the Security Group of our Web Server:

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

#### <br>Step 3: Connect to the Webserver via the Terminal using the SSH Client<br/>

After we have provisioned our server and we have opened the necessary port, we must next connect to the web server via an SSH client. This will enable us to subsequently be able to run commands on the terminal of our web server. We carry this out by doing the following:

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


 #### <br>Step 4: Prepare and Execute Script<br/>

To prepare and execute our script, we shall carry out the actions below:

**i.** We proceed to create and open a file called install.sh file by entering the following command:

**`$ sudo vi install.sh`**

**ii.** We switch the Vi editor to insert mode by pressing **`i`** and then we copy and paste in the following script:

```
#!/bin/bash

####################################################################################################################
##### This automates the installation and configuring of apache webserver to listen on port 8000
##### Usage: Call the script and pass in the Public_IP of your EC2 instance as the first argument as shown below:
######## ./install_configure_apache.sh 127.0.0.1
####################################################################################################################

set -x # debug mode
set -e # exit the script if there is an error
set -o pipefail # exit the script when there is a pipe failure

PUBLIC_IP=$1

[ -z "${PUBLIC_IP}" ] && echo "Please pass the public IP of your EC2 instance as an argument to the script" && exit 1

sudo apt update -y &&  sudo apt install apache2 -y

sudo systemctl status apache2

if [[ $? -eq 0 ]]; then
    sudo chmod 777 /etc/apache2/ports.conf
    echo "Listen 8000" >> /etc/apache2/ports.conf
    sudo chmod 777 -R /etc/apache2/

    sudo sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8000>/' /etc/apache2/sites-available/000-default.conf

fi
sudo chmod 777 -R /var/www/
echo "<!DOCTYPE html>
        <html>
        <head>
            <title>My EC2 Instance</title>
        </head>
        <body>
            <h1>Welcome to my EC2 instance</h1>
            <p>Public IP: "${PUBLIC_IP}"</p>
        </body>
        </html>" > /var/www/html/index.html

sudo systemctl restart apache2
```

**iii.** Afterwards, on our keyboard, we press **`esc`**, type **`:wq!`** to save and quit immediately and press **`enter`** to confirm exit.

**iv.** We change permissions to make the file executable by running the command below:

**`$ sudo chmod +x user-input.sh`**

**v.** We run our Shell Script by executing the following command:

  **`$ sudo ./install.sh PUBLIC_IP`**

#### <br>Step 5: Prepare and Execute Script<br/>

### <br>Automating the Deployment and Configuration of Nginx Load Balancer<br/>
