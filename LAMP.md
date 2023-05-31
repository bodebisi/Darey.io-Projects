# WEB STACK IMPLEMENTATION (LAMP STACK) IN AWS

## LAMP stands for Linux, Apache, MySQL, PHP
## LAMP is one of the webstack Technology which is a set of frameworks and tools which are very specifically chosen to work together in creating a well-functioning software.
## If you google LAMP stack you will find that it is a bundle of four different software technologies (which i listed above in line 3) that developers use to build websites and web applications. LAMP is an acronym for the operating system, Linux; the web server, Apache; the database server, MySQL; and the programming language, PHP.
### Lets get started.
### First we we create and log into out AWS cloud server and Launch an EC2 Instance, you can click the link watch a video on how to sign up to AWS, launch an EC2 instance. https://youtu.be/xxKuB9kJoYM
### Then you connect by stating up your Virtual machine and connecting to your EC2 instaance service. click link to see how to connect https://youtu.be/TxT6PNJts-s
### It is important to note that you have to be in the directory you downloaded your key-pair which was created when you launched the EC2 instance.
### So change the directory, most times the key-pair will be in your download folder. 
### so cd ~/Downloads
### Also note that anywhere you see these anchor tags < > , going forward, it means you will need to replace the content in there with values specific to your situation. For example, if we need you to replace the name you have saved the private key on your machine
### sudo chmod 0400 <private-key-name>.pem to confirm your private key is in the right directory
### then ssh -i <private-key-name>.pem ubuntu@<Public-IP-address> to connect to your terminal. see an example <img width="1280" alt="Screen Shot 2023-05-31 at 7 02 28 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/6b5324fa-9d50-4d8b-9035-1f864a2bad3f">
### Now as you are well aware, LAMP server means Linux, Apache, MySQL, PHP. and we already have our Linux running on Ubuntu when we created our instance and connected it to our VM or Terminal(in my case), so the next step is to install Apache.
  What is Apache: Apache is an open source software available for free. It runs on 67% of all webservers in the world. It is fast, reliable, and secure. It can be highly customized to meet the needs of many different environments by using extensions and modules.
  Install Apache using Ubuntu’s package manager ‘apt’
  First, i will update a list of packages in package manager 
  sudo apt update
  run apache2 package installation
  sudo apt install apache2
  To verify that apache2 is running as a Service in our OS, use following command
  sudo systemctl status apache2
  you should see this image to let you know Apache is running <img width="534" alt="Screen Shot 2023-05-31 at 7 16 19 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/5721bde9-82db-47ba-a202-9ca325b301e0">
  If it is green and running, then you did everything correctly – you have just launched your first Web Server in the Clouds! Before we can receive any traffic by our Web Server, we need to open TCP port 80 which is the default port that web browsers use to access web pages on the Internet. As we know, we have TCP port 22 open by default on our EC2 machine to access it via SSH, so we need to add a rule to EC2 configuration to open inbound connection through port 80:
  After opening your HTTP port 80 by editing your inbound rules in your security and setting your sourxe type to Anywhere-IPv4, Now it is time for us to test how our Apache HTTP server can respond to requests from the Internet. Open a web browser of your choice and try to access following url http://<Public-IP-Address>:80 (your public server is your Instance Public IPv4 as highlighted in the image <img width="1003" alt="Screen Shot 2023-05-31 at 7 31 52 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/7b61d47c-3932-4e2f-849f-3d68a2593765">
  Another way to retrieve your Public IP address, other than to check it in AWS Web console, is to use following command curl -s http://169.254.169.254/latest/meta-data/public-ipv4
  The URL in browser shall also work if you do not specify port number since all web browsers use port 80 by default. If you see following page, then your web server is now correctly installed and accessible through your firewall 
  If you see following page, then your web server is now correctly installed and accessible through your firewall <img width="1280" alt="Screen Shot 2023-05-31 at 7 35 56 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/36154e51-e1c5-4ed6-853e-0bb8cb04ca7b">
  
 
