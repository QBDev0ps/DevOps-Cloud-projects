## ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES

[Dynamic assignments](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse.html#includes-dynamic-re-use) in ansible refer to the use of variables whose values are calculated or determined at runtime. This is in contrast to static assignments, where the values of variables are defined and fixed at the time of writing the playbook. In this project, we will continue configuring our UAT Servers whilst learning and practicing new Ansible concepts and modules. We shall discover the flexibility of Ansible with dynamic assignments (include) and  we shall leverage on Ansible pre-built solutons to expand our automation capabilities.

### <br>Introduction to Ansible Dynamic Assignments (include) and Community Roles<br/>

Dynamic assignments are useful in scenarios where the values of variables need to be determined based on certain conditions or inputs. From our work in [Project 12](https://github.com/QBDev0ps/DevOps-Cloud-projects/blob/main/Project12.md) we can surmise that static assignments make use of the **`import`** ansible module. However, on the other hand, the module that enables dynamic assignments is the **`include`** ansible module.

```
import = Static
include = Dynamic
```

When the **`import`** module is used, all statements are pre-processed at the time playbooks are parsed. This means that when the **`site.yml`** playbook is executed, Ansible will process all the playbooks referenced during the time it is parsing the statements. This also means that, during actual execution, if any statement changes, such statements will not be considered. Hence, it is static.

On the other hand, when **`include`** module is used, all statements are processed only during execution of the playbook. This means that after the statements are parsed, any changes to the statements encountered during execution will be used.Although it is quite notable that static assignments are mostly used for playbooks (due to the fact that Dynamic assignments are less reliable and can be difficult to debug). Dynamic assignments have however proved useful for environment specific variables as will be introduced in this project. This project shall consist of two parts:

**1.** Making use of Dynamic Assignments.

**2.** Making use of Community Roles.

### <br>Making use of Dynamic Assignments<br/>

#### <br>Step 1: Introducing Dynamic Assignment Into the Structure<br/>

**i.** In our GitHub repository, we execute the following command to create and move into a new branch that we will call **`dynamic-assignments`**.

**`$ git checkout -b dynamic-assignments`**

![git checkout -b](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/8da27569-6e77-4652-9439-0f970b4135f4)

**ii.** Next we create a folder and name it **`dynamic-assignments`**

**`$ mkdir dynamic-assignments`**

![mkdir dynamic assignments](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/075643a0-5b53-48ef-a951-00ce266c5fb6)

**iii.** We use the set of commands below to move into the dynamic-assignments folder and create a file named **`env-vars.yml`**

```
$ cd dynamic-assignments

$ touch env-vars.yml
```

![cd touch env-vars](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/49eea68b-0195-497f-8240-176b35354bcc)

**iv.** At this point, our GitHub directory structure is as shown in the image below:

![directory structure 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/af3e864d-ac1f-4074-afb6-a20ed5936b1c)

**v.** Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as servername, ip-address etc., we will need a way to set values to variables per specific environment. For this reason, we proceed to create a folder **`env-vars`** to keep each environment’s variables file.

**`$ mkdir env-vars`**

![mkdir env-vars](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/43692f09-2d1f-438e-80dc-cacc2d5f2fac)

**vi.** Then, we move into the created **`env-vars`** folder, and for each environment, we create new **`YAML`** files which we will use to set variables.

```
$ cd env-vars

$ touch dev.yml stage.yml uat.yml prod.yml
```

![cd env-vars and create files](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/00de21e8-3e20-4fdc-ab9e-712a0a099c91)

**vi.** Our directory layout now looks as shown in the image below:

![directory structure 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e97bad3a-856d-47fe-9049-50a67396e8d1)

**vii.** Now satisfied with our folder structure, we move ahead and paste the following block of instruction into the **`env-vars.yml`** file.

```
---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
    - name: looping through list of available files
      include_vars: "{{ item }}"
      with_first_found:
        - files:
            - dev.yml
            - stage.yml
            - prod.yml
            - uat.yml
          paths:
            - "{{ playbook_dir }}/../env-vars"
      tags:
        - always
```

![env-vars-yml config](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f251f135-e39a-4ba6-8b02-d9db467fef96)

**viii.** There are three things to note about the configuration above in **vii.**

1. We used **`include_vars`** syntax instead of **`include`**, this is because Ansible developers decided to separate different features of the module. From Ansible version **2.8**, the **`include module`** is deprecated and variants of **`include_*`** must be used. These are:

+ [include_role](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_role_module.html#include-role-module)
  
+ [include_tasks](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_tasks_module.html#include-tasks-module)
  
+ [include_vars](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_vars_module.html#include-vars-module)

  
In the same version, variants of **`import`** were also introduced, such as:

+ [import_role](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/import_role_module.html#import-role-module)
  
+ [import_tasks](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/import_tasks_module.html#import-tasks-module)
  
2. We made use of [special variables](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html) **`{{ playbook_dir }}`** and **`{{ inventory_file }}`**. **`{{ playbook_dir }}`** will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem. **`{{ inventory_file }}`** on the other hand will dynamically resolve to the name of the inventory file being used, then append **`.yml`** so that it picks up the required file within the **`env-vars`** folder.

3. We are including the variables using a loop. **`with_first_found`** implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment specific env file does not exist.

#### <br>Step 2: Update `site.yml` with Dynamic Assignments<br/>

The next step is to update the **`site.yml`** file to make use of the dynamic assignment. (It is however important to note that we cannot test it just yet. Rather, this is us preparing the stage for what is to come.) **`site.yml`** should look like this:

```
---
- hosts: all
- name: Include dynamic variables 
  tasks:
  import_playbook: ../static-assignments/common.yml 
  include: ../dynamic-assignments/env-vars.yml
  tags:
    - always

-  hosts: webservers
- name: Webserver assignment
  import_playbook: ../static-assignments/webservers.yml
```

![site-yml config](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/0e49a73d-e805-4f00-b778-1f5a58ef01d2)

#### <br>Step 3: Update Git with Latest Code<br/> 

At this point, we we need to push all the changes we made locally to our remote Github repository.

**i.** We use the following commands to stage, commit and push our branch to GitHub:

```
$ git status

$ git add <selected files>

$ git commit -m "commit message"

$ git push --set-upstream origin dynamic-assignments
```

![git push set upstream](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/753db294-3be8-4b12-8abb-3a911d8148e9)

**ii.** The next thing we do is to create a **Pull Request** in GitHub by following [these steps:](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request) 

+ From the **`ansible-config-mgt`** repository page, we click on the **"Pull requests"** tab and then in the next page we click on the **"New pull request"** button.

![pull requests](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a27f564c-53f7-49c8-9905-aa7e440fc17b)

![new pull requests](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/46e0162b-0cb7-4ff4-8514-6fd0c8ab0a7c)

+ This takes us to the **"Compare changes"** page where we choose the **`dynamic-assignments`** branch to set up a comparison with the **`main`** branch.

![compare changes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/6eddc09c-5460-4d07-a11b-ac6e38daaeb0)

+ Once we set up the comparisons between the **`main`** and the **`dynamic-assignments`** branch, we then proceed to click on the **"Create pull request"** button.

![comparing changes create pull request](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/85e25bdb-6afc-4a40-a644-73389a193d5d)

+ In the next page, we input a pull request message inside the dialogue box and we click on the  **"Create pull request"** button.

![open a pull request](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f8b48d19-4265-445b-a576-615bd6b7b371)

**iii.** Now as shown in the image below, we act as a reviewer and we examine the changes in the **`dynamic-assignments`** branch and check for conflicts with the **`main`** branch.

![merge pull request 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/272752ff-9315-44c9-9987-003e6904c9d1)

**iv.** As we are satisfied and happy with the changes made in **`dynamic-assignments`**, we click on **"Merge pull request"** and then we click on **"Confirm merge"**

![confirm merge](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/cca3dc74-e690-4243-b32f-166f5f994e9f)

**v.** This takes us to the next page which shows that **`dynamic-assignments`** has been successfully merged to **`main`** branch.

![merge successful](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2a2402b5-1279-452c-8d71-6e7983600ca5)

### <br>Making use of Community Roles<br/>

Now it is time to create a role for MySQL database which will perform the functions of installing the MySQL package, creating a database and configuring users. It is important to note that there are tons of roles that have already been developed by other open source engineers out there. These roles are actually production ready, and dynamic to accomodate most of Linux flavours. So rather than re-inventing the wheel, we simply make use of Ansible Galaxy again and we can simply download a ready to use ansible role.

#### <br>Step 1: Download Mysql Ansible Role<br/>

**i.** We proceed to browse available community roles [here](https://galaxy.ansible.com/ui/). We decide to use a [MySQL role developed by](https://galaxy.ansible.com/ui/standalone/roles/geerlingguy/mysql/) **`geerlingguy`**.

**ii.** For now, we no longer need webhooks and Jenkins to update our codes on the **`Jenkins-Ansible`** server, so we disable it by clicking on **"Disable Project"** in the project status page in our Jenkins environment.

![disable project](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e8476a18-db68-4167-8304-8bc72699feb3)

**ii.** Next, we head to our Jenkins-Ansible server and then we go to the **`ansible-config-mgt`** directory and we checkout from the **`dynamic-assignments`** branch into the main branch, and then we pull down the latest changes.

```
$ git checkout main

$ git pull
```

![git checkout main git pull](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9e766003-76a0-4292-977c-d2a0103a2f62)

**iii.** Then we execute the following command to create and move into a new branch that we will call **`roles-feature`**.

**`$ git checkout -b roles-feature`**

![git-checkout-b roles-feature](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/bccf8b97-bf72-4af3-a2f1-73b5bb30bfc7)

**iv.** Inside the **`roles`** directory, we create a new MySQL role with ansible-galaxy and then we rename the folder to **`mysql`**:

```
$ ansible-galaxy install geerlingguy.mysql

$ mv geerlingguy.mysql/ mysql
```

![ansible galaxy geerling guy](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/003e1e95-c811-4f29-9c93-ddd11c8180bf)


#### <br>Step 2: Configure Mysql Ansible Role<br/>

**i.** The next thing we do is to go through the **README.md** file, and edit roles configuration to use the correct credentials for MySQL required for the **`tooling`** website. We navigate via VS Code explorer through **`roles > mysql > defaults > main.yml`** and we edit configuration in the **`main.yml`** file as shown in the image below:

![main-yml edit](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5d778f8a-33f3-47d9-a736-f8af44d8ba05)

**ii.** To reference, the database role, we do the following:

+ Go into the **README.md** file and copy the following block of code:

  ```
  ---
  - hosts: database
    roles:
    - role: geerlingguy.mysql
      become: yes
  ```

+ Then we move into the **`static-assignments`** folder and create a file we name **`db.yml`**

 ```
$ cd static-assignments

$ touch db.yml
```

![cd and touch db-yml](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/63ed4dcd-7b8d-4fa3-9804-5357c963279a)

+ Then we enter the file via VS code explorer and we paste in the configuration as shown in the image below:

![reference db-yml role](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/3d1151c1-7006-4f20-a829-d835362ce739)

**iii.** We also need to reference the **`db.yml`** role inside **`site.yml`**. So, we navigate via the VS Code file explorer and put the following configuration inside site.yml:

```
-  hosts: db
- name: Installing db
  import_playbook: ../static-assignments/db.yml
```

![db-yml in site-yml](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/804f1ce1-9efa-4893-89d0-96803d6de96b)

**iv.** We proceed to update the inventory **`ansible-config/inventory/dev.yml`** file with the Private IP address of our DB server using the following syntax:

```
[db]
<db-Server-Private-IP-Address> ansible_ssh_user=ubuntu
```

![dev-yml config](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5c242a60-a8ad-41b2-b490-e5edc7f9b84e)


#### <br>Step 3: Commit and Test DB Playbook<br/>

In this step, we shall stage and commit our changes in the **`roles-feature`** branch. then we shall test run the db playbook. We decided to do this with the consideration that it would be cleaner and less cumbersome rather than executing it with the other roles we are going to download and configure later on in this project. It will also be easier to debug and investigate any potential errors our playbook might encounter.

**i.** Using the following set of commands, we add all untracked files to the staging area and then we commit the changes:

```
$ git add .

$ git commit -m "saved all updates to role-features branch"
```

![commit to roles-feature branch](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7d3051cd-6b9c-40d8-afca-d1944498de7f)

**ii.** The next step is to ensure connection to our **`Jenkins-Ansible`** server via **`ssh-agent`** as we did in [Project 11](https://github.com/QBDev0ps/DevOps-Cloud-projects/blob/main/Project11.md) and then with the commands below, we run the playbook against our **`dev inventory`**.

```
$ cd /home/ubuntu/ansible-config-mgt

$ ansible-playbook -i /inventory/dev.yml playbooks/site.yml
```

#### <br>Step 4: Load Balancer Roles<br/>

We want to be able to make a choice on which Load Balancer to use, Nginx or Apache, so we need to have two roles respectively: Nginx and Apache.

**i.** We refer to what we did earlier with **`msql`** role and we do the same for both nginx and apache load balancer roles. Inside the **`roles`** directory, we create new roles for both nginx and apache load balancers with ansible-galaxy. And then we rename the folders to **`nginx-lb`** and  **`apache-lb`** respectively.

```
$ ansible-galaxy install geerlingguy.nginx

$ mv geerlingguy.nginx/ nginx-lb

$ ansible-galaxy install geerlingguy.apache

$ mv geerlingguy.apache/ apache-lb
```

