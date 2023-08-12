In this Project, we want to introduce a set of DevOps tools that will help our team in day to day activities in managing, developing, testing, deploying and monitoring different projects.

The tools we want our team to be able to use are well known and widely used by multiple DevOps teams, so we will introduce a single DevOps Tooling Solution that will consist of:

#### Jenkins – free and open source automation server used to build CI/CD pipelines.
#### Kubernetes – an open-source container-orchestration system for automating computer application deployment, scaling, and management.
#### Jfrog Artifactory – Universal Repository Manager supporting all major packaging formats, build tools and CI servers. Artifactory.
#### Rancher – an open source software platform that enables organizations to run and manage Docker and Kubernetes in production.
#### Grafana – a multi-platform open source analytics and interactive visualization web application.
#### Prometheus – An open-source monitoring system with a dimensional data model, flexible query language, efficient time series database and modern alerting approach.
#### Kibana – Kibana is a free and open user interface that lets you visualize your Elasticsearch data and navigate the Elastic Stack.

## Side Self Study
Read about Network-attached storage (NAS), Storage Area Network (SAN) and related protocols like NFS, (s)FTP, SMB, iSCSI. Explore what Block-level storage is and how it is used by Cloud Service providers, know the difference from Object storage.
On the example of AWS services understand the difference between Block Storage, Object Storage and Network File System.

In this project you will implement a solution that consists of following components:

Infrastructure: AWS
Webserver Linux: Red Hat Enterprise Linux 8
Database Server: Ubuntu 20.04 + MySQL
Storage Server: Red Hat Enterprise Linux 8 + NFS Server
Programming Language: PHP
Code Repository: GitHub

## Step 1 – Prepare NFS Server
Spin up 4 new EC2 instance with RHEL Linux 8 Operating System. (one instance for your NFS server and the remaining 3 for webservers 1,2 & 3)
Also spin up an additional instance with Ubuntu operating system to serve as the database server) 
Create abd attach 3 volumes to the NFS server. 
### use lsblk 
To confirm the attched volumes when you connect the instance to your terminal

Based on your LVM experience from Project 6, Configure LVM on the Server.
### sudo gdisk /dev/xvdf
### sudo gdisk /dev/xvdg
### sudo gdisk /dev/xvdh
### lsblk to confirm 
![Screen Shot 2023-08-12 at 11 30 44 PM](https://github.com/bodebisi/Darey.io-Projects/assets/132711315/8d0c50c8-a071-46b7-b1d2-871c18eb913d)

Run the LVM package 
### sudo yum install lvm2 -y
### sudo lvmdiskscan

Now create the physical volumes with pvcreate
### sudo pvcreate /dev/xvdf1 
### sudo pvcreate /dev/xvdg1
### sudo pvcreate /dev/xvdh1
### sudo pvs (to confirm)

Now create the volume groups with vgcreate 
### sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
### sudo vgs to (confirm)

Ensure there are 3 Logical Volumes. lv-opt lv-apps, and lv-logs
### sudo lvcreate -n lv-apps -L 9G webdata-vg
### sudo lvcreate -n lv-logs -L 9G webdata-vg
### sudo lvcreate -n lv-opt -L 9G webdata-vg
### sudo lvs to confirm
### lsblk to see the attachments
![Screen Shot 2023-08-12 at 11 53 05 PM](https://github.com/bodebisi/Darey.io-Projects/assets/132711315/f4239ddd-57ee-4d88-92cd-da9b999cac07)

Verify the entire setup 
### sudo vgdisplay -v #view complete setup - VG, PV, and LV

Instead of formating the disks as ext4 you will have to format them as xfs
### sudo mkfs -t xfs /dev/webdata-vg/lv-apps
### sudo mkfs -t xfs /dev/webdata-vg/lv-logs
### sudo mkfs -t xfs /dev/webdata-vg/lv-opt

Create mount points on /mnt directory for the logical volumes as follow:
### sudo mkdir /mnt/apps
### sudo mkdir /mnt/logs
### sudo mkdir /mnt/opt

Mount lv-apps on /mnt/apps – To be used by webservers
### sudo mount /dev/webdata-vg/lv-apps /mnt/apps/
Mount lv-logs on /mnt/logs – To be used by webserver logs
### sudo mount /dev/webdata-vg/lv-logs /mnt/logs/
Mount lv-opt on /mnt/opt – To be used by Jenkins server in Project 8
### sudo mount /dev/webdata-vg/lv-opt /mnt/opt/

Install NFS server, configure it to start on reboot and make sure it is u and running
### sudo yum -y update
### sudo yum install nfs-utils -y
### sudo systemctl start nfs-server.service
### sudo systemctl enable nfs-server.service
### sudo systemctl status nfs-server.service

Export the mounts for webservers’ subnet cidr to connect as clients. For simplicity, you will install your all three Web Servers inside the same subnet, but in production set up you would probably want to separate each tier inside its own subnet for higher level of security.

To check your subnet cidr – open your EC2 details in AWS web console and locate ‘Networking’ tab and open a Subnet link:
        
Make sure we set up permission that will allow our Web servers to read, write and execute files on NFS:
### sudo chown -R nobody: /mnt/apps
### sudo chown -R nobody: /mnt/logs
### sudo chown -R nobody: /mnt/opt

### sudo chmod -R 777 /mnt/apps
### sudo chmod -R 777 /mnt/logs
### sudo chmod -R 777 /mnt/opt

### sudo systemctl restart nfs-server.service

Configure access to NFS for clients within the same subnet (example of Subnet CIDR – 172.31.32.0/20 ):

### sudo vi /etc/exports

### /mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
### /mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
### /mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)

Esc + :wq!

### sudo exportfs -arv

Check which port is used by NFS and open it using Security Groups (add new Inbound Rule)
### rpcinfo -p | grep nfs
      

## Important note: In order for NFS server to be accessible from your client, you must also open following ports: TCP 111, UDP 111, UDP 2049



## STEP 2 — CONFIGURE THE DATABASE SERVER

Install MySQL server
### sudo apt update
### sudo apt install mysql-server -y

Create a database and name it tooling
### sudo mysql
### create database tooling

Create a database user and name it webaccess
Grant permission to webaccess user on tooling database to do anything only from the webservers subnet cidr
![Screen Shot 2023-08-13 at 12 34 57 AM](https://github.com/bodebisi/Darey.io-Projects/assets/132711315/3d5a96ce-0bb6-4a18-a064-c4e7d781e8dc)
![Screen Shot 2023-08-13 at 12 35 53 AM](https://github.com/bodebisi/Darey.io-Projects/assets/132711315/1208564f-dde4-48e6-a142-1627c2c04192)
### create user 'webaccess'@'172.31.80.0/20' identified by 'password';
### grant all privileges on tooling.* to 'webaccess'@'172.31.80.0/20';
### flush privileges;
### show databases;

## Step 3 — Prepare the Web Servers
We need to make sure that our Web Servers can serve the same content from shared storage solutions, in our case – NFS Server and MySQL database.
You already know that one DB can be accessed for reads and writes by multiple clients. For storing shared files that our Web Servers will use – we will utilize NFS and mount previously created Logical Volume lv-apps to the folder where Apache stores files to be served to the users (/var/www).

This approach will make our Web Servers stateless, which means we will be able to add new ones or remove them whenever we need, and the integrity of the data (in the database and on NFS) will be preserved.

During the next steps we will do following:

Configure NFS client (this step must be done on all three servers)
Deploy a Tooling application to our Web Servers into a shared NFS folder
Configure the Web Servers to work with a single MySQL database
Launch a new EC2 instance with RHEL 8 Operating System
Install NFS client
sudo yum install nfs-utils nfs4-acl-tools -y
Mount /var/www/ and target the NFS server’s export for apps
sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www
Verify that NFS was mounted successfully by running df -h. Make sure that the changes will persist on Web Server after reboot:
sudo vi /etc/fstab
add following line

<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0
Install Remi’s repository, Apache and PHP
sudo yum install httpd -y

sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

sudo dnf module reset php

sudo dnf module enable php:remi-7.4

sudo dnf install php php-opcache php-gd php-curl php-mysqlnd

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

setsebool -P httpd_execmem 1
Repeat steps 1-5 for another 2 Web Servers.

Verify that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /mnt/apps. If you see the same files – it means NFS is mounted correctly. You can try to create a new file touch test.txt from one server and check if the same file is accessible from other Web Servers.
Locate the log folder for Apache on the Web Server and mount it to NFS server’s export for logs. Repeat step №4 to make sure the mount point will persist after reboot.
Fork the tooling source code from Darey.io Github Account to your Github account. (Learn how to fork a repo here)
Deploy the tooling website’s code to the Webserver. Ensure that the html folder from the repository is deployed to /var/www/html
Note 1: Do not forget to open TCP port 80 on the Web Server.

Note 2: If you encounter 403 Error – check permissions to your /var/www/html folder and also disable SELinux sudo setenforce 0
To make this change permanent – open following config file sudo vi /etc/sysconfig/selinux and set SELINUX=disabledthen restrt httpd.

        

Update the website’s configuration to connect to the database (in /var/www/html/functions.php file). Apply tooling-db.sql script to your database using this command mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql
Create in MySQL a new admin user with username: myuser and password: password:
INSERT INTO ‘users’ (‘id’, ‘username’, ‘password’, ’email’, ‘user_type’, ‘status’) VALUES
-> (1, ‘myuser’, ‘5f4dcc3b5aa765d61d8327deb882cf99’, ‘user@mail.com’, ‘admin’, ‘1’);

Open the website in your browser http://<Web-Server-Public-IP-Address-or-Public-DNS-Name>/index.php and make sure you can login into the website with myuser user.
        

Congratulations!
You have just implemented a web solution for a DevOps team using LAMP stack with remote Database and NFS servers.
