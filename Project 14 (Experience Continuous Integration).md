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

To deploy to other environments, we will need to use parameters.

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

2. Update `Jenkinsfile` to introduce parameterization. Below is just one parameter. It has a default value in case if no value is specified at execution. It also has a description so that everyone is aware of its purpose.

```
pipeline {
    agent any

    parameters {
      string(name: 'inventory', defaultValue: 'dev',  description: 'This is the inventory file for the environment to deploy configuration')
    }
...
```

3. In the Ansible execution section, remove the hardcoded `inventory/dev` and replace with `${inventory}

From now on, each time you hit on execute, it will expect an input.

<img src="https://darey-io-nonprod-pbl-projects.s3.eu-west-2.amazonaws.com/project14/Jenkins-Parameter.png" width="936px" height="550px">

Notice that the default value loads up, but we can now specify which environment we want to deploy the configuration to. Simply type `sit` and hit **Run**

<img src="https://darey-io-nonprod-pbl-projects.s3.eu-west-2.amazonaws.com/project14/Jenkins-Parameter-Sit.png" width="936px" height="550px">

4. Add another parameter. This time, introduce `tagging` in Ansible. You can limit the Ansible execution to a specific role or playbook desired.  Therefore, add an Ansible tag to run against `webserver` only. Test this locally first to get the experience. Once you understand this, update `Jenkinsfile` and run it from Jenkins.
