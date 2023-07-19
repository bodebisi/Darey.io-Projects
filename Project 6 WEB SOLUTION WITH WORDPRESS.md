### In this project you will be tasked to prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress. WordPress is a free and open-source content management system written in PHP and paired with MySQL or MariaDB as its backend Relational Database Management System (RDBMS).

# Note that this Project will consist of two parts:

### 1. Configure storage subsystem for Web and Database servers based on Linux OS. The focus of this part is to give you practical experience of working with disks, partitions and volumes in Linux.
### 2. Install WordPress and connect it to a remote MySQL database server. This part of the project will solidify your skills of deploying Web and DB tiers of Web solution.

# PART 1. Configure storage subsystem 
## Step 1 — Prepare a Web Server
### Launch an EC2 instance that will serve as “Web Server”. Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB. Attached is a link to help you with this step https://youtu.be/HPXnXkBzIHw 

### Attach all three volumes one by one to your Web Server EC2 instance <add pic>

### Open up the Linux terminal to begin configuration
### Use lsblk command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg. <add pic>
### Use df -h command to see all mounts and free space on your server
### Use gdisk utility to create a single partition on each of the 3 disks
#### sudo gdisk /dev/xvdf   <add pic>

### Use lsblk utility to view the newly configured partition on each of the 3 disks. <add pic>

### Install lvm2 package using 
#### sudo yum install lvm2. 
#### Run sudo lvmdiskscan command to check for available partitions <add pic>

### Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM <Add pic)

### Verify that your Physical volume has been created successfully by running 
#### sudo pvs <Add pic>

### Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg
#### sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1

### Verify that your VG has been created successfully by running 
#### sudo vgs <Add pic>

### Create 2 logical volumes, apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. 
## NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.
#### Use lvcreate utility  <Add pic>

### Verify that your Logical Volume has been created successfully by running sudo lvs <add pic>

### Verify the entire setup
#### sudo vgdisplay -v #view complete setup - VG, PV, and LV
#### sudo lsblk <add pic>

### Use mkfs.ext4 to format the logical volumes with ext4 filesystem
#### sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
#### sudo mkfs -t ext4 /dev/webdata-vg/logs-lv

### Create /var/www/html directory to store website files
#### sudo mkdir -p /var/www/html
### Create /home/recovery/logs to store backup of log data
#### sudo mkdir -p /home/recovery/logs
### Mount /var/www/html on apps-lv logical volume
#### sudo mount /dev/webdata-vg/apps-lv /var/www/html/

### Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)
#### sudo rsync -av /var/log/. /home/recovery/logs/

### Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very important)
#### sudo mount /dev/webdata-vg/logs-lv /var/log
### Restore log files back into /var/log directory
#### sudo rsync -av /home/recovery/logs/log/. /var/log

### Update /etc/fstab file so that the mount configuration will persist after restart of the server.
## The UUID of the device will be used to update the /etc/fstab file;
#### sudo blkid <Add pic>

#### sudo vi /etc/fstab
### Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.

### Test the configuration and reload the daemon
#### sudo mount -a
#### sudo systemctl daemon-reload

### Verify your setup by running df -h, output must look like this: <add pic> 

## Step 2 — Prepare the Database Server
### Launch a second RedHat EC2 instance that will have a role – ‘DB Server’
### Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.

# Part 2  — Install WordPress on your Web Server EC2
## Step 1
### Update the repository
#### sudo yum -y update

### Install wget, Apache and it’s dependencies
#### sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json

### Start Apache
#### sudo systemctl enable httpd
#### sudo systemctl start httpd

### Install PHP and it’s depemdencies
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1

### Restart Apache
#### sudo systemctl restart httpd

### Download wordpress and copy wordpress to var/www/html
  mkdir wordpress
  cd   wordpress
  sudo wget http://wordpress.org/latest.tar.gz
  sudo tar xzvf latest.tar.gz
  sudo rm -rf latest.tar.gz
  cp wordpress/wp-config-sample.php wordpress/wp-config.php
  cp -R wordpress /var/www/html/
  
### Configure SELinux Policies
#### sudo chown -R apache:apache /var/www/html/wordpress
#### sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
#### sudo setsebool -P httpd_can_network_connect=1

## Step 2 — Install MySQL on your DB Server EC2
#### sudo yum update
#### sudo yum install mysql-server

### Verify that the service is up and running by using 
#### sudo systemctl status mysql, 

### if it is not running, restart the service and enable it so it will be running even after reboot:
#### sudo systemctl restart mysql
#### sudo systemctl enable mysql

## Step 3 — Configure DB to work with WordPress
#### sudo mysql
#### CREATE DATABASE wordpress;
#### CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
#### RANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
#### FLUSH PRIVILEGES;
#### SHOW DATABASES;
#### exit

## Step 4 — Configure WordPress to connect to remote database.
## Hint: Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server’s IP address, so in the Inbound Rule configuration specify source as /32 <Add pic)

### Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client
#### sudo yum install mysql
#### sudo mysql -u admin -p -h <DB-Server-Private-IP-address>

### Verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases.
### Change permissions and configuration so Apache could use WordPress:
#### Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)
#### Try to access from your browser the link to your WordPress http://<Web-Server-Public-IP-Address>/wordpress/ <Add pic>

### Fill out your DB credentials: <Add pic>

### If you see this message – it means your WordPress has successfully connected to your remote MySQL database <Add Pic>

## Important: Do not forget to STOP your EC2 instances after completion of the project to avoid extra costs.

# CONGRATULATIONS!
### You have learned how to configure Linux storage subsystem and have also deployed a full-scale Web Solution using WordPress CMS and MySQL RDBMS!
