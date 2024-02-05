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
  
2. Security groups: For the purposes of this project, we can have one security group that is open to all traffic. This should however not be attempted in a real DevOps enviroment.
  
3. Our Ansible inventory is expected to look like this:

├── ci
├── dev
├── pentest
├── pre-prod
├── prod
├── sit
└── uat

**`ci`** inventory file

[jenkins]
<Jenkins-Private-IP-Address>

[nginx]
<Nginx-Private-IP-Address>

[sonarqube]
<SonarQube-Private-IP-Address>

[artifact_repository]
<Artifact_repository-Private-IP-Address>

