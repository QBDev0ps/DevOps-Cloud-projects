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

+ Jenkins server: This will be used to implement our CI/CD workflows or pipelines. Select a t2.medium at least, RHEL 8.7.0 and Security group should be open to port 8080.
  
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

We begin by spinning up an EC2 Instance of Red Hat Linux Server: We launch our EC2 instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance) 

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

![obtain jenkins password](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/58f4b959-103d-42ba-945e-11c77e0ace43)

**vii.** Next, from the server, we copy the password as seen in the image above and we paste it in the dialogue box in the **"Unlock Jenkins"** page after which we click on **"Continue"**.

![unlock jenkins 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ce0044ef-005e-498d-adc1-620b9a1f1e92)

**viii.** This brings us to the **"Customize Jenkins"** page. Here we select and click on **"Install Suggested Plugins"**.

![jenkins install suggested plugins](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7b78280e-b378-4529-86db-be2804b09de0)

![installing plugins](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4dd246cf-eb66-42e0-ac53-f038475ca8ca)

**ix.** As shown in the above image, we were able to successfully install the plugins. After completion, Jenkins prompts us to **"Create a First Admin User"** we can either fill in our details to do this or we just choose to **"Skip and continue as admin"**. We decide to go with the latter option.

![skip and continue as admin](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/82c59719-18a2-49b5-b1aa-610527076f30)

**x.** In the instance configuration page, we  click on **"Save and Finish"**.

![instance configuration](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7a0e7627-7661-4af4-b270-d9efbca0207f)

**xi.** At this point the set-up is complete and Jenkins is ready to be used. We click on **"Start using Jenkins"** to move into the main Jenkins Environment.

![installation complete](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b725826c-ef8b-4906-b6ac-02d37cca0d43)

### Configuring Ansible For Jenkins Deployment

In previous projects, we were launching Ansible commands manually from a CLI. Now, with Jenkins, we will start running Ansible from Jenkins UI.

To do this,

1. Navigate to Jenkins URL

![navigate to jenkins url](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/b828c6ab-c700-4a4d-ab7a-0d18659312c4)

2. From the Jenkins Dashboard, we click on **`Manage Jenkins`**, the we click the **`Plugins`** button, then we select **`Available plugins`**, and then in the search bar, we type in Blue Ocean, and we subsequently install and open Blue Ocean Jenkins Plugin.

![install blue ocean plug in](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/4c03d450-ac8d-4065-9fa7-04f18e6ba5ba)

3. In the blue Ocean User Interface, we click on **`Create a new pipeline`** button.

![create new pipeline](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/6eafdff7-5709-4f62-9b55-b26a7a8ca576)

+ We select GitHub as where we store our code.

![select github as code storage](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/ab932f72-a380-4e11-8522-d204717ac010)

+ Next, we proceed to connect Jenkins with GitHub.

![connect jenkins to github](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/22548a10-6429-4b84-8ca3-4401b4bbcb62)

+ We login to our Github account and generate an access token.

![jenkins token1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/ce2f0c23-8d4b-422d-9b0e-608cedb02b3d)

![jenkins token 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/4c9bf178-3b33-4e15-ac11-00e8aa403429)

+ We copy the access token

![copy jenkins token](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/77cc67da-6116-4e18-bf86-46990b61b69a)

+ Then we paste it and connect

![paste token and connect](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/8b81b77e-f323-44ac-8736-dc8ba59a507a)

+ Next we select the Repo owner and the **`ansible-config-mgt`** repository and then we click on **`Create Pipeline`**

![create pipeline](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/2a1a682e-b4cf-44eb-b522-cd98791ff6b3)

+ At this point we do not have a Jenkinsfile in the Ansible repository, so Blue Ocean attempts to give us some guidance to create one. But we do not need this. Rather, we opt to ceate one ourselves. So, we click on Administration to exit the Blue Ocean console.

![click on administration](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/16e1c76b-fa11-4f4c-b5ec-d4bccc1b684b)

+ We can find our newly created pipeline in our Jenkins dashboard.

![created pipeline](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/9afe4af0-9fd1-40b6-a32a-6d59a93a2c2f)


### Creating our `Jenkinsfile`

1. We naviagte to the ansible-config-mgmt repository in our github account and we copy the HTTPS clone URL.

![copy ansible-config-mgt clone url](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/54539b55-1d58-4b36-930c-327ee73e7713)

2. We navigate to our VS Code terminal and install git using the following command:

**` $ sudo yum install git -y`**

![install git](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/5b4d3d33-6dc0-4055-91f8-fad195939701)

3. After completing the installation we clone our ansible config mgt repo using the URL we copied:

**` $ git clone https://github.com/QuadriBello/ansible-config-mgt.git`**

![git clone](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/770dc96e-b9a0-4692-b1ca-7c006ea7ed37)

4. Inside the Ansible project, we create a new directory **`deploy`** and then start a new file **`Jenkinsfile`** inside the directory.

![creae directory deploy and file jenkinsfile](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/6b681ec4-ae1d-45e7-b07d-1498b7bc7e62)

5. Next, we add the code snippet below to start building the **`Jenkinsfile`** gradually. This pipeline currently has just one stage called **`Build`** and the only thing we are doing is using the **`shell script`** module to echo **`Building Stage`**

```
pipeline {
    agent any


  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }
    }
}
```

![jenkins file code](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/307082c8-15d6-47dd-a870-c7e04a1bdbd9)

6. Now we go back into the Ansible pipeline in Jenkins, and select **`configure`**.

![select configure](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/65ffadfe-7330-43c9-9857-be35852f6496)
   
7. We scroll down to **`GitHub`** section and select Jenkins Credential provider, then we enter our GitHub credentials.

![Jenkins credential provider](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/baae7fb8-7692-4f19-a03b-5c89327a2250)
   
8. We scroll down to **`Build Configuration`** section and specify the location of the **Jenkinsfile** at **`deploy/Jenkinsfile`**.

![build configuration](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/61aefbab-1422-4743-a38f-d6ee241d03f5)

9.  We navigate to our VS Code terminal and execute the following commands:

```
$ git config --global user.name "QuadriBello"      #enter username credentials for github 

$ git config --global user.email "moyor_bello@yahoo.co.uk"  #enter email credentials for github 
 
$ cd ansible-config-mgt           #move into the ansible-config-mgt repository

$ git branch                      #confirm you are in the main branch

$ git add .                       #stage all changes

$ git commit -m "saved Jenkinsfile" #save staged changes and include a commit message

$ git push                       #push changes to remote repository
```

![git push](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/2af054c0-0aa0-41e1-900b-665b3eb41617)

10. We go back to the pipeline again, and this time click on **`Scan Repository now`** and we refresh the Jenkins GUI page.

![scan repo now](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/1b0a67f8-4d06-437d-bfea-f47701c33024)

11. This triggers a build and we are able to see the effect of our basic Jenkinsfile configuration by going through the console output of the build.

![console output](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/9004f0ce-3a19-4c48-a221-4b49e9ebc0d9)

To really appreciate and feel the difference of Cloud Blue UI, we proceed to trigger the build again from Blue Ocean interface.

1. We click on the Blue Ocean plugin on the Jenkins dashboard.

2. We select our project
   
3. And we click on the play button against the branch

![blue ocean build](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/a555ee04-b008-4ae7-b0dc-e1e6bf069e2c)

4. This pipeline is a multibranch one. This means, if there were more than one branch in GitHub, Jenkins would have scanned the repository to discover them all and we would have been able to trigger a build for each branch.
  
Let us see this in action.

1. Using the command below, we create a new git branch and name it **`feature/jenkinspipeline-stages`**

**`$ git checkout -b feature/jenkinspipeline-stages`**

![git checkout](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e6ac9769-9e73-4b9f-b305-3124badbf045)

2. Currently we only have the `Build` stage. Let us add another stage called `Test`. We paste the code snippet below:
   
```
   pipeline {
    agent any

  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }

    stage('Test') {
      steps {
        script {
          sh 'echo "Testing Stage"'
        }
      }
    }
    }
}

```

![jenkins file test stage](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/199d9e93-3a70-49f1-bfcb-c3f86cc65545)

3. And then we add, commit and push the new changes to github with the following set of commands:

```
$ git add .

$ git commit -m "added test stage"

$ git push --set-upstream origin feature/jenkinspipeline-stages
```

![git add commit and push](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/5727bde0-bb34-484b-acac-d63bae02d4ab)

4. To make our new branch show up in Jenkins, we need to tell Jenkins to scan the repository. We navigate to the Ansible project and click on "Scan repository now" and we refresh the Jenkins dashboard.

![scan repository for chnages](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/0fd088a5-fb25-45ee-bd56-f45656357c53)

5. We refresh the page and both branches start building automatically. We go into Blue Ocean and see both branches there too.

6. In Blue Ocean, we can now see how the **`Jenkinsfile`** has caused a new step in the pipeline launch build for the new branch.

![blue ocean testing stage](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/9798db71-3d1a-44ab-a7ea-3f2fd153fb96)

7. Create a pull request to merge the latest code into the `main branch`

![pull request](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/85aa152f-d47a-497d-914c-39ac1bc7cf9f)

8. After merging the `PR`, we go back into our terminal and switch into the `main` branch.

**`$ git switch main`**

![git switch main](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/8b9b3b9c-3bcb-4bad-8e3c-bbba29e65535)

9. Then we pull the latest change from our GitHub repository:

**`$ git pull`**

![git pull](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/b31ddcc1-fb91-4a5b-9645-846eca3ea680)

10. Next we create a new branch, and add more stages into the Jenkins file to simulate the **`Package`**, **`Deploy`** and **`Clean up`** phases.

![add new stages](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/ca2d7e03-30d4-49e8-bfc7-133180f5f1f1)

11. And then we add, commit and push the new changes to github with the following set of commands:

```
$ git add .

$ git commit -m "added new stages"

$ git push
```

![git add commit and push 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/4b0dbc40-44e1-4b43-a73b-1d3880c67bad)

12. We navigate back to our Jenkins pipeline and we click on "Scan repository now" and we refresh the Jenkins dashboard.

![scan repository now 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/62629230-3fd1-4c85-b5b8-e0f1b811303f)

13. Then we verify in Blue Ocean that all the stages are working.

![blue ocean new stages](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/4884ce7b-1bef-4eff-a26d-7e87efc421bd)

### Running Ansible Playbook from Jenkins

Now that we have a broad overview of a typical Jenkins pipeline. We proceed to get the actual Ansible deployment to work by implementing the following: 

1. Installing Ansible and its dependencies on Jenkins. To do this, we execute the following commands:

```
$ sudo yum install ansible -y

$ sudo yum install python3 python3-pip wget unzip git -y

$ sudo python3 -m pip install --upgrade setuptools

$ sudo python3 -m pip install --upgrade pip

$ sudo python3 -m pip install PyMySQL

$ sudo python3 -m pip install mysql-connector-python

$ sudo python3 -m pip install psycopg2==2.7.5 --ignore-installed

$ ansible-galaxy collection install community.mysql

$ ansible-galaxy collection install community.postgresql
```

![install ansible](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/dc0b2182-3c41-4988-ac14-d0f921353893)

![install dependencies](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/4f86f833-cd93-4ed7-a70e-51882b739ff4)

2. Installing Ansible plugin in Jenkins UI. From the Jenkins Dashboard, we click on **`Manage Jenkins`**, then we click the **`Plugins`** button, next we select **`Available plugins`**, and then in the search bar, we type in "Ansible", and we subsequently install the Ansible plugin.

![install ansible plugin](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c49a8e68-5460-4348-b36a-71e4d34e496a)

3. Next, we specify ssh keys to be used for ansible by adding the keys as global credentials in Jenkins. To do this, we go to the Jenkins dashboard, click on "Manage Jenkins", navigate to "Credentials", click on "System", then select "Global Credentials" and click on the "Add Credentials" button. In the dilogue box, we fill in the credential ID, the username and then we paste in the contents of our .pem key. This will enable us run Ansible playbook via the Jenkinsfile.

![generate ssh key](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/345e00b7-8d38-4a4b-8d23-821fa6bd351e)

4. Ansible plugin will require the ansible interpreter that is installed on the Jenkins server for it to work. So we will need to specify the path for the interpreter in Jenkins. We obtain the path where ansible is installed on our server by running the command below:

**`$ which ansible`**

![which ansible](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e8a5d64c-042e-4c42-8bbf-de0f817eb103)

5. Then we navigate to the global tool configuration in Jenkins, we click on "Add Ansible" and then we enter the name of the ansible executable and specify the path to the ansible executable folder.

![add ansible](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/93fde6d9-ddce-4c99-a236-6b6fe69112b1)

6. Introduce parameterization by creating `Jenkinsfile` from scratch. To do this we delete all of the code we have in the file and we paste in the following lines of code:

```
pipeline {
  agent any

  environment {
      ANSIBLE_CONFIG="${WORKSPACE}/deploy/ansible.cfg"
    }

  parameters {
      string(name: 'inventory', defaultValue: 'dev',  description: 'This is the inventory file for the environment to deploy configuration')
    }

  stages{
      stage("Initial cleanup") {
          steps {
            dir("${WORKSPACE}") {
              deleteDir()
            }
          }
        }

      stage('Checkout SCM') {
         steps{
            git branch: 'main', url: 'https://github.com/QuadriBello/ansible-config-mgt.git'
         }
       }

      stage('Prepare Ansible For Execution') {
        steps {
          sh 'echo ${WORKSPACE}' 
          sh 'sed -i "3 a roles_path=${WORKSPACE}/roles" ${WORKSPACE}/deploy/ansible.cfg'  
        }
     }

      stage('Run Ansible playbook') {
        steps {
           ansiblePlaybook become: true, colorized: true, credentialsId: 'private-key', disableHostKeyChecking: true, installation: 'ansible', inventory: 'inventory/${inventory}', playbook: 'playbooks/site.yml'
         }
      }

      stage('Clean Workspace after build'){
        steps{
          cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenUnstable: true, deleteDirs: true)
        }
      }
   }

}
```

![create jenkins file from scratch](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e1058e12-8e1e-4506-ae87-9707cc1723dc)

7. Using the Pipeline Syntax tool in Ansible, we generate the syntax to create environment variables to set. We navigate and enter the parameters as shown in the image below:

![pipeline syntax](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e3e9bd11-08e1-4f6b-8e10-14166e9a36dd)

8. Then we ensure we check the box for disabling the host SSH key check and we click on "Generate Pipeline Script". Next, we copy the script up to the point specified in the image below and we use it to replace the script in the `Run Ansible playbook` stage in our **`Jenkinsfile`**.

![pipeline script](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/8030d52e-7e5a-4991-9a98-2feb2f25797f)

![ansible playbook script replacement](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/3b72fc1b-6f9b-4c1a-a811-45096e5e31b6)

8. Next, we need to create a global ansible configuration file that will specify pointers on variables to use and default settings on how we want ansible to run. In ansible, the global configuration file we are creating will take precedence over the local configuration file that came with our ansible installation. So, in the deploy folder, we create a file named **`ansible.cfg`** and paste in the configuration below.

```
[defaults]
timeout = 160
callback_whitelist = profile_tasks
log_path=~/ansible.log
host_key_checking = False
gathering = smart
ansible_python_interpreter=/usr/bin/python3
allow_world_readable_tmpfiles=true

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=30m -o ControlPath=/tmp/ansible-ssh-%h-%p-%r -o ServerAliveInterval=60 -o ServerAliveCountMax=60 -o ForwardAgent=yes
```

![create ansible configuration file](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/cb759136-c537-4c2c-8876-3ab154ce00fa)

9. We note that **`ansible.cfg`** must be exported to environment variable so that Ansible knows where to find Roles. To do this, we make sure to specify the role path in the global ansible configuration file.

![specify role path](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e7ff7cf2-bbed-4b43-9dc8-8fc5a4c9e067)

10.  Next, we ensure that Ansible runs against the Dev environment successfully. To do this, we implement the following:

+ Provison EC2 instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance)

+ Add provisioned instance to ansible inventory **`dev.yml`** using the private IP address.

 ![dev inventory](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/84bdb33e-f786-4d28-a706-bc549f28b6d4)

+ Specify the nginx role to be used under **`static-assignments`**

![specify roles](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/1e193d31-5c10-4008-ac36-2d5d78b81cd5)

+ update the ansible playbook file **`site.yml`** with the relevant nginx configuration.

![specify dev playbook](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/fc05b4d6-2354-4eb3-8d70-be62db49aaf4)

+ And then we add, commit and push the new changes to github with the following set of commands:

```
$ git add .

$ git commit -m "updated Jenkinsfile"

$ git push
```

![git add commit and push 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/4b0dbc40-44e1-4b43-a73b-1d3880c67bad)

+ We navigate back to our Jenkins pipeline and we click on "Scan repository now" and we refresh the Jenkins dashboard.

![scan repository now 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/62629230-3fd1-4c85-b5b8-e0f1b811303f)

+ Then we verify in Blue Ocean that our ansible playbook is working.

![invoke ansible playbook](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/1923d4cd-0533-4d59-97e2-e6829c60f74b)

![ansible run dev successful](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/815e30e0-d503-44ff-befb-6af268b9947e)


### Parameterizing `Jenkinsfile` For Ansible Deployment

But what if we need to deploy to other environments, manually updating the **`Jenkinsfile`** is definitely not an option. To deploy to other environments, we will need to use parameters.

1. Update `sit` inventory with new servers

```
[tooling]
<SIT-Tooling-Web-Server-Private-IP-Address>

[todo]
<SIT-Todo-Web-Server-Private-IP-Address>

[nginx]
<SIT-Nginx-Private-IP-Address>

[db:vars]
ansible_user=ec2-user
ansible_python_interpreter=/usr/bin/python

[db]
<SIT-DB-Server-Private-IP-Address>
```

2. We update `Jenkinsfile` to introduce parameterization. Below is just one parameter. It has a default value in case if no value is specified at execution. It also has a description so that everyone is aware of its purpose.

```
pipeline {
    agent any

    parameters {
      string(name: 'inventory', defaultValue: 'dev',  description: 'This is the inventory file for the environment to deploy configuration')
    }
...
```

3. In the Ansible execution section, we remove the hardcoded `inventory/dev` and replace with **`${inventory}`**

![parameter](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/9d4ac4b0-2c3f-4f07-a17f-4b29a52931ef)

### CI/CD Pipeline for TODO application

We already have `tooling` website as a part of deployment through Ansible. Here we will introduce another PHP application to add to the list of software products we are managing in our infrastructure. The good thing with this particular application is that it has unit tests, and it is an ideal application to show an end-to-end CI/CD pipeline for a particular application.

Our goal here is to deploy the application onto servers directly from `Artifactory` rather than from `git`.

#### Phase 1 - Prepare Jenkins

1. We Fork the repository below into our GitHub account.
   
```
https://github.com/darey-devops/php-todo.git
```

![fork todo ](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c9516921-cccb-4f6a-b257-ff22dbc17523)

![fork todo 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/493f10d2-7d59-4a84-8601-52f4fb029bf0)

2. On our Jenkins server, we install PHP and its dependencies then we install [Composer tool](https://getcomposer.org)

```
$ sudo yum module reset php -y

$ sudo yum module enable php:remi-7.4 -y

$ sudo yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json

$ sudo systemctl start php-fpm

$ sudo systemctl enable php-fpm

$ curl -sS https://getcomposer.org/installer | php

$ sudo mv composer.phar /usr/bin/composer

$ sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm -y

$ sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm

$ sudo dnf module reset php -y

$ sudo dnf module install php:remi-7.4 -y

$ sudo dnf --enablerepo=remi install php-phpunit-phploc

$ wget -O phpunit https://phar.phpunit.de/phpunit-7.phar

$ chmod +x phpunit 

$ sudo yum install php-xdebug
```

![php installation 1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/5ee6ce04-d7f8-4640-a7c7-2730599e2fa2)

![php installation 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/01d7bcbf-d6c0-403d-8456-2685e491a06b)

![php installation 3](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/3ca8f425-cad7-412b-b5b9-89835ba9958f)

![php installation 4](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/f6489734-9708-4fe5-a101-0221bf530715)

3. Install Jenkins plugins

+ [Plot plugin](https://plugins.jenkins.io/plot/) - We will use `plot` plugin to display tests reports, and code coverage information. From the Jenkins Dashboard, we click on **`Manage Jenkins`**, then we click the **`Plugins`** button, then we select **`Available plugins`**, and then in the search bar, we type in "Plot", and we subsequently install the Plot Plugin.

![plot plugin](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/0e319210-f5d6-4769-be52-34d21362931d)

+ [Artifactory plugin](https://www.jfrog.com/confluence/display/JFROG/Jenkins+Artifactory+Plug-in) - The `Artifactory` plugin will be used to easily upload code artifacts into an Artifactory server. From the Jenkins Dashboard, we click on **`Manage Jenkins`**, then we click the **`Plugins`** button, then we select **`Available plugins`**, and then in the search bar, we type in "Artifactory", and we subsequently install the Artifactory Plugin.

![artifactory plugin](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/20fceba5-87a2-49d6-8af7-7071a99a1619) 

4.  Next, we carry out the following steps to install artifactory using a role via our Jenkins pipeline:

**i.** We spin up an EC2 instance for our artifactory server by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance)

![artifactory instance](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/663ad090-f6b9-4827-a57f-5393585c8ed4)

**ii.** We open ports TCP 8081 and 8082 in the server's security group.

![open ports 8081 and 8082](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/d180be1d-c63e-4d32-ad67-4ad6dd8ef058)

**iii.** Then we proceed to add the private IP address of our artifactory server to our ansible inventory in **`ci.yml`**

![private ip inventory](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/9b88f000-6f42-4afb-98f2-17b945ded1c6)

**iv.** Next, we update Ansible with an Artifactory role.

![artifactory role](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/5f23d6a6-6f90-44ee-9e5b-b6c6776c3cc7)

**v.** Under **`static-assignments`**, we reference the artifactory host and role in the **`uat-webservers.yml`** file.

![static assignments](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/eaf9fe8d-2843-46c5-b79e-512e77c52262)

**vi.** Then in our **`site.yml`** file in the playbooks folder, we import the configuration from **`static-assignments`**

![artifactory siteyml](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/fe4ad800-ff93-45c6-8393-bb416562e4cd)

**vii.** Next we edit the **Checkout SCM** stage in **`Jenkinsfile`** and we paste in the URL to our Git repository the git branch to main and push code

![checkout scm](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/920bbb3c-20bf-49de-8411-276108b801f4)

**viii.** Then using the following commands, we add, commit and push all our changes to our remote Git repository.

```
$ git add .

$ git commit -m "updated Jenkinsfile"

$ git push
```

![add commit and push](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/27283a28-1cad-4e86-b2d2-c7a9e165f175)

**ix.** We navigate back to our Jenkins pipeline and we click on "Scan repository now" and we refresh the Jenkins dashboard.

![scan reposit now](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/8e5ce85b-8fc0-435c-b580-4616d3b5cb1f)

**x.** Then we click on the **`ansible-config-mgt`** repository, in the repository page we click on **Build with Parameters** and we enter **`ci.yml`** as the inventory path.

![build with parameters](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/266470ab-5561-4569-87ff-1dc2fee460a9)

**xi.** Subsequently we click on the Blue Ocean plugin to view the output of our build.

![artifactory playbook success 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/d830383b-89d7-4d0a-9633-0d07f7090f1c)

![artifactory playbook success](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/dcbf9780-05b1-4f6a-966b-b2e007f0d204)

**xii.** After successful completion of the build, we Login into the Jfrog Artifactory using the public IP address of the server with port 8081 and we login using username (admin) and password (password).

![jfrog login](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/203e46f4-9d0b-40e8-957a-f5a00843083c)

**xiii.** Next, we reset the admin password and complete the log in process.

![reset admin password](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/56b97719-2171-458e-948c-850b1b5b566d)

5. In Jenkins UI configure Artifactory

**i.** To initiate this configuration, we move to the dashboard of our Jenkins UI, from here, we click on **"Manage Jenkins"** and under **"System Configuration"**, we click on **"System"**.

![system configuration](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/d7bbdbab-5458-4635-bf6d-074bba64cebc)

**ii.** Under **"Jfrog Plugin Configuration"**, click on **"Add Jfrog Platform Instance"**

**iii.** Next, we configure the server ID, and then the Jfrog URL and Credentials.

![jfrog plugin configuration](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/af98e701-be28-43c0-bd13-3494f8213f59)

#### Phase 2 - Integrate Artifactory repository with Jenkins

**i.** We create a dummy Jenkinsfile in the php-todo repository.

**ii.** Using Blue Ocean, we create a multibranch Jenkins pipeline. From the dashboard, we click on the Blue Ocean plug in, then in the Blue Ocean UI, we click on **`New pipeline`**, we select Github, we choose the php-todo repository and then we click on **"Create pipeline"**

![create multibranch pipeline](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/2fd7068c-669b-4968-ae91-87f6894ff4b6)

**iii.** We launch an ec2 instance for our database server by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance)

![launch db instance](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/47b5f07b-5cae-4e56-aa6e-7135ba7ff3ac)

**iv.** On the database server, we create database and user.

```
Create database homestead;
CREATE USER 'homestead'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON * . * TO 'homestead'@'%';
```

**v.** We use the following code to update the database connectivity requirements in the file **`.env.sample`**

```
DB_CONNECTION=mysql
DB_PORT=3306
```

![envsample](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/d00402a8-f920-4383-96d3-d7d0d8d81634)

**vi.** Then we proceed to add the private IP address of our database server to our ansible inventory in **`dev.yml`**

![db private ip dev inventory](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/69988431-77e4-44b5-90f6-b89cb7d6cc18)

**vii.** Under **`static-assignments`**, we reference the db host and role in the **`uat-webservers.yml`** file.

![static-assignments reference mysql role](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/6d03f3f4-38b7-4911-8291-7a86d84459e0)

**viii.** Then in our **`site.yml`** file in the playbooks folder, we point the configuration to our **`db`** role in **`static-assignments`**

![point siteyml to db](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/fbb06262-c5f9-42f0-8579-060666eb22f3)

**ix.** Then using the following commands, we add, commit and push all our changes to our remote Git repository.

```
$ git add .

$ git commit -m "message"

$ git push
```

![commit and push mysql role](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/920e2691-493c-4aee-9c46-3a94d6af4b6c)

**x.** We navigate back to our Jenkins pipeline and we click on "Scan repository now" and we refresh the Jenkins dashboard.

![scan reposit now](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/8e5ce85b-8fc0-435c-b580-4616d3b5cb1f)

**xi.** Then we click on the **`ansible-config-mgt`** repository, in the repository page we click on **Build with Parameters** and we enter **`dev.yml`** as the inventory path.

![build with parameters dev](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/3e9b5f8b-5e24-417b-8958-cb69d89ff59f)

**xii.** Subsequently we click on the Blue Ocean plugin to view the output of our build.

![mysql playbook successful](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/8dd5d7cc-f13e-44a2-9803-da838aa3db78)

![mysql playbook successful 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/3674933b-8c10-40b2-a0c3-1ec5e3e6284f)

**xiii.** As we can see in the images above, our playbook ran sucessfully, to further confirm if the sql databaase and user was created, we carry out the following steps:

+ We execute the following command to install mysql client on the Jenkins server.

**`$ sudo yum install mysql -y`**

![install mysql client](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/37c8c7cd-53c9-41a6-9a61-9e5682463bf0)

+ We use the user **`homestead`** to initiate a remote connection with the DB Server and then we check to see if the database exists.

```
$ sudo mysql -u homestead -h 172.31.44.66 -p

mysql> show databases;

```

![connect and show databases](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/640baa3a-74fa-4a3b-8bb9-cb1f376f3406)


**xiv.** We update **`Jenkinsfile`** in the **`php-todo`** folder with proper pipeline configuration.

```
pipeline {
    agent any

  stages {

     stage("Initial cleanup") {
          steps {
            dir("${WORKSPACE}") {
              deleteDir()
            }
          }
        }
  
    stage('Checkout SCM') {
      steps {
            git branch: 'main', url: 'https://github.com/darey-devops/php-todo.git'
      }
    }

    stage('Prepare Dependencies') {
      steps {
             sh 'mv .env.sample .env'
             sh 'composer install'
             sh 'php artisan migrate'
             sh 'php artisan db:seed'
             sh 'php artisan key:generate'
      }
    }
  }
}
```

![jenkins file php-todo](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/0492b6f6-e5d5-43a7-890b-e91a40aba92e)

As noticed in the Prepare Dependencies section: 

+ The required file by PHP is .env so we are renaming .env.sample to .env

+ Composer is used by PHP to install all the dependent libraries used by the application

+ php artisan uses the .env file to setup the required database objects
  
**xiv.** Then using the following commands, we add, commit and push all our changes to our remote Git repository.

```
$ git add .

$ git commit -m "mysql save"

$ git push
```

![git push phptodo](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/597cbe31-b671-44e6-beda-4a11e375918d)

**xv.** We move to the Jenkins dashboard and we click on the **`php-todo`** repository.

![jenkins repo php todo](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/f50b9b95-e987-4dd4-a9bb-78644d3a48df)

**xvi.** Then we click on "Scan repository now" and we click on the main branch.

![phptodo scan repo now](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/dc830b49-d448-4ccd-a405-59175c52116c)

**xvii.** Subsequently we click on the Blue Ocean plugin to view the output of our build.

![phptodo pipeline successful](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/4fcf9d2b-c87f-4781-b670-a7f8d0368e19)

**xviii.** Next, we update the **`Jenkinsfile`** to include Unit tests step.

```
    stage('Execute Unit Tests') {
      steps {
             sh './vendor/bin/phpunit'
      }
   }
```

![unit test stage](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c01203c4-9d80-49d3-a957-a5cc62854dbb)

**xix.** For this stage to run successfully, code coverage needs to be enabled in **`php.ini`** by setting 'xdebug.mode' to 'coverage'. We do this by executing the following command.

```
$ sudo nano /etc/php.d/15-xdebug.ini

$ sudo nano /etc/php.ini
```

![xdebug mode coverage](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/743bcaee-fdb4-4ad6-a615-a0bc8b427a45)

![xdebug mode coverage 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e8e8a44a-329b-4cf1-a0e2-c6dcd11bb57a)

**xx.** We repeat steps **xiv.** to **xvii.** and we can see the output of our build in the image below.

![unit test success](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/fd34144a-8afa-41fe-96dd-88723b7a5bee)

### Phase 3 - Code Quality Analysis

This is one of the areas where developers, architects and many stakeholders are mostly interested in as far as product development is concerned. For PHP, the most commonly tool used for code quality analysis is **`phploc`**.

**i.** We proceed by adding the code analysis step in **`Jenkinsfile`**. The output of the data will be saved in **`build/logs/phploc.csv`** file.

```
    stage('Code Analysis') {
      steps {
            sh 'phploc app/ --log-csv build/logs/phploc.csv'

      }
    }
```

**ii.** Next, we plot the data using plot Jenkins plugin. This plugin provides generic plotting (or graphing) capabilities in Jenkins. It will plot one or more single values variations across builds in one or more plots. Plots for a particular job (or project) are configured in the job configuration screen, where each field has additional help information. Each plot can have one or more lines (called data series). After each build completes the plots' data series latest values are pulled from the CSV file generated by phploc.

```
      stage('Plot Code Coverage Report') {
      steps {

            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Lines of Code (LOC),Comment Lines of Code (CLOC),Non-Comment Lines of Code (NCLOC),Logical Lines of Code (LLOC)                          ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'A - Lines of code', yaxis: 'Lines of Code'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Directories,Files,Namespaces', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'B - Structures Containers', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Average Class Length (LLOC),Average Method Length (LLOC),Average Function Length (LLOC)', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'C - Average Length', yaxis: 'Average Lines of Code'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Cyclomatic Complexity / Lines of Code,Cyclomatic Complexity / Number of Methods ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'D - Relative Cyclomatic Complexity', yaxis: 'Cyclomatic Complexity by Structure'      
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Classes,Abstract Classes,Concrete Classes', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'E - Types of Classes', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Methods,Non-Static Methods,Static Methods,Public Methods,Non-Public Methods', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'F - Types of Methods', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Constants,Global Constants,Class Constants', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'G - Types of Constants', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Test Classes,Test Methods', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'I - Testing', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Logical Lines of Code (LLOC),Classes Length (LLOC),Functions Length (LLOC),LLOC outside functions or classes ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'AB - Code Structure by Logical Lines of Code', yaxis: 'Logical Lines of Code'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Functions,Named Functions,Anonymous Functions', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'H - Types of Functions', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Interfaces,Traits,Classes,Methods,Functions,Constants', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'BB - Structure Objects', yaxis: 'Count'

      }
    }
```

**iii.** Then, we bundle the application code into an artifact (archived package) and upload to Artifactory.

```
stage ('Package Artifact') {
    steps {
            sh 'zip -qr php-todo.zip ${WORKSPACE}/*'
     }
    }
```

**iv.** Subsequently, we publish the resulting artifact into Artifactory.

```
stage ('Upload Artifact to Artifactory') {
          steps {
            script { 
                 def server = Artifactory.server 'artifactory-server'                 
                 def uploadSpec = """{
                    "files": [
                      {
                       "pattern": "php-todo.zip",
                       "target": "<name-of-artifact-repository>/php-todo",
                       "props": "type=zip;status=ready"

                       }
                    ]
                 }""" 

                 server.upload spec: uploadSpec
               }
            }
  
        }
```

**v.** Afterwards, we deploy the application to the **`dev`** environment by launching Ansible pipeline.

```
stage ('Deploy to Dev Environment') {
    steps {
    build job: 'ansible-project/main', parameters: [[$class: 'StringParameterValue', name: 'env', value: 'dev']], propagate: false, wait: true
    }
  }
```

**vi.** 
