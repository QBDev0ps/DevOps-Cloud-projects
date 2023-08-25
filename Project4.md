# DOCUMENTATION FOR WEB STACK IMPLEMENTATION (LEMP) PROJECT

This project shows the implementation of the LEMP Stack. It covers essential aspects of the LEMP stack implementation such as the process of setting up a Linux Environment, configuring Nginx for optimal performance, managing MySQL databases, and writing PHP code to bring applications to life.

### Pre Installations and Dependencies

In order to execute this project successfully we need to first of all complete the following prerequisites:

1. Setup an AWS Account : [Create a free tier AWS Account](https://aws.amazon.com/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all) by following these [steps](https://repost.aws/knowledge-center/create-and-activate-aws-account).

2. Spin up an EC2 Instance with Ubuntu Server: Launch an EC2 instance by following [these steps.](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance) __IMPORTANT: From the Amazon Machine Image (AMI Image) tab, be sure to select the free tier eligible version of Ubuntu Linux Server 20.04 LTS (HVM) rather than the HVM version of Amazon Linux 2 Server as directed in the link.__

3. Download and Install an SSH client: Download and install [git](https://git-scm.com/downloads) (the ssh client - git bash will be packaged with the git installation)

4. Establish connection with your EC2 instance: Connect to your EC2 instance via your SSH client by following [these instructions.](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect-linux-inst-ssh.html)

_**After completing the necessary prerequisites, we proceed with the following steps to fully implement our LEMP Stack and deploy it in AWS Cloud:**_


## STEP 1: Installing the Nginx Web Server

Nginx is a high performance web server we will employ to display web pages to our site visitors. However, before we install Nginx we need to firstly update our ubuntu web server by running the command below:

**`$ sudo apt update`**

![sudo apt update](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/77a5ae94-bcd7-42ef-974f-a0662313001c)

After completing the update process we can now proceed to install the Nginx web server, To do this, we execute the following command:

**`$ sudo apt install nginx`**

![sudo apt install](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/774f3a51-4e34-4bf0-a31b-73cc3ec34cf5)

When prompted, we enter **`Y`** to confirm that we want to install Nginx. When the installation is complete, the Nginx service should be active and running on our ubuntu server. To verify this, we enter the following command:

**`$ sudo systemctl status nginx`**

![sudo systemctl status](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fc9e7ce0-a782-466b-8ef0-218320e1232c)

As can be seen in the status image above, Nginx is fully active and running.

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

![edit inbound rules](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/52ad1dc6-1d47-4b1d-a6e9-d9deafdc5a90)

Selecting the **Source** setting as **0.0.0.0/0** means we can access our server from any IP address. i.e. both locally and from the internet.

To verify that the apache2 webpage is accessible locally from our ubuntu machine, we run the following command:

**`$ curl http://localhost:80`**   or  **`$ curl http://127.0.0.1:80`**

![curl command](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/1793dc4b-a4aa-4e5f-bae6-bc8dd140dead)

In the output image above, we can see the [HTML](https://en.wikipedia.org/wiki/HTML) code of our Nginx Service web page displayed on our terminal.

Next we need to test to ensure that our web server is responding to requests from the internet. To implement this, we open a web browser and in the adress bar, we enter the url using this syntax **`http://<PUBLIC-IP-ADDRESS>:80`**. We can retrieve our public IP address by entering the following command:

**`$ curl -s http://169.254.169.254/latest/meta-data/public-ipv4`**

![IP adress](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/cc27425a-dd76-46f8-a8e4-f80e532c0825)

Now that we have the public IP address, we proceed to enter the following url in our browser:

**`http://16.171.139.68:80`**

![nginx web server home page](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/6c659594-3de3-4e89-8bcc-f6503617fc9f)

As shown in the output image above, we can see the Nginx default web page which tells us that our web server is now properly installed and accessible through our firewall.


## STEP 2: Installing MySQL

After completing the setup of our Web Server, we need to install a Database Management System to be able to store and manage data for our webpage. MySQL is an open-source Relational Database Management System (RDBMS) that enables users to store, manage, and retrieve structured data efficiently. It is widely used for various applications, from small-scale projects to large-scale websites and enterprise-level solutions. To run this installation, we enter the following command:

**`$ sudo apt install mysql-server -y`**

![MySQL server installation](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4910b0ed-424d-49f9-883e-affcd7c60556)

As shown in the output image above, the -y flag allows the installation to run till completion without the need for any prompts. After the installation is complete, we log into the MySQL console by executing the command below:

**`$ sudo mysql`**

We should see an output like in the image below:

![log into mysql](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2bb027e7-fa71-4b3d-b741-2ff98a059fe1)

Next, we need to run a script that is preinstalled with MySQL to help secure access to our database system. However, before running the script, we need to set a password (which will be defined as **`PassWord.1`**) for the root user. We implement this by entering the command below:

**`mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';`**

Then we exit with the following command:

**`mysql> exit`**

![alter mysql user password and exit](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4d305f16-e23e-4b0b-89a8-1f1dfcd3a1bb)

Next we run the MySQL interactive script by executing the following command:

**`$ sudo mysql_secure_installation`**

As shown in the output image below, we use the **`VALIDATE PASSWORD PLUGIN`** to set up a strong password for MySQl root user and we follow the remaining process whilst choosing **`Y`** for the rest of the prompts.

![mysql script](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/341a72ca-db10-4da4-bc76-0924980e00d9)

Next, with the following command, we confirm our abilty to log into the MySQl console with our newly created root password:

**`$ sudo mysql -p`**

This prompts us for the root password and as we can see in the output image below, we are able to log in.

![test mysql password](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/c0187915-df38-4999-a2aa-2d27b023bcbb)

As shown in the above image, to exit the MySQL console, we simply execute the command below:

**`mysql> exit`**


## STEP 3: Installing PHP

After installing Nginx Web Server and MySQL Database Management System. It is now time for our PHP installation. PHP is the component of our LEMP Stack setup that allows webpages run dynamic processes to enable content to be displayed on the end user's browser. While Apache embeds the PHP interpreter in each request, Nginx requires an external program to handle PHP processing and act as a conduit between the PHP interpreter itself and the webserver. Although this ensures a better overall performance in most PHP websites, it however requires some additional configuration. To proceed, we will need to install the following PHP packages:

a. **`php-fm`**: This stands for “PHPfastCGIptocess manager”. We will need to configure Nginx to pass PHP requests to this software for processing. 

**`php-mysql`**: This enables communication between PHP and MySQL based databases.

Along with the above two, other core PHP packages will be automatically installed as dependencies. To proceed, we install the above two packages together at the same time by entering the following command:

**`$ sudo apt install php-fpm php-mysql`**

![PHP installation](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/932cf2ef-aa1c-4ae9-bf75-904c2fbd0ac9)

As seen in the ouput image above, the sytem installs the two PHP modules. Now that we have our PHP modules installed, the next step is to configure Nginx to use them.


## STEP 4: Configuring Nginx to use PHP Processor

Similar to Virtual Hosts in Apache, Nginx enables us to create server blocks so we can have multiple websites on a single machine. Nginx already has one server block enabled by default which is configured to serve files out of a directory **`/var/www/html`**. This works well for single sites but not multiple websites so we will need to create a directory structure within **`/var/www`** to hold our website's files and folders. To do this we need to firstly set up a domain we will be calling **`projectLEMP`**.

To create the directory for **`projectLEMP`**, we execute the following command:

**`$ sudo mkdir /var/www/projectLEMP`**

Next we execute the command below to assign ownership of the created directory by using the `$USER environment variable which references your current system's user:

**`$ sudo chown -R $USER:$USER /var/www/projectLEMP`**

Then, using the nano editor, we create and open a new configuration file in Nginx’s **`sites-available`** directory by executing the following command:

**`$ sudo nano /etc/nginx/sites-available/projectLEMP`** 

The above command creates a blank file. Then we copy and paste the following configuration into the nano editor:

```
#/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}
```

![nano editor](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/41bb001b-d270-4bc6-bdea-cf3217074dd9)

To save the file, press **`CTRL+O`** and press enter. To exit the editor, press **`CTRL+X`**

Next, we activate our configuration by linking to the config file from Nginx's **`sites-enabled`** directory:

**`$ sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/`**

This will tell Nginx to use the configuration next time it is reloaded. We can test for syntax errors by executing the following command:

**`$ sudo nginx -t`**

The output for this is shown in the image below:

![test nginx configuration](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/75e3b66d-15c6-4ac9-87ed-907687db7bf1)

We also need to disable the default Nginx host that is currently configured to listen on Port 80. For this, we execute the following:

**`$ sudo unlink /etc/nginx/sites-enabled/default`**

And afterwards, using the command below, we reload Nginx so that the changes can take effect:

**`$ sudo systemctl reload nginx`**

Next, to test that our new server block works as expected, we create an index.html file in the /var/www/projectLEMP directory:

```
sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
```

Then the next step is to go to our browser to open our website URL via our public IP address (syntax is: **http://<Public-IP-Address>:80**) or via our server's DNS name (syntax is: **http://<Public-DNS-Name>:80**) In our own use case, we enter the following url in our browser:

**`http://16.171.139.68:80`**

The ouput from the browser is as shown in the image below:

![lempstack webpage1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fd42a64f-3956-43c0-be6c-2914456c3026)


## STEP 5: Testing PHP with Nginx

At this point, our LEMP stack is fully installed and operational. We can however test it to validate that Nginx can correctly hand **`.php`** files off to our PHP processor. To do this, we shall create a test PHP file within our document root called **`info.php`**. We implement this with the nano editor by executing the following command:

**`$ nano /var/www/projectLEMP/info.php`**

Next, we paste in the below valid PHP code that should return information about our server:

```
<?php
phpinfo();
```

We save and exit the file by pressing **`CTRL+X`**, then **`y`** and **`ENTER`**

We can now access this page in our web browser by visiting the domain name or IP address we have set up in our Nginx configuration file, followed by **`/info.php`**. The syntax for this is **`http://`server_domain_or_IP`/info.php**. In our own use case, we enter the following in our browser:

**`http://16.171.139.68:80/info.php`**

The output is as shown in the image below:
