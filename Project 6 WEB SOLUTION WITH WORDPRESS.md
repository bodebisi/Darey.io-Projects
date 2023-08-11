In this project you will be tasked to prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress. WordPress is a free and open-source content management system written in PHP and paired with MySQL or MariaDB as its backend Relational Database Management System (RDBMS).

## Note that this Project will consist of two parts:

1. Configure storage subsystem for Web and Database servers based on Linux OS. The focus of this part is to give you practical experience of working with disks, partitions and volumes in Linux.
2. Install WordPress and connect it to a remote MySQL database server. This part of the project will solidify your skills of deploying Web and DB tiers of Web solution.

# PART 1. Configure storage subsystem 
## Step 1 — Prepare a Web Server
Launch an EC2 instance that will serve as “Web Server”. 
<img width="1278" alt="Screen Shot 2023-07-24 at 10 49 00 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/3f41a01d-d83f-4ad6-b68f-ff79ccea3922">

Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB. Attached is a link to help you with this step https://youtu.be/HPXnXkBzIHw 

Attach all three volumes one by one to your Web Server EC2 instance 
<img width="1056" alt="Screen Shot 2023-08-08 at 1 00 29 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/f2efb9f8-72b7-4c64-bee2-2e51c08cc397">

Open up the Linux terminal to begin configuration
#### Use lsblk command to inspect what block devices are attached to the server. 
Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdg, xvdh. 
<img width="650" alt="Screen Shot 2023-07-24 at 11 44 11 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/2a253516-af9f-43a8-9dab-30a94400ef71">

#### Use df -h command to see all mounts and free space on your server

Use gdisk utility to create a single partition on each of the 3 disks
### sudo gdisk /dev/xvdf  

#### Use lsblk utility to view the newly configured partition on each of the 3 disks 
<img width="644" alt="Screen Shot 2023-07-25 at 12 02 39 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/1a9e02aa-b24b-476a-a5e9-4f0ad8b59354"> 

Install lvm2 package using 
### sudo yum install lvm2. 
### Run sudo lvmdiskscan command to check for available partitions 
<img width="669" alt="Screen Shot 2023-07-25 at 12 13 16 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/9e123d34-f673-488d-b2f6-0a85aa0a3ec3">

Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM
### sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1 

Verify that your Physical volume has been created successfully by running 
### sudo pvs 
<img width="712" alt="Screen Shot 2023-07-25 at 12 19 08 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/fc4f68a9-6e13-4b8c-b45f-c1be3538b46e">

Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg
### sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1

Verify that your VG has been created successfully by running 
### sudo vgs 
![Screen Shot 2023-07-25 at 12 30 16 AM](https://github.com/bodebisi/Darey.io-Projects/assets/132711315/ef23ea8c-1d0b-442b-8ddb-5f94bdb16bc7)

Use lvcreate utility to create 2 logical volumes, apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. 
## NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

### sudo lvcreate -n apps-lv -L 14G webdata-vg
### sudo lvcreate -n logs-lv -L 14G webdata-vg

Verify that your Logical Volume has been created successfully by running 
### sudo lvs 
<img width="707" alt="Screen Shot 2023-07-25 at 12 40 52 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/881d362d-7152-49d5-8b74-f90796105d17">

Verify the entire setup
### sudo vgdisplay -v #view complete setup - VG, PV, and LV
### sudo lsblk 
![Screen Shot 2023-07-25 at 12 47 07 AM](https://github.com/bodebisi/Darey.io-Projects/assets/132711315/35fe9fa1-fb38-4024-aa0c-e491c344bf9b)

Use mkfs.ext4 to format the logical volumes with ext4 filesystem
### sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
### sudo mkfs -t ext4 /dev/webdata-vg/logs-lv

Create /var/www/html directory to store website files
### sudo mkdir -p /var/www/html
## Note: It is good practice that you ls -l the first directory (i.e ls -l /var), to confirm the directory is not existing

Create /home/recovery/logs to store backup of log data
### sudo mkdir -p /home/recovery/logs

Mount /var/www/html on apps-lv logical volume
### sudo mount /dev/webdata-vg/apps-lv /var/www/html/
## Note: It is also good practice that you ls -l to confirm there is nothing in the directory you are about to mount (i.e ls -l /var/www/html/)

#### run df -h to confirm the mount
<img width="726" alt="Screen Shot 2023-07-25 at 1 04 34 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/fd30d27b-c218-4fc0-8c31-138f581fa6cf">

Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)
### sudo rsync -av /var/log/. /home/recovery/logs/

Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why the action above is very important)
### sudo mount /dev/webdata-vg/logs-lv /var/log

Restore log files back into /var/log directory
### sudo rsync -av /home/recovery/logs/. /var/log
####  df -h to confirm.

Update /etc/fstab file so that the mount configuration will persist after restart of the server.
## The UUID of the device will be used to update the /etc/fstab file;

### sudo blkid
<img width="1280" alt="Screen Shot 2023-07-25 at 1 42 52 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/f83ac734-114d-4a24-8efd-eb0c09489407">

### sudo vi /etc/fstab
Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.
<img width="719" alt="Screen Shot 2023-07-25 at 1 55 40 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/c0ffe817-9b71-4db0-a057-8ab849663842">

Test the configuration and reload the daemon
### sudo mount -a
### sudo systemctl daemon-reload

Verify your setup by running 
### df -h

Your Output should look like this

<img width="721" alt="Screen Shot 2023-07-25 at 2 18 28 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/5e34fe66-5795-445a-8095-14f9ed828adf">

# Step 2 — Prepare the Database Server
Launch a second RedHat EC2 instance that will have a role – ‘DB Server’

Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.

### sudo vgcreate vg-database /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
### sudo lvcreate -n db-lv -L 20G vg-database

<img width="714" alt="Screen Shot 2023-07-25 at 2 45 56 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/b9cd2449-5eff-4f87-b11a-242e80d444f2">

### sudo mkdir /db
### sudo mkfs.ext4 /dev/vg-database/db-lv

<img width="716" alt="Screen Shot 2023-07-25 at 2 49 58 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/f99dd704-b627-47aa-84c5-de1c5d08fc5c">

### sudo ls -l /db (To confirm nothing is in the directory)
### sudo mount /dev/vg-database/db-lv /db
### df -h 
### blkid 

<img width="1096" alt="Screen Shot 2023-07-25 at 2 56 29 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/0d0e7691-af77-4a5f-b4f5-f071fbf227d5">

### sudo vi /etc/fstab

<img width="609" alt="Screen Shot 2023-08-11 at 1 44 46 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/b6112865-a81f-4ab4-8d0c-215e8e9de5f9">

### sudo mount -a
### sudo systemctl daemon-reload
 
<img width="698" alt="Screen Shot 2023-07-25 at 3 04 32 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/7b0cb55e-6bbf-4ce3-b94d-54fcfccf3170">

#### Verify your setup by running df -h, output must look like this:

<img width="531" alt="Screen Shot 2023-08-11 at 1 48 56 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/fd82f5a0-faf1-41f7-a8c3-96a2a042592c">

## Install WordPress on your Web Server EC2

Update the repository
### sudo yum -y update

Install wget, Apache and it’s dependencies
### sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json

Install PHP and it’s dependencies
## Note: It is always advisable to google (how to install the latest version of php on red hat) so as to get the latest php installation as the installation below might have been updated.

### sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
### sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
### sudo dnf module list php
### sudo dnf module enable php:remi-8.2
### sudo dnf install php php-cli php-common 
### sudo yum install php php-opcache php-gd php-curl php-mysqlnd
### php -v 
### sudo systemctl start php-fpm
### sudo systemctl enable php-fpm
### sudo setsebool -P httpd_execmem 1
### sudo systemctl status php-fpm

<img width="525" alt="Screen Shot 2023-08-11 at 2 26 14 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/1ed7bf60-7eb3-4325-a655-1f8bd5a40dd3">

Start Apache
### sudo systemctl start httpd
### sudo systemctl enable httpd
### sudo systemctl status httpd

<img width="535" alt="Screen Shot 2023-08-11 at 2 32 28 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/113b04b1-3d2f-4a9b-aa58-9ffcce7362ae">

Download wordpress and copy wordpress to var/www/html
#### sudo ls -l /var/www/html
#### mkdir wordpress
#### cd wordpress/
#### sudo wget http://wordpress.org/latest.tar.gz
#### ls -l
#### sudo tar xzvf latest.tar.gz
#### ls -l
#### cd wordpress/
#### ls -l
#### sudo cp -R wp-config-sample.php wp-config.php 
#### ls -l
#### cat wp-config.php
#### pwd
#### cd ..
#### ls
#### sudo cp -R wordpress/ /var/www/html
#### cd /var/www/html
#### ls -l
#### sudo rm -rf wordpress/
#### sudo rm -rf lost+found/
#### ls
#### ls -l
#### cd ../..
#### cd
#### ls
#### cd wordpress/
#### ls
#### ls -l wordpress
#### sudo cp -R wordpress/. /var/www/html/
#### sudo ls -l /var/www/html
#### cd /var/www/html
#### ls

<img width="778" alt="Screen Shot 2023-08-11 at 3 04 40 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/59dc77e0-df6e-45c7-8f05-2d258e2062bb">

Then Install mysql in the web-server
### sudo yum install mysql-server
### sudo systemctl start mysqld
### sudo systemctl enable mysqld
### sudo systemctl status mysqld

Step 2 — Install MySQL on your DB Server EC2
### sudo yum update
### sudo yum install mysql-server -y
### sudo systemctl start mysqld
### sudo systemctl enable mysqld
### sudo systemctl status mysqld

Configure DB to work with WordPress
### sudo mysql_secure_installation
### sudo mysql
if there's an error 1045, then type 
### sudo mysql -u root -p
input password


Configure DB to work with WordPress
#### sudo mysql
 CREATE DATABASE wordpress;
 SHOW DATABASES;
 CREATE USER `admin`@`%` IDENTIFIED BY ‘password’;
 GRANT ALL ON wordpress.* TO 'admin'@'%';
 FLUSH PRIVILEGES;
 select user, host from mysql.user; (To check if the user is created)
 exit

Set your bind-address to 0.0.0.0 on your DB-server
### sudo vi /etc/my.cnf

Do the following edits In your Web-server. Edit DB Name, DB User, DB Password and DB Host. (remeber to use the DB server private ip for your host)
### sudo vi wp-config.php

<img width="623" alt="Screen Shot 2023-08-11 at 3 42 21 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/297c8304-b97e-4e69-babd-f8c0571b3198">

Change permissions and configuration so default Apache is disabled to use WordPress
### sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_backup

Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server’s IP address, so in the Inbound Rule configuration specify source as /32

<img width="1277" alt="Screen Shot 2023-08-10 at 1 51 11 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/75a8f0ee-3923-42a7-b0a0-bde5e75f707b">

Test that you can connect from your Web Server to your DB server by using mysql-client
### sudo mysql - h 172.31.91.121-u admin-p

Verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases.

<img width="623" alt="Screen Shot 2023-08-11 at 4 03 57 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/df2aae98-4b4f-4ba9-8154-cb9c6c96edec">

Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)

<img width="1020" alt="Screen Shot 2023-08-11 at 4 06 44 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/f5059bc6-7e75-4791-b984-fbbb8e38fdeb">

Configure SELinux Policies
### sudo chown -R apache:apache /var/www/html/
### sudo chcon -t httpd_sys_rw_content_t /var/www/html/ -R
### sudo setsebool -P httpd_can_network_connect=1
### sudo setsebool -P httpd_can_network_connect_db 1

Try to access from your browser the link to your WordPress http://Web-Server-Public-IP-Address

Fill out your DB credentials:

<img width="1276" alt="Screen Shot 2023-08-11 at 4 20 44 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/f50b76e2-3047-4106-80a8-3138ea537daf">

### If you see this message – it means your WordPress has successfully connected to your remote MySQL database and you are signed in:

<img width="1280" alt="Screen Shot 2023-08-11 at 4 23 16 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/f58a1df6-34f8-4c53-aacf-8bf7eefe2bd5">

## Important: Do not forget to STOP your EC2 instances after completion of the project to avoid extra costs.

# CONGRATULATIONS!
You have learned how to configure Linux storage subsystem and have also deployed a full-scale Web Solution using WordPress CMS and MySQL RDBMS!
