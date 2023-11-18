In this project we will introduce dynamic assignments by using include module.
Now you may be wondering, what is the difference between static and dynamic assignments?
Well, from Project 12, you can already tell that static assignments use import Ansible module. The module that enables dynamic assignments is include.
Hence,

import = Static
include = Dynamic

When the import module is used, all statements are pre-processed at the time playbooks are parsed (https://en.wikipedia.org/wiki/Parsing). Meaning, when you execute site.yml playbook, Ansible will process all the playbooks referenced during the time it is parsing the statements. This also means that, during actual execution, if any statement changes, such statements will not be considered. Hence, it is static.
On the other hand, when include module is used, all statements are processed only during execution of the playbook. Meaning, after the statements are parsed, any changes to the statements encountered during execution will be used.
Take note that in most cases it is recommended to use static assignments for playbooks, because it is more reliable. With dynamic ones, it is hard to debug playbook problems due to its dynamic nature. However, you can use dynamic assignments for environment specific variables as we will be introducing in this project.

## Introducing Dynamic Assignment Into Our structure

In your https://github.com/<your-name>/ansible-config-mgt GitHub repository start a new branch and call it dynamic-assignments.
Create a new folder, name it dynamic-assignments. Then inside this folder, create a new file and name it 
env-vars.yml. We will instruct site.yml to include this playbook later. For now, let us keep building up the structure.

#### Your GitHub shall have following structure by now.
Depending on what method you used in the previous project you may have or not have roles folder in your GitHub repository - if you used ansible-galaxy, then roles directory was only created on your Jenkins-Ansible server locally. It is recommended to have all the codes managed and tracked in GitHub, so you might want to recreate this structure manually in this case - it is up to you.

<img width="635" alt="Screen Shot 2023-11-15 at 10 54 00 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/b4a9ca3a-3ae5-4096-9e4f-85a7454efd15">

Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as servername, ip-address etc., we will need a way to set values to variables per specific environment.
For this reason, we will now create a folder to keep each environment's variables file. Therefore, create a new folder env-vars, then for each environment, create new YAML files which we will use to set variables.
Your layout should now look like this.

<img width="246" alt="Screen Shot 2023-11-18 at 8 30 43 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/e6a4d2cb-104e-40e5-b1d6-2d2044a0530c">

Now paste the instruction below into the env-vars.yml file. 

<img width="686" alt="Screen Shot 2023-11-18 at 8 34 02 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/9bd1c2e6-6be2-49c5-bcfd-e1dfba9bae8b">

#### Notice 3 things here:
1. We used include_vars syntax instead of include, this is because Ansible developers decided to separate different features of the module. From Ansible version 2.8, the include module is deprecated and variants of include_* must be used. These are:

include role https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_role_module.html#include-role-module

include tasks  https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_tasks_module.html#include-tasks-module

include vars https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_vars_module.html#include-vars-module

### In the same version, variants of import were also introduces, such as:
import_role
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/import_role_module.html#import-role-module

import_tasks
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/import_tasks_module.html#import-tasks-module

2. We made use of a special variables (https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html)
{{ playbook_dir }} and {{ inventory_file }}. {{ playbook_dir }} will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem. {{ inventory_file }} on the other hand will dynamically resolve to the name of the inventory file being used, then append .yml so that it picks up the required file within the env-vars folder.

3. We are including the variables using a loop. with_first_found implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment specific env file does not exist.

## Update site.yml with dynamic assignments

Update site.yml file to make use of the dynamic assignment. (At this point, we cannot test it yet. We are just setting the stage for what is yet to come. So hang on to your hats)

site.yml should now look like this 

<img width="737" alt="Screen Shot 2023-11-18 at 8 35 40 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/22810591-6cb9-45ec-80d0-9f1bdaa9a24f">

<img width="563" alt="Screen Shot 2023-11-18 at 8 36 59 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/b1de2804-c028-4ca4-aac2-cc9f411c46a3">

### Community Roles
Now it is time to create a role for MySQL database - it should install the MySQL package, create a database and configure users. But why should we re-invent the wheel? There are tons of roles that have already been developed by other open source engineers out there. These roles are actually production ready, and dynamic to accomodate most of Linux flavours. With Ansible Galaxy again, we can simply download a ready to use ansible role, and keep going.

### Download Mysql Ansible Role
You can browse available community roles (https://galaxy.ansible.com/home)
We will be using a MySQL role developed by geerlingguy. (https://galaxy.ansible.com/geerlingguy/mysql)

Hint: To preserve your your GitHub in actual state after you install a new role - make a commit and push to master your 'ansible-config-mgt' directory. Of course you must have git installed and configured on Jenkins-Ansible server and, for more convenient work with codes, you can configure Visual Studio Code to work with this directory. In this case, you will no longer need webhook and Jenkins jobs to update your codes on Jenkins-Ansible server, so you can disable it - we will be using Jenkins later for a better purpose.

On Jenkins-Ansible server make sure that git is installed with git --version, then go to 'ansible-config-mgt' directory and run (pic)
git init
git pull https://github.com/<your-name>/ansible-config-mgt.git
git remote add origin https://github.com/<your-name>/ansible-config-mgt.git
git branch roles-feature
git switch roles-feature

Inside roles directory create your new MySQL role with ansible-galaxy install geerlingguy.mysql and rename the folder to mysql

mv geerlingguy.mysql/ mysql (Pic)

Read README.md file, and edit roles configuration to use correct credentials for MySQL required for the tooling website. (Pic)

#### Now it is time to upload the changes into your GitHub: (Pic)

git add .
git commit -m "Commit new role files into GitHub"
git push --set-upstream origin roles-feature

Now, if you are satisfied with your codes, you can create a Pull Request and merge it to main branch on GitHub.

## Load Balancer roles
We want to be able to choose which Load Balancer to use, Nginx or Apache, so we need to have two roles respectively:

1. Nginx
2. Apache

With your experience on Ansible so far you can:
Decide if you want to develop your own roles, or find available ones from the community
Update both static-assignment and site.yml files to refer the roles

### Important Hints:
Since you cannot use both Nginx and Apache load balancer, you need to add a condition to enable either one - this is where you can make use of variables.

Declare a variable in defaults/main.yml file inside the Nginx and Apache roles. Name each variables enable_nginx_lb and enable_apache_lb respectively.

Set both values to false like this enable_nginx_lb: false and enable_apache_lb: false.

Declare another variable in both roles load_balancer_is_required and set its value to false as well

Update both assignment and site.yml files respectively

loadbalancers.yml file (Pic)

site.yml file (pic)

Now you can make use of env-vars\uat.yml file to define which loadbalancer to use in UAT environment by setting respective environmental variable to true.

You will activate load balancer, and enable nginx by setting these in the respective environment's env-vars file. (Pic)

The same must work with apache LB, so you can switch it by setting respective environmental variable to true and other to false.

To test this, you can update inventory for each environment and run Ansible against each environment.

# Congratulations!

You have learned and practiced how to use Ansible configuration management tool to prepare UAT environment for Tooling web solution.




