# ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)

In this project you will continue working with ansible-config-mgt repository and make some improvements of your code. Now you need to refactor your Ansible code, create assignments, and learn how to use the imports functionality. Imports allow to effectively re-use previously created playbooks in a new playbook – it allows you to organize your tasks and reuse them when needed.

### Side Self Study: 
For better understanding or Ansible artifacts re-use – read this article (https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse.html)

### Code Refactoring
Refactoring (https://en.wikipedia.org/wiki/Code_refactoring) is a general term in computer programming. It means making changes to the source code without changing expected behaviour of the software. The main idea of refactoring is to enhance code readability, increase maintainability and extensibility, reduce complexity, add proper comments without affecting the logic.

I will move things around a little bit in the code, but the overal state of the infrastructure remains the same. 

## Step 1 – Jenkins job enhancement
Before we begin, let us make some changes to our Jenkins job – now every new change in the codes creates a separate directory which is not very convenient when we want to run some commands from one place. Besides, it consumes space on Jenkins serves with each subsequent change. Let us enhance it by introducing a new Jenkins project/job – we will require Copy Artifact plugin (https://plugins.jenkins.io/copyartifact/)

1. Go to your Jenkins-Ansible server and create a new directory called ansible-config-artifact – we will store there all artifacts after each build.
#### sudo mkdir /home/ubuntu/ansible-config-artifact   

<img width="572" alt="Screen Shot 2023-10-10 at 8 03 33 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/06ebb0b8-284c-437b-9ad3-0c9790df3fda">

2. Change permissions to this directory, so Jenkins could save files there
#### chmod -R 0777 /home/ubuntu/ansible-config-artifact   

<img width="577" alt="Screen Shot 2023-10-10 at 8 08 46 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/1f9a9649-a4ba-490e-8ee2-41a60e263c3e">

3. Go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab search for Copy Artifact and install this plugin without restarting Jenkins

<img width="1279" alt="Screen Shot 2023-10-10 at 8 10 46 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/3007b483-b0a0-4216-90b1-cb8f7c1afed7">

4. Create a new Freestyle project on Jenkins and name it save_artifacts.
5. This project will be triggered by completion of your existing ansible project. Configure it accordingly

<img width="487" alt="Screen Shot 2023-11-01 at 11 42 33 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/9c8d7def-20e1-4fc2-9c91-b21d8ff5b15e">


#### Note: 
You can configure number of builds to keep in order to save space on the server, for example, you might want to keep only last 2 or 5 build results. You can also make this change to your ansible job.   

6. The main idea of save_artifacts project is to save artifacts into /home/ubuntu/ansible-config-artifact directory. To achieve this, create a Build step and choose Copy artifacts from other project, specify ansible as a source project and /home/ubuntu/ansible-config-artifact as a target directory.

![Screen Shot 2023-11-01 at 11 42 58 AM](https://github.com/bodebisi/Darey.io-Projects/assets/132711315/5674c733-2551-4c1e-888c-d78121324c4b)

7. Test your set up by making some change in README.MD file inside your ansible-config-mgt repository (right inside master branch).

<img width="907" alt="Screen Shot 2023-11-01 at 11 37 02 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/e4ef566c-7880-4694-ad75-b8ee3124d5e0">

If both Jenkins jobs have completed one after another – you shall see your files inside /home/ubuntu/ansible-config-artifact directory and it will be updated with every commit to your master branch.

### Note:
But if your build fails in the save_artifact, Then run this commands in the picture below  

<img width="650" alt="Screen Shot 2023-11-01 at 11 36 02 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/267c8042-12c9-43a3-92a6-b63d59869c1d">

Then test the set up again by making change to the master.

<img width="1221" alt="Screen Shot 2023-11-01 at 11 51 03 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/1093e74d-b049-4586-b843-d684ad128b38">

Now your Jenkins pipeline is more neat and clean.

### Step 2 – Refactor Ansible code by importing other playbooks into site.yml

Before starting to refactor the codes, ensure that you have pulled down the latest code from master (main) branch, and created a new branch, name it refactor.

DevOps philosophy implies constant iterative improvement for better efficiency – refactoring is one of the techniques that can be used, but you always have an answer to question "why?". Why do we need to change something if it works well?

In Project 11 you wrote all tasks in a single playbook common.yml, now it is pretty simple set of instructions for only 2 types of OS, but imagine you have many more tasks and you need to apply this playbook to other servers with different requirements. In this case, you will have to read through the whole playbook to check if all tasks written there are applicable and is there anything that you need to add for certain server/OS families. Very fast it will become a tedious exercise and your playbook will become messy with many commented parts. Your DevOps colleagues will not appreciate such organization of your codes and it will be difficult for them to use your playbook.

Most Ansible users learn the one-file approach first. However, breaking tasks up into different files is an excellent way to organize complex sets of tasks and reuse them.

Let see code re-use in action by importing other playbooks.

1. Within playbooks folder, create a new file and name it site.yml – This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, site.yml will become a parent to all other playbooks that will be developed. Including common.yml that you created previously. Dont worry, you will understand more what this means shortly.
2. Create a new folder in root of the repository and name it static-assignments. The static-assignments folder is where all other children playbooks will be stored. This is merely for easy organization of your work. It is not an Ansible specific concept, therefore you can choose how you want to organize your work. You will see why the folder name has a prefix of static very soon.
3. Move common.yml file into the newly created static-assignments folder.
4. Inside site.yml file, import common.yml playbook.

---
- hosts: all
- import_playbook: ../static-assignments/common.yml

The code above uses built in import_playbook Ansible module.(https://docs.ansible.com/ansible/latest/collections/ansible/builtin/import_playbook_module.html) 

Your folder structure should look like this;
<img width="716" alt="Screen Shot 2023-09-04 at 9 30 26 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/43cb2f81-185b-4c4c-897e-e6f7e52dee6a">

5. Run ansible-playbook command against the dev environment

Since you need to apply some tasks to your dev servers and wireshark is already installed – you can go ahead and create another playbook under static-assignments and name it common-del.yml. In this playbook, configure deletion of wireshark utility.

(add pic of the code)
 
update site.yml with - import_playbook: ../static-assignments/common-del.yml instead of common.yml and run it against dev servers:

#### cd /home/ubuntu/ansible-config-mgt/

#### ansible-playbook -i inventory/dev.yml playbooks/site.yaml

Make sure that wireshark is deleted on all the servers by running
#### wireshark --version

Now you have learned how to use import_playbooks module and you have a ready solution to install/delete packages on multiple servers with just one command.

### Step 3 – Configure UAT Webservers with a role ‘Webserver’

We have our nice and clean dev environment, so let us put it aside and configure 2 new Web Servers as uat. We could write tasks to configure Web Servers in the same playbook, but it would be too messy, instead, we will use a dedicated role (https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html) to make our configuration reusable.

1. Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our uat servers, so give them names accordingly – Web1-UAT and Web2-UAT.

#### Tip:
Do not forget to stop EC2 instances that you are not using at the moment to avoid paying extra. For now, you only need 2 new RHEL 8 servers as Web Servers and 1 existing Jenkins-Ansible server up and running.

2. To create a role, you must create a directory called roles/, relative to the playbook file or in /etc/ansible/ directory.

There are two ways how you can create this folder structure:

i. Use an Ansible utility called ansible-galaxy inside ansible-config-mgt/roles directory (you need to create roles directory upfront)
#### mkdir roles
#### cd roles
#### ansible-galaxy init webserver

ii. Create the directory/files structure manually
####  Note:
You can choose either way, but since you store all your codes in GitHub, it is recommended to create folders and files there rather than locally on Jenkins-Ansible server.

The entire folder structure should look like below, but if you create it manually – you can skip creating tests, files, and vars or remove them if you used ansible-galaxy

<img width="739" alt="Screen Shot 2023-09-04 at 9 42 28 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/a65c9fb1-ff95-4e7e-93d3-e84c3a8ae998">

After removing unnecessary directories and files, the roles structure should look like this

<img width="729" alt="Screen Shot 2023-09-04 at 9 43 08 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/b8c35b64-125a-4770-aa27-3103faddece2">

3. Update your inventory ansible-config-mgt/inventory/uat.yml file with IP addresses of your 2 UAT Web servers
### NOTE:
Ensure you are using ssh-agent to ssh into the Jenkins-Ansible instance just as you have done in project 11;

To learn how to setup SSH agent and connect VS Code to your Jenkins-Ansible instance, please see this video:

For Windows users – ssh-agent on windows (https://youtu.be/OplGrY74qog)

For Linux users – ssh-agent on linux (https://youtu.be/OplGrY74qog)

(Pic)

4. In /etc/ansible/ansible.cfg file uncomment roles_path string and provide a full path to your roles directory roles_path    = /home/ubuntu/ansible-config-mgt/roles, so Ansible could know where to find configured roles.

5. It is time to start adding some logic to the webserver role. Go into tasks directory, and within the main.yml file, start writing configuration tasks to do the following:

   Install and configure Apache (httpd service)

   Clone Tooling website from GitHub https://github.com/<your-name>/tooling.git.

   Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.

   Make sure httpd service is started

Your main.yml may consist of following tasks: (pic)

### Step 4 – Reference ‘Webserver’ role

Within the static-assignments folder, create a new assignment for uat-webservers uat-webservers.yml. This is where you will reference the role. (pic)

Remember that the entry point to our ansible configuration is the site.yml file. Therefore, you need to refer your uat-webservers.yml role inside site.yml.

So, we should have this in site.yml (pic)

### Step 5 – Commit & Test

Commit your changes, create a Pull Request and merge them to master branch, make sure webhook triggered two consequent Jenkins jobs, they ran successfully and copied all the files to your Jenkins-Ansible server into /home/ubuntu/ansible-config-mgt/ directory.

Now run the playbook against your uat inventory and see what happens:
#### sudo ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/uat.yml /home/ubuntu/ansible-config-mgt/playbooks/site.yaml

You should be able to see both of your UAT Web servers configured and you can try to reach them from your browser:

http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php (Pic)

#Congratulations!
You have learned how to deploy and configure UAT Web Servers using Ansible imports and roles!
