## ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)

In this project, we will continue working with **`ansible-config-mgt`** repository and make some improvements on our code. We will refactor our Ansible code, create assignments and learn how to use the import functionality. Imports enables the ability to effectively re-use previously created playbooks in a new playbook. In essence, it allows us to organise our tasks and reuse them when necessary.

### <br>Introduction to Code Refactoring<br/>

The goal of this project is to demonstrate how Ansible refactoring works and its usefulness. [Refactoring](https://en.wikipedia.org/wiki/Code_refactoring) is a general term in computer programming. It means making changes to the source code without changing the expected behaviour of the software. The main idea of refactoring is to enhance code readability, increase maintainability and extensibility, reduce complexity and add proper comments without affecting the logic. In this project, we will be making some slight changes to the code but the overall state of the infrastructure shall remain the same. This project shall consist of three parts:

**1.** Refactor Ansible Code.

**2.** Configure UAT Webservers with Roles.

**3.** Reference Webserver Role.

### <br>Refactor Ansible Code <br/>

In the first part of our project, we shall refactor Ansible code by importing other playbooks into **`site.yml`**. However, before we begin, we need to make some changes to our Jenkins Job. With the way it is currently configured, every new change in the codes creates a separate directory which is not very convenient when we want to run some commands from one place. Besides, it consumes space on Jenkins serves with each subsequent change. To enhance it, we shall be introducing a new Jenkins project/job and we will require **`Copy Artifact`** plugin.

#### <br>Step 1: Jenkins Job Enhancement<br/>



