## ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES

Dynamic assignments in ansible refer to the use of variables whose values are calculated or determined at runtime. This is in contrast to static assignments, where the values of variables are defined and fixed at the time of writing the playbook. In this project, we will continue configuring our UAT Servers whilst learning and practicing new Ansible concepts and modules. We shall discover the flexibility of Ansible with dynamic assignments (include) and  we shall leverage on Ansible pre-built solutons to expand our automation capabilities.

### <br>Introduction to Ansible Dynamic Assignments (include) and Community Roles<br/>

Dynamic assignments are useful in scenarios where the values of variables need to be determined based on certain conditions or inputs. From our work in [Project 12] we can surmise that static assignments make use of the **`import`** ansible module. However, on the other hand, the module that enables dynamic assignments is the **`include`** ansible module.

```
import = Static
include = Dynamic
```

When the **`import`** module is used, all statements are pre-processed at the time playbooks are parsed. This means that when the **`site.yml`** playbook is executed, Ansible will process all the playbooks referenced during the time it is parsing the statements. This also means that, during actual execution, if any statement changes, such statements will not be considered. Hence, it is static.

On the other hand, when **`include`** module is used, all statements are processed only during execution of the playbook. This means that after the statements are parsed, any changes to the statements encountered during execution will be used.Although it is quite notable that static assignments are mostly used for playbooks (due to the fact that Dynamic assignments are less reliable and can be difficult to debug). Dynamic assignments have however proved useful for environment specific variables as will be introduced in this project. This project shall consist of three parts:

**1.** Introducing Dynamic Assignment Into our Structure.

**2.** Update **`site.yml`** with Dynamic Assignments.

### <br>Introducing Dynamic Assignment Into the Structure<br/>

**i.** In our GitHub repository

