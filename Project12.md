## ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)

In this project, we will continue working with **`ansible-config-mgt`** repository and make some improvements on our code. We will refactor our Ansible code, create assignments and learn how to use the import functionality. Imports enables the ability to effectively re-use previously created playbooks in a new playbook. In essence, it allows us to organise our tasks and reuse them when necessary.

### <br>Introduction to Code Refactoring<br/>

The goal of this project is to demonstrate how Ansible refactoring works and its usefulness. [Refactoring](https://en.wikipedia.org/wiki/Code_refactoring) is a general term in computer programming. It means making changes to the source code without changing the expected behaviour of the software. The main idea of refactoring is to enhance code readability, increase maintainability and extensibility, reduce complexity and add proper comments without affecting the logic. In this project, we will be making some slight changes to the code but the overall state of the infrastructure shall remain the same. This project shall consist of three parts:

**1.** Refactor Ansible Code.

**2.** Configure UAT Webservers with Roles.

**3.** Reference Webserver Role.

### <br>Refactor Ansible Code <br/>

In the first part of our project, we shall refactor Ansible code by importing other playbooks into **`site.yml`**. However, before we begin, we need to make some changes to our Jenkins Job. 

#### <br>Step 1: Jenkins Job Enhancement<br/>

With the way our Jekins job is currently configured, every new change in the codes creates a separate directory which is not very convenient when we want to run some commands from one place. Besides, it consumes space on Jenkins serves with each subsequent change. To enhance it, we shall be introducing a new Jenkins project/job and we will require **`Copy Artifact`** plugin.

**i.** In our **`Jenkins-Ansible`** server, we use the comand below to create a new directory called **`ansible-config-artifact`** – which we will be using to store all artifacts after each build.

**` $ sudo mkdir /home/ubuntu/ansible-config-artifact`**

**ii.** Then with the following command, we change permissions for this directory, so that Jenkins would be able to save files there:

**` $ chmod -R 0777 /home/ubuntu/ansible-config-artifact`**

![mkdir-chmod](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9b18781a-c240-4219-a69c-7650aef46f9e)

**iii.** Next, we will need to install the **`copy artifact`** plugin without restarting Jenkins:

+ From the main Jenkins Environment, we click on **"Manage Jenkins"**.

 ![manage jenkins](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/490e07d7-4916-493e-bb16-f541914e99ff)

+ On the System Configuration page, we click on **"Plugins"**.

 ![plugins](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9f23b0ef-46e5-43ae-a876-ffb499a7896d)

+ On the Plugins page, we click on Available plugins, then we type **`copy artifact`** in the search box.

 ![copy artifact plugin](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/853dd4bf-213f-4ced-afc2-551766f50411)

+ From here, we can see the **`copy artifact`** plugin we wish to install, so we check the **"Install"** checkbox beside it and we click on the **"Install"** button. After doing this we will be able to see the download progress page.

![download progress](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/092efd2f-0771-4987-ad3a-de3c189c547f)

**iv.** Next, we create a new Freestyle project (as we did in [Project 11](https://github.com/QBDev0ps/DevOps-Cloud-projects/blob/main/Project11.md)) and we name it **`save_artifacts`**:

+ From the Jenkins web console, we click on **"New item"**

![jenkins new item](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/6dd5ad81-1d92-4908-813f-eecce8717871)

+ In the next page under **"Enter an item name"** we type in **`save_artifacts`**, then we select **"Freestyle project"** and we click on **"Ok"** at the bottom of the page.

![save-artifacts](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a6d3b084-9bcf-4cba-a2cd-c99d4680b78a)

**v.** This project will be triggered by the completion of our existing **`ansible`** project. So therefore we configure it accordingly:

+ In the Jenkins configuration page, under **"General"** we click on check box for **"Discard old builds"** and under **"Max # of builds to keep"** we enter our desired number.

![discard old builds](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e356d83f-2a65-43ff-a46c-0525726eafad)

+ Next under **"Build Triggers"**, we click on the check box beside **"Build after other projects are built"** and under the **"Projects to watch"** dialogue box we type in **`ansible`**.

![build trigger](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4342fb71-c919-4146-9d71-f6f62837b38a)

**vi.** The main idea of the **`save_artifacts`** project is to save artifacts into the **`/home/ubuntu/ansible-config-artifact`** directory. To achieve this, we proceed as follows:

+ Under **"Build Steps"** we click on the **"Add build step"** drop down button and we select **"Copy artifacts from another project"**.

 ![build steps](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/8e593233-ab61-4346-9968-8aaa52f0f646)

+ Under **"Project name"**, we specify **`ansible`** as the source project, under **"Artifacts to copy"**, we input __**__ to copy all artifacts and then we put in  **`/home/ubuntu/ansible-config-artifact`** as the **"Target directory"**.

 ![copy artifacts](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/1e887922-96f9-4ed9-b8be-334a8b01dbaa)

+ Then we click on **"Apply"** and **"Save"** at the bottom of the page.

![apply and save](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ab54ff0a-83b4-41cd-b599-f11164d6e3a6)

**vii.** The next thing we do is to test our set up by making some changes in the **README.MD** file inside our **`ansible-config-mgt`** repository (right inside the **`master`** branch). If both Jenkins jobs have completed one after another – we shall see our files inside the **`/home/ubuntu/ansible-config-artifact`** directory and it will be updated with every commit to our **`master`** branch.

![console output](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b56eac24-c25a-4fa9-862d-624168008446)

As seen in the above image, the **`save_artifacts`** job was successfully triggered and completed. To confirm that our files are inside the **`/home/ubuntu/ansible-config-artifact`** directory, we execute the following command:

**` $ ls /home/ubuntu/ansible-config-artifact`**

![confirmation ansible artifacts](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a1a1831f-57de-491d-997a-c0519e88061e)

#### <br>Step 2: Refactor Ansible code by importing other playbooks into `site.yml`<br/>

**i.** Before starting to refactor the codes, we need to pull down the latest code from the **`main`** branch, and create a new branch, which we will name **`refactor`**:

+ From the VS code GUI in the **`main`** branch of our ansible-config-mgt repository, we click on the **"Views and More Actions"** button then we select **"pull"** to pull down the latest code from the **`main`** branch.

 ![pull latest code](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/19605205-1fa8-4317-9248-f13ace4be134)

+ From the VS Code environment we go to the bottom of the page and we click on **`main`**.

+ ![create new branch](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/659f1c32-4fa4-4d80-a97f-1c53bb1d70a1)

+ Afterwards, we enter the new branch name **`refactor`** and we press **Enter**.

![branch name](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ef38ba25-19e3-4420-bda2-63182009bd1a)

+ Then we subsequently click on the **"Publish Branch"** button.

 ![publish branch](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f3a6dad0-134c-40dd-a3b8-f7c37dba12ac)

**ii.** Within playbooks folder, create a new file and name it site.yml – This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, site.yml will become a parent to all other playbooks that will be developed. Including common.yml that you created previously.
