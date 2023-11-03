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

push and create pull request

### <br>Making use of Community Roles<br/>

Now it is time to create a role for MySQL database which will perform the functions of installing the MySQL package, creating a database and configuring users. It is important to note that there are tons of roles that have already been developed by other open source engineers out there. These roles are actually production ready, and dynamic to accomodate most of Linux flavours. So rather than re-inventing the wheel, we simply make use of Ansible Galaxy again and we can simply download a ready to use ansible role.

#### <br>Step 1: Download Mysql Ansible Role<br/>

**i.** We proceed to browse available community roles [here](https://galaxy.ansible.com/ui/). We decide to use a [MySQL role developed by](https://galaxy.ansible.com/ui/standalone/roles/geerlingguy/mysql/) **`geerlingguy`**.

**ii.** On Jenkins-Ansible server make sure that git is installed with **`git --version`**, and then we go to the **`ansible-config-mgt`** directory and run the following commands:

```
$ git init
$ git pull https://github.com/<your-name>/ansible-config-mgt.git
$ git remote add origin https://github.com/<your-name>/ansible-config-mgt.git
$ git branch roles-feature
$ git switch roles-feature
```

**iii.** Inside the **`roles`** directory, we create a new MySQL role with ansible-galaxy and then we rename the folder to **`mysql`**:

```
ansible-galaxy install geerlingguy.mysql
mv geerlingguy.mysql/ mysql
```

**iv.** 
