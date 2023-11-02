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

**i.** In our GitHub repository, we execute the following command to create a new branch that we will call **`dynamic-assignments`**.

**`$ git checkout -b dynamic-assignments`**

**ii.** Next we create a folder and name it **`dynamic-assignments`**

**`$ mkdir dynamic-assignments`**

**iii.** We use the set of commands below to move into the dynamic-assignments folder and create a file named **`env-vars.yml`**

```
$ cd dynamic-assignments

$ touch env-vars.yml
```

**iv.** At this point, our GitHub structure is as shown in the image below:

**v.** Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as servername, ip-address etc., we will need a way to set values to variables per specific environment. For this reason, we proceed to create a folder **`env-vars`** to keep each environment’s variables file.

**`$ mkdir env-vars`**

**vi.** Then, we move into the created **`env-vars`** folder, and for each environment, we create new **`YAML`** files which we will use to set variables.

```
$ cd env-vars

$ touch dev stage uat prod
```

**vi.** Our directory layout now looks as shown in the image below:

**vii.** The next set is for us to paste the following block of instruction into the **`env-vars.yml`** file.

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

#### <br>Step 2: Introducing Dynamic Assignment Into the Structure<br/>
