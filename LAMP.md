# WEB STACK IMPLEMENTATION (LAMP STACK) IN AWS

## LAMP stands for Linux, Apache, MySQL, PHP
## LAMP is one of the webstack Technology which is a set of frameworks and tools which are very specifically chosen to work together in creating a well-functioning software.
## If you google LAMP stack you will find that it is a bundle of four different software technologies (which i listed above in line 3) that developers use to build websites and web applications. LAMP is an acronym for the operating system, Linux; the web server, Apache; the database server, MySQL; and the programming language, PHP.
### Lets get started.
## STEP 1 — INSTALLING APACHE AND UPDATING THE FIREWALL
### First we create and log into out AWS cloud server and Launch an EC2 Instance, you can click the link watch a video on how to sign up to AWS, launch an EC2 instance. https://youtu.be/xxKuB9kJoYM
### Then you connect by stating up your Virtual machine and connecting to your EC2 instaance service. click link to see how to connect https://youtu.be/TxT6PNJts-s
### It is important to note that you have to be in the directory you downloaded your key-pair which was created when you launched the EC2 instance.
### So change the directory, most times the key-pair will be in your download folder. 
### so cd ~/Downloads
### Also note that anywhere you see these anchor tags < > , going forward, it means you will need to replace the content in there with values specific to your situation. For example, if we need you to replace the name you have saved the private key on your machine
### sudo chmod 0400 <private-key-name>.pem to confirm your private key is in the right directory
### then ssh -i <private-key-name>.pem ubuntu@<Public-IP-address> to connect to your terminal. see an example <img width="1280" alt="Screen Shot 2023-05-31 at 7 02 28 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/6b5324fa-9d50-4d8b-9035-1f864a2bad3f">
### Now as you are well aware, LAMP server means Linux, Apache, MySQL, PHP. and we already have our Linux running on Ubuntu when we created our instance and connected it to our VM or Terminal(in my case), so the next step is to install Apache.
### What is Apache: Apache is an open source software available for free. It runs on 67% of all webservers in the world. It is fast, reliable, and secure. It can be highly customized to meet the needs of many different environments by using extensions and modules.
### Install Apache using Ubuntu’s package manager ‘apt’
### First, i will update a list of packages in package manager 
### sudo apt update
### run apache2 package installation
### sudo apt install apache2
### To verify that apache2 is running as a Service in our OS, use following command
### sudo systemctl status apache2
### you should see this image to let you know Apache is running <img width="534" alt="Screen Shot 2023-05-31 at 7 16 19 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/5721bde9-82db-47ba-a202-9ca325b301e0">
### If it is green and running, then you did everything correctly – you have just launched your first Web Server in the Clouds! Before we can receive any traffic by our Web Server, we need to open TCP port 80 which is the default port that web browsers use to access web pages on the Internet. As we know, we have TCP port 22 open by default on our EC2 machine to access it via SSH, so we need to add a rule to EC2 configuration to open inbound connection through port 80:
### After opening your HTTP port 80 by editing your inbound rules in your security and setting your sourxe type to Anywhere-IPv4, Now it is time for us to test how our Apache HTTP server can respond to requests from the Internet. Open a web browser of your choice and try to access following url http://<Public-IP-Address>:80 (your public server is your Instance Public IPv4 as highlighted in the image <img width="1003" alt="Screen Shot 2023-05-31 at 7 31 52 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/7b61d47c-3932-4e2f-849f-3d68a2593765">
### Another way to retrieve your Public IP address, other than to check it in AWS Web console, is to use following command curl -s http://169.254.169.254/latest/meta-data/public-ipv4
### The URL in browser shall also work if you do not specify port number since all web browsers use port 80 by default. If you see following page, then your web server is now correctly installed and accessible through your firewall 
### If you see following page, then your web server is now correctly installed and accessible through your firewall <img width="1280" alt="Screen Shot 2023-05-31 at 7 35 56 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/36154e51-e1c5-4ed6-853e-0bb8cb04ca7b">
## STEP 2 — INSTALLING MYSQL
    
### Now that you have a web server up and running, you need to install a Database Management System (DBMS) to be able to store and manage data for your site in a relational database. MySQL (Which represents the M acronym in our LAMP) is a popular relational database management system used within PHP environments. so we will install it.
### sudo apt install mysql-server
### sudo mysql
### You should see this <img width="520" alt="Screen Shot 2023-05-31 at 7 48 49 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/e766b345-32f9-4917-ba3a-3697a782183f">
### It’s recommended that you run a security script that comes pre-installed with MySQL. This script will remove some insecure default settings and lock down access to your database system. Before running the script you will set a password for the root user, using mysql_native_password as default authentication method.
### ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '*Input your password*'; 
### Then Exit the MySQL shell with: mysql> exit
### Now Start the interactive script by running: sudo mysql_secure_installation
### This will ask if you want to configure the VALIDATE PASSWORD PLUGIN. Answer Y for yes, or anything else to continue without enabling
### Note: VALIDATE PASSWORD PLUGIN can be used to test passwordsand improve security. It checks the strength of password and allows the users to set only those passwords which aresecure enough. Would you like to setup VALIDATE PASSWORD plugin? Press y|Y for Yes, any other key for No:
### If you answer “yes”, you’ll be asked to select a level of password validation. Keep in mind that if you enter 2 for the strongest level, you will receive errors when attempting to set any password which does not contain numbers, upper and lowercase letters, and special characters, or which is based on common dictionary words
### For the rest of the questions, press Y and hit the ENTER key at each prompt
### When you’re finished, test if you’re able to log in to the MySQL console by typing: sudo mysql -p
### The -p will prompt you for your password. If it works, you can exit again by typing mysql> exit
### Your MySQL server is now installed and secured. Next, we will install PHP, the final component in the LAMP stack
    
## STEP 3 — INSTALLING PHP
    
### What is PHP?  PHP is the component of our setup that will process code to display dynamic content to the end user
### In addition to the php package, you’ll need php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases. You’ll also need libapache2-mod-php to enable Apache to handle PHP files. Core PHP packages will automatically be installed as dependencies. To install these 3 packages at once, run:
### sudo apt install php libapache2-mod-php php-mysql
### Once the installation is finished, you can run the following command to confirm your PHP version: 
### php -v
### You should see something like this <img width="544" alt="Screen Shot 2023-05-31 at 8 36 54 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/90d36a9d-8466-4bef-919e-4b10b4b9c4b1">
### At this point, your LAMP stack is completely installed and fully operational.
### To test your setup with a PHP script, it’s best to set up a proper Apache Virtual Host to hold your website’s files and folders. Virtual host allows you to have multiple websites located on a single machine and users of the websites will not even notice it.

## STEP 4 — CREATING A VIRTUAL HOST FOR YOUR WEBSITE USING APACHE

### To do this we would start by setting up a domain called projectlamp, to create the directory for projectlamp using ‘mkdir’ command as follows:
### sudo mkdir /var/www/projectlamp
### Next, assign ownership of the directory with your current system user:
### sudo chown -R $USER:$USER /var/www/projectlamp
### Then, create and open a new configuration file in Apache’s sites-available directory using your preferred command-line editor. Here, we’ll be using vi or vim
### sudo vi /etc/apache2/sites-available/projectlamp.conf
### This will create a new blank file. Paste in the following bare-bones configuration by hitting on i on the keyboard to enter the insert mode, and paste the text
### <VirtualHost *:80>
    ServerName projectlamp
    ServerAlias www.projectlamp 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/projectlamp
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
### To save and close the file, simply follow the steps below:
Hit the esc button on the keyboard
Type wq. w for write and q for quit
Hit ENTER to save the file

### You can use the ls command to show the new file in the sites-available directory:

### sudo ls /etc/apache2/sites-available

### With this VirtualHost configuration, we’re telling Apache to serve projectlamp using /var/www/projectlampl as its web root directory. If you would like to test Apache without a domain name

### You can now use a2ensite command to enable the new virtual host:

### sudo a2ensite projectlamp

### You might want to disable the default website that comes installed with Apache. This is required if you’re not using a custom domain name, because in this case Apache’s default configuration would overwrite your virtual host.

### sudo a2dissite 000-default

### Make sure your configuration file doesn’t contain syntax errors, run:

### sudo apache2ctl configtest

### Finally, reload Apache so these changes take effect:

### sudo systemctl reload apache2

### Your new website is now active, but the web root /var/www/projectlamp is still empty. so create an index.html file in that location so that we can test that the virtual host works as expected:

### sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html

### Now go to your browser and try to open your website URL using IP address:

### http://<Public-IP-Address>:80 (note that anywhere you see these anchor tags < > , going forward, it means you will need to replace the content in there with values specific to your situation)

### you should see this <img width="994" alt="Screen Shot 2023-05-31 at 8 54 07 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/be708e82-5aeb-4bde-900a-645162f7a091">

### You can leave this file in place as a temporary landing page for your application until you set up an index.php file to replace it.
    
## STEP 5 — ENABLE PHP ON THE WEBSITE

### With the default DirectoryIndex settings on Apache, a file named index.html will always take precedence over an index.php.

### In case you want to change this behavior, you’ll need to edit the /etc/apache2/mods-enabled/dir.conf file and change the order in which the index.php file is listed within the DirectoryIndex directive:

### sudo vim /etc/apache2/mods-enabled/dir.conf

### <IfModule mod_dir.c>
        #Change this:
        #DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
        #To this:
        DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>

### save and exit with esc :wqa

### Then run: sudo systemctl reload apache2

### Finally, we will create a PHP script to test that PHP is correctly installed and configured on your server

### vim /var/www/projectlamp/index.php

### This will open a blank file. Add the following text, which is valid PHP code, inside the file:

### <?php
phpinfo();

### When you are finished, save and close the file, refresh the page and you will see a page similar to this: <img width="991" alt="Screen Shot 2023-05-31 at 10 00 27 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/f427783b-eff2-426b-809c-6fb08abb292c">

### This page provides information about your server from the perspective of PHP. It is useful for debugging and to ensure that your settings are being applied correctly.

### If you can see this page in your browser, then your PHP installation is working as expected.

### After checking the relevant information about your PHP server through that page, it’s best to remove the file you created as it contains sensitive information about your PHP environment -and your Ubuntu server. You can use rm to do so:

### sudo rm /var/www/projectlamp/index.php

### You can always recreate this page if you need to access the information again later.

### Hope you had as much fun as i did in this Project, see you in the next Project, Thank you.




  
  
  
  
  
  
  
  



  
  
 
