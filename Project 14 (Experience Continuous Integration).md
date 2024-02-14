## EXPERIENCE CONTINUOUS INTEGRATION WITH JENKINS | ANSIBLE | ARTIFACTORY | SONARQUBE | PHP

This project is partly a continuation of the ongoing infrastructure development with Ansible started from Project 11. We will be setting up a pipeline that simulates continuous integration and delivery for a PHP based application. The target end to end CI/CD pipeline is represented as shown in the diagram below:

![ci-cd pipeline for todo php app](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/d6297db1-2dc2-4df9-af96-9a998474aa7e)

### <br>Introduction to Continuous Integration<br/>

In software engineering, Continuous Integration (CI) is a practice of merging all developers' working copies to a shared mainline (e.g., Git Repository or some other version control system) several times per day. Frequent merges reduce chances of any conflicts in code and allow to run tests more often to avoid massive rework if something goes wrong. 

However, the concept of CI is not only about committing your code. There is a general workflow, which is described as follows:

**Run tests locally:** Before developers commit their code to a central repository, it is recommended to test the code locally. So, Test-Driven Development (TDD) approach is commonly used in combination with CI. Developers write tests for their code called  unit-tests, and before they commit their work, they run their tests locally. This practice helps a team to avoid having one developer's work-in-progress code from breaking other developers' copy of the codebase.

**Compile code in CI:** After testing codes locally, developers commit and push their work to a central repository. Rather than building the code into an executable locally, a dedicated CI server picks up the code and runs the build there. In this project we will use, already familiar to you, Jenkins as our CI server. Build happens either periodically - by polling the repository at some configured schedule, or after every commit. Having a CI server where builds run is a good practice for a team, as everyone has visibility into each commit and its corresponding builds.


**Run further tests in CI:** Even though tests have been run locally by developers, it is important to run the unit-tests on the CI server as well. But, rather than focusing solely on unit-tests, there are other kinds of tests and code analysis that can be run using CI server. These are extremely critical to determining the overall quality of code being developed, how it interacts with other developers' work, and how vulnerable it is to attacks. A CI server can use different tools for Static Code Analysis, Code Coverage Analysis, Code smells Analysis, and Compliance Analysis. In addition, it can run other types of tests such as Integration and Penetration tests. Other tasks performed by a CI server include production of code documentation from the source code and facilitate manual quality assurance (QA) testing processes.


**Deploy an artifact from CI:** At this stage, the difference between CI and CD is spelt out. As you now know, CI is Continuous Integration, which is everything we have been discussing so far. CD on the other hand is Continuous Delivery which ensures that software checked into the mainline is always ready to be deployed to users. The deployment here is manually triggered after certain QA tasks are passed successfully. There is another CD known as Continuous Deployment which is also about deploying the software to the users, but rather than manual, it makes the entire process fully automated. Thus, Continuous Deployment is just one step ahead in automation than Continuous Delivery.

### Project Dependencies

In order to successfully execute this project, the following prerequisites need to be in place:

1. Servers: We will be making use of six(6) AWS virtual machines for this project and these include:

+ Jenkins server: This will be used to implement our CI/CD workflows or pipelines. Select a t2.medium at least, Ubuntu 20.04 and Security group should be open to port 8080.
  
+ Nginx server: This would act as the reverse proxy server to our site and tool.
  
+ SonarQube server: This will be used for Code quality analysis. Select a t2.medium at least, Ubuntu 20.04 and Security group should be open to port 9000.
  
+ Artifactory server: This will be implemented as the binary repository where the outcome of your build process is stored. Select a t2.medium at least and Security group should be open to port 8081.
  
+ Database server: This will serve as the databse server for the Todo application
  
+ Todo webserver: This will be used to host the Todo web application.
  
2. Security Groups: For the purposes of this project, we can have one security group that is open to all traffic. This should however not be attempted in a real DevOps enviroment.
  
3. Ansible Inventory: Our Ansible inventory is expected to look like this:

```
├── ci
├── dev
├── pentest
├── pre-prod
├── prod
├── sit
└── uat
```

**`ci`** inventory file

```
[jenkins]
<Jenkins-Private-IP-Address>

[nginx]
<Nginx-Private-IP-Address>

[sonarqube]
<SonarQube-Private-IP-Address>

[artifact_repository]
<Artifact_repository-Private-IP-Address>
```

**`dev`** inventory file

```
[tooling]
<Tooling-Web-Server-Private-IP-Address>

[todo]
<Todo-Web-Server-Private-IP-Address>

[nginx]
<Nginx-Private-IP-Address>

[db:vars]
ansible_user=ec2-user
ansible_python_interpreter=/usr/bin/python

[db]
<DB-Server-Private-IP-Address>
```

**`pentest`** inventory file

```
[pentest:children]
pentest-todo
pentest-tooling

[pentest-todo]
<Pentest-for-Todo-Private-IP-Address>

[pentest-tooling]
<Pentest-for-Tooling-Private-IP-Address>
```

4. Ansible Roles: We need to add two more roles to ansible for our CI Environment:

+ [SonarQube:](https://www.sonarqube.org) SonarQube is an open-source platform developed by SonarSource for continuous inspection of code quality, it is used to perform automatic reviews with static analysis of code to detect bugs, [code smells](https://en.wikipedia.org/wiki/Code_smell), and security vulnerabilities.
  
+  [Artifactory:](https://jfrog.com/artifactory/) Artifactory is a product by [JFrog](https://jfrog.com) that serves as a binary repository manager. The binary repository is a natural extension to the source code repository, in that the outcome of your build process is stored. It can be used for certain other automation, but we will it strictly to manage our build artifacts.

### <br>Install and Set-up Jenkins on EC2 Instance<br/>

To begin our project we need to install and set up Jenkins. Jenkins is an open-source automation tool written in Java with plugins built for continuous integration. Jenkins is used to build and test software projects continuously making it easier for developers to integrate changes to the project, and making it easier for users to obtain a fresh build. It also allows developers to continuously deliver software by integrating with a large number of testing and deployment technologies. We deploy Jenkins by implementing the following steps:

#### <br>Step 1: Provision EC2 Instance<br/>

We begin by spinning up an EC2 Instance of Ubuntu Server: We launch our EC2 instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance) 

**i.** We open the AWS console and click on **"EC2"**, then we scroll up and click on **"Launch Instance"**.

![launch EC2 instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d331142c-a425-485d-9338-5e8f21d2a37d)

**ii.** Under **Name and tags**, we provide a unique name for our server.
  
**iii.** From the **Applications and Amazon Machine Image (AMI Image)** tab, we ensure we select the free tier eligible version of RHEL Linux Server 8.7.0(HVM).

![launch instance](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/09c8d44a-43a4-4900-b906-e4c6087cfacf)

**iv.** Under **Key pair**, we select an existing one. (You can create a new key pair if you do not have one and the same key pair can be used for all the instances that will be provisioned in this project.)

![Key Pair](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/65facdd1-4be3-4ec5-aac4-aadd74821653)
  
**v.** And then finally, we click on **"Launch Instance"**
  
![Launch Instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ed623db9-831f-4c86-bc46-f0e7201c18f6)

#### <br>Step 2: Open Port 8080<br/>

**Jenkins** uses TCP Port 8080 by default and as this will be our first installation, we will therefore need to open Port 8080 to allow traffic from anywhere. To implement this, we need to add a rule to the Security Group of our Web Server:

**i.** In the AWS  console navigation pane, we choose **Instances**.

![Instances](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/75d2208f-d030-4f44-9667-23521332607f)

**ii.** We click on our Instance ID to get the details of our EC2 instance and in the bottom half of the screen, we choose the **Security** tab. **Security groups** lists the security groups that are associated with the instance. Inbound rules displays a list of the **inbound rules** that are in effect for the instance.

![instance summary](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ee3841b0-95f1-43be-94b1-6210d7bea296)

**iii.** For the security group to which we will add the new rule, we choose the security group ID link to open the security group.

![security groups](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f4453010-cf80-4e64-aab5-d6ac89c2a5fc)

**iv.** On the **Inbound rules** tab, we choose **Edit inbound rules**.

![Edit Inbound Rules](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ca7e7378-eba1-455e-a439-f91dd34cc038)

**v.** On the **Edit inbound rules** page, we do the following:

+ Choose **Add rule**.

+ For **Port Range**, enter **8080** 

+ In the space with the magnifying glass under **Source**, choose **Anywhere**.

+ Click on **Save rules** at the bottom right corner of the page.

![open port 8080](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d41b3f43-59c9-4c78-8a80-0fa40515399a)

#### <br>Step 3: Create and Allocate Elastic IP Address to Jenkins-Ansible Server<br/>

Considering that we'll be using Jenkins with Github and configuring Web Hooks in this project, it will make our job easier to create and allocate an elastic IP adress to our Jenkins-Ansible Server. This is beacuse everytime we stop/start the server, there will be a need to keep reconfiguring Github Web Hooks to a new IP address. Having an elastic IP address (which will not change when we stop/start the server) is the ideal way to overcome this issue.

**i.** From the EC2 cockpit, we click on **"Elastic Ips"**.

![elastic IPs](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2bd8c559-b18f-4db9-9446-84eed4c34c81)

**ii.** In the next page displayed, we click on **"Allocate Elastic IP Address"**.

![Allocate elastic IP address](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/633bb57d-7414-425e-ba73-66faa341eeb1)

**iii.** In the elastic IP address settings page we ensure to choose the same network border group as our EC2 instance. Then we select the option to choose from Amazon's pool of IPv4 addresses. Then we click on **"Allocate"**.

![elastic IP address settings](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2b22c080-2bfa-4b8f-aa59-808fc7d1ba9f)

**iv.** In the next page which shows us that the elastic IP has been allocated successfully, we click on the **"Actions"** drop down tab and we select **"Associate Elastic IP address"**.

![associate elatic ip 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/00c69fbe-bea1-435c-ba5a-37323e19ccc7)

**v.** In the Associate Elastic IP Address page, we scroll down to **"Instance"** and we select our EC2 instance. After this, we click on the **"Associate"** button at the bottom of the page.

![Associate Elastic IP address](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a8ee57a4-991e-4505-9fb8-9b7132353d8a)

**vi.** As can be seen in the output image below, the elastic IP address was successfully asssociated with our EC2 instance.

![Elastic IP successfully associated](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a6dba97b-2953-485e-a13f-ecbe8ac62dd4)

#### <br>Step 4: Connect to the Jenkins-Ansible Server via the Terminal using the SSH Client<br/>

After we have provisioned our server and we have opened the necessary port, we must next connect to the server via an SSH client. This will enable us to subsequently be able to run commands on the terminal of our server. We carry this out by doing the following:

**i.** Download and Install an SSH client: Download and install [Visual Studio Code.](https://code.visualstudio.com/download)
Visual Studio Code is a streamlined code editor with support for development operations like debugging, task running, and version control. It aims to provide just the tools a developer needs for a quick code-build-debug cycle and leaves more complex workflows to fuller featured IDEs, such as Visual Studio IDE. VS Code Remote SSH lets you access a remote computer or virtual machine securely over a network connection. You can connect over SSH into another machine from Visual Studio Code and interact with files and folders anywhere on that remote filesystem.

**ii.** Establish connection with the EC2 instance: We configure connection to our EC2 instance via our VS Code platform by following [these instructions:](https://code.visualstudio.com/docs/remote/ssh)

#### <br>Step 5: Jenkins Installation and Set-up<br/>

**i.** After connecting to our server we must first update all installed packages and their dependencies before commencing other installations or configurations. We do this by executing the following command: 

**`$ sudo yum update -y`**

![update jenkins server](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/cd776836-fe2c-4acf-9fe6-a7dc69d4a40a)

**ii.** Next, we run the following set of commands to [install dependencies for Jenkins](https://pkg.jenkins.io/debian-stable/):

```
$ sudo wget -O /etc/yum.repos.d/jenkins.repo \ https://pkg.jenkins.io/redhat-stable/jenkins.repo

$ sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

$ sudo dnf upgrade

# Add required dependencies for the jenkins package

$ sudo dnf install fontconfig java-11-openjdk
```

![jenkins installation1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/411dbe67-5f08-456a-a58c-f703b4b2ecf2)

**iii.** Then we execute the command below to install Jenkins:

```
$ sudo dnf install jenkins

$ sudo systemctl daemon-reload
```

![jenkins installation 3](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/18b922b5-11a6-4681-9a6f-dd1bbf970bf9)

**iv.** To start and enable Jenkins, then ensure Jenkins is up and running, we enter the commands below:

```
$ sudo systemctl start jenkins

$ sudo systemctl enable jenkins

$ sudo systemctl status jenkins
```

![start and enable jenkins](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/78d95951-446a-4121-9136-749bf23cbc66)

**v.** Then we begin setting up Jenkins by accessing it via our browser using the following syntax: 

**`http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080`**

![unlock Jenkins](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/730aadd0-bfb3-4774-b009-c7c7de539a2d)

**vi.** As shown in the output image above, we are prompted to provide an Administrator password to unlock Jenkins. To retrieve the password from the Jenkins Server, we enter the following command: 

**`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`**

![initial admin password](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b4b11c70-6079-4fef-a5bd-996b8959fc5a)

**vii.** Next, from the server, we copy the password as seen in the image above and we paste it in the dialogue box in the **"Unlock Jenkins"** page after which we click on **"Continue"**.

![unlock jenkins 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ce0044ef-005e-498d-adc1-620b9a1f1e92)

**viii.** This brings us to the **"Customize Jenkins"** page. Here we select and click on **"Install Suggested Plugins"**.

![jenkins install suggested plugins](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7b78280e-b378-4529-86db-be2804b09de0)

![installing plugins](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4dd246cf-eb66-42e0-ac53-f038475ca8ca)

**ix.** As shown in the above image, we were able to successfully install the plugins. After completion, Jenkins prompts us to **"Create a First Admin User"** we can either fill in our details to do this or we just choose to **"Skip and continue as admin"**. We decide to go with the latter option.

![skip and continue as admin](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/82c59719-18a2-49b5-b1aa-610527076f30)

**x.** In the instance configuration page, we paste in our elastic ip address and click on **"Save and Finish"**.

![instance configuration](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7a0e7627-7661-4af4-b270-d9efbca0207f)

**xi.** At this point the set-up is complete and Jenkins is ready to be used. We click on **"Start using Jenkins"** to move into the main Jenkins Environment.

![installation complete](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b725826c-ef8b-4906-b6ac-02d37cca0d43)

