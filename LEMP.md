# WEB STACK IMPLEMENTATION (LEMP STACK)

## LEMP is an open-source web application stack used to develop web applications. The term LEMP is an acronym that represents L for the Linux Operating system, Nginx (pronounced as engine-x, hence the E in the acronym) web server, M for MySQL database, and P for PHP scripting language. LEMP enjoys good community support and is used around the world in many highly-scaled web applications. Nginx is the second most widely used web server in the world following Apache. From our previous project, we established that web stacks consisting of a bundle of software and frameworks or libraries which are used for building full-stack web apps.
### Now we have a brief understanding of LEMP stack, lets start a project implementation on it.
### First, we launch our EC2 instance and connect our Terminal (For MacOS user) or Virtual Machine.
## STEP 1 – INSTALLING THE NGINX WEB SERVER
### In order to display web pages to our site visitors, we are going to employ Nginx, a high-performance web server, we start by running: 
#### sudo apt update
#### sudo apt install nginx
### When prompted, enter Y to confirm that you want to install Nginx
### To verify that nginx was successfully installed and is running as a service in Ubuntu, run:
#### sudo systemctl status nginx
### It should be green and running like this: <img width="577" alt="Screen Shot 2023-05-31 at 11 54 30 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/13ce0c40-cd6a-4f7b-b990-7c4e91524823">
### we need to open TCP port 80 in tyhe security of our EC2 instance which is default port that web browsers use to access web pages in the Internet so we can receive any traffic by our Web Server.
### AFter opening our port 80: <img width="1220" alt="Screen Shot 2023-06-01 at 12 00 50 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/42bc9caf-0365-4a86-b0bd-09fa137332bb">
### Let us try to check how we can access it locally in our Ubuntu shell, run:
#### curl http://localhost:80
### Now it is time for us to test how our Nginx server can respond to requests from the Internet. Open a web browser of your choice and try to access following url
#### http://<Public-IP-Address>:80
## Note That Another way to retrieve your Public IP address, other than to check it in AWS Web console, is to use following command:
#### curl -s http://169.254.169.254/latest/meta-data/public-ipv4
### You should see this page <img width="1001" alt="Screen Shot 2023-06-01 at 12 07 05 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/b6aa6e39-9e84-449b-886f-fe884155399b">
### With the Nginx running, we can go to the next step which is to install a Database Management System (DBMS) to be able to store and manage data for your site in a relational database. MySQL is a popular relational database management system used within PHP environments, so we will use it in our project.
## STEP 2 — INSTALLING MYSQL
#### sudo apt install mysql-server
### When prompted, confirm installation by typing Y, and then ENTER.
#### sudo mysql
### This will connect to the MySQL server as the administrative database user root, which is inferred by the use of sudo when running this command
#### Then riun this script ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '<insert your password here>';
#### type exit
### Start the interactive script by running:
#### sudo mysql_secure_installation
### put in the password you inserted earlier
### This will ask if you want to configure the VALIDATE PASSWORD PLUGIN
### Answer Y for yes, or anything else to continue without enabling
### Regardless of whether you chose to set up the VALIDATE PASSWORD PLUGIN, your server will next ask you to select and confirm a password for the MySQL root user. This is not to be confused with the system root
### For the rest of the questions, press Y and hit the ENTER key at each prompt
### When you’re finished, test if you’re able to log in to the MySQL console by typing:
#### sudo mysql -p
### To exit the MySQL console, type: exit
## STEP 3 – INSTALLING PHP
### You have Nginx installed to serve your content and MySQL installed to store and manage your data. Now you can install PHP to process code and generate dynamic content for the web server:
#### sudo apt install php-fpm php-mysql
### When prompted, type Y and press ENTER to confirm installation
### You now have your PHP components installed. Next, you will configure Nginx to use them
## STEP 4 — CONFIGURING NGINX TO USE PHP PROCESSOR
### When using the Nginx web server, we can create server blocks (similar to virtual hosts in Apache) to encapsulate configuration details and host more than one domain on a single server
### Create the root web directory for your_domain as follows:
#### sudo mkdir /var/www/projectLEMP
### Next, assign ownership of the directory with the $USER environment variable, which will reference your current system user:
#### sudo chown -R $USER:$USER /var/www/projectLEMP
### Then, open a new configuration file in Nginx’s sites-available directory using your preferred command-line editor
#### sudo nano /etc/nginx/sites-available/projectLEMP
### This will create a new blank file. Paste in the following bare-bones configuration:
#### #/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}
### When you’re done editing, save and close the file. If you’re using nano, you can do so by typing CTRL+X and then y and ENTER to confirm
### Activate your configuration by linking to the config file from Nginx’s sites-enabled directory:
#### sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
### Then Test you configuration for syntax errors
#### sudo nginx -t
### If any errors are reported, go back to your configuration file to review its contents before continuing
### We also need to disable default Nginx host that is currently configured to listen on port 80, for this run:
#### sudo unlink /etc/nginx/sites-enabled/default
### reload Nginx to apply the changes:
#### sudo systemctl reload nginx
### Your new website is now active, but the web root /var/www/projectLEMP is still empty. Create an index.html file in that location so that we can test that your new server block works as expected:
#### sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
### Now go to your browser and try to open your website URL using IP address:
#### http://<Public-IP-Address>:80
### Yous should see this <img width="992" alt="Screen Shot 2023-06-01 at 12 39 04 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/d7a30cee-0e50-400b-8ddc-7075e5b07b3d">
### You can leave this file in place as a temporary landing page for your application until you set up an index.php file to replace it.
### Once you do that, remember to remove or rename the index.html file from your document root, as it would take precedence over an index.php file by default.
## Step 5 – Testing PHP with Nginx
### You can test it to validate that Nginx can correctly hand .php files off to your PHP processor
#### sudo nano /var/www/projectLEMP/info.php
### You can now access this page in your web browser by visiting the domain name or public IP address you’ve set up in your Nginx configuration file, followed by /info.php i.e
#### http://`server_domain_or_IP`/info.php
### You will see a web page containing detailed information about your server:
### it’s best to remove the file you created as it contains sensitive information about your PHP environment and your Ubuntu server. 
#### sudo rm /var/www/your_domain/info.php
## STEP 6 – RETRIEVING DATA FROM MYSQL DATABASE WITH PHP (CONTINUED)
### First, connect to the MySQL console using the root account:
#### 
