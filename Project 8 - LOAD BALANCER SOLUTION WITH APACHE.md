# LOAD BALANCER SOLUTION WITH APACHE
In this project we will enhance our Tooling Website solution by adding a Load Balancer to distribute traffic between Web Servers and allow users to access our website using a single URL.

In our set up in Project-7 we had 3 Web Servers and each of them had its own public IP address and public DNS name. A client has to access them by using different URLs, which is not a nice user experience to remember addresses/names of even 3 server, let alone millions of Google servers.

In order to hide all this complexity and to have a single point of access with a single public IP address/name, a Load Balancer can be used. 

A Load Balancer (LB) distributes clients’ requests among underlying Web Servers and makes sure that the load is distributed in an optimal way.

## Side Self Study:
Read about different Load Balancing concepts (https://www.nginx.com/resources/glossary/load-balancing/) and difference between L4 Network LB (https://www.nginx.com/resources/glossary/layer-4-load-balancing/)  and L7 Application LB (https://www.nginx.com/resources/glossary/layer-7-load-balancing/).

Read more about different configuration aspects of Apache mod_proxy_balancer module.(https://httpd.apache.org/docs/2.4/mod/mod_proxy_balancer.html) and understand what sticky session means and when it is used.

## Task
Deploy and configure an Apache Load Balancer for Tooling Website solution on a separate Ubuntu EC2 intance. Make sure that users can be served by Web servers through the Load Balancer.

To simplify, let us implement this solution with 2 Web Servers, the approach will be the same for 3 and more Web Servers.

## Prerequisites
Make sure that you have following servers installed and configured within Project-7:

Two RHEL8 Web Servers
One MySQL DB Server (based on Ubuntu 20.04)
One RHEL8 NFS server

## Configure Apache As A Load Balancer
Create an Ubuntu Server 20.04 EC2 instance and name it Project-8-apache-lb, so your EC2 list will look like this (pic)
        
Open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in Security Group.(pic)

#Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers:
### sudo apt update
### sudo apt install apache2 -y
### sudo apt-get install libxml2-dev

#Enable following modules:
### sudo a2enmod rewrite
### sudo a2enmod proxy
### sudo a2enmod proxy_balancer
### sudo a2enmod proxy_http
### sudo a2enmod headers
### sudo a2enmod lbmethod_bytraffic

#Restart apache2 service
### sudo systemctl restart apache2

<img width="1280" alt="Screen Shot 2023-08-15 at 3 36 42 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/af7c9a66-0938-4b71-b5ae-a4c66ef6ddf2">

Make sure apache2 is up and running 
### sudo systemctl status apache2

#Configure load balancing
### sudo vi /etc/apache2/sites-available/000-default.conf

#Add this configuration into this section <VirtualHost *:80>  </VirtualHost> 
<Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/

<img width="648" alt="Screen Shot 2023-08-15 at 3 33 07 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/ea4706f7-15e2-4910-b6f8-2cc40e816f2d">

#Restart apache server
### sudo systemctl restart apache2

#bytraffic balancing method will distribute incoming load between your Web Servers according to current traffic load. We can control in which proportion the traffic must be distributed by loadfactor parameter.

## Note: You can also study and try other methods, like: 
bybusyness (https://httpd.apache.org/docs/2.4/mod/mod_lbmethod_bybusyness.html) , 
byrequests (https://httpd.apache.org/docs/2.4/mod/mod_lbmethod_byrequests.html) , 
heartbeat (https://httpd.apache.org/docs/2.4/mod/mod_lbmethod_heartbeat.html).

#Verify that our configuration works – try to access your LB’s public IP address or Public DNS name from your browser:(pic)
http://Load-Balancer-Public-IP-Address-or-Public-DNS-Name/index.php

<img width="1046" alt="Screen Shot 2023-08-15 at 3 44 11 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/ff7f9706-658b-4c65-b5c1-490613fd3c80">
<img width="1280" alt="Screen Shot 2023-08-15 at 3 43 48 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/a306caed-6cc3-47ba-a44d-de1ea11d81a2">
<img width="1280" alt="Screen Shot 2023-08-15 at 3 45 14 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/c3a5ed6c-694e-40a7-baab-ad0a91e3cd50">

## Note: If in the Project-7 you mounted /var/log/httpd/ from your Web Servers to the NFS server – unmount them and make sure that each Web Server has its own log directory.

#Open two ssh/Putty consoles for both Web Servers and run following command:
### sudo tail -f /var/log/httpd/access_log

<img width="1275" alt="Screen Shot 2023-08-15 at 3 54 50 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/d0948116-c39a-4454-b5d3-acfbfc992f76">

#Try to refresh your browser page http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php several times and make sure that both servers receive HTTP GET requests from your LB – new records must appear in each server’s log file. 
The number of requests to each server will be approximately the same since we set loadfactor to the same value for both servers – it means that traffic will be disctributed evenly between them.

If you have configured everything correctly – your users will not even notice that their requests are served by more than one server.


## Optional Step – Configure Local DNS Names Resolution
Sometimes it is tedious to remember and switch between IP addresses, especially if you have a lot of servers under your management.
What we can do, is to configure local domain name resolution. The easiest way is to use /etc/hosts file, although this approach is not very scalable, but it is very easy to configure and shows the concept well. So let us configure IP address to domain name mapping for our LB.

#Open this file on your LB server
### sudo vi /etc/hosts

#Add 2 records into this file with Local IP address and arbitrary name for both of your Web Servers 
<WebServer1-Private-IP-Address> Web1
<WebServer2-Private-IP-Address> Web2

![Screen Shot 2023-08-15 at 4 18 54 AM](https://github.com/bodebisi/Darey.io-Projects/assets/132711315/1eb8a739-023d-4ce9-bd2a-e089c3568866)

#Now you can update your LB config file with those names instead of IP addresses.
BalancerMember http://Web1:80 loadfactor=5 timeout=1
BalancerMember http://Web2:80 loadfactor=5 timeout=1

![Screen Shot 2023-08-15 at 4 22 14 AM](https://github.com/bodebisi/Darey.io-Projects/assets/132711315/b2df6610-219d-4950-87c5-c5b287bb9a78)

#You can try to curl your Web Servers from LB locally curl http://Web1 or curl http://Web2 – it shall work.

![Screen Shot 2023-08-15 at 4 26 03 AM](https://github.com/bodebisi/Darey.io-Projects/assets/132711315/57523d85-6e5e-4f04-b1fa-662cb9407f7e)
![Screen Shot 2023-08-15 at 4 27 08 AM](https://github.com/bodebisi/Darey.io-Projects/assets/132711315/a1d0b38b-6a22-41ba-8019-4af803bcdb75)

Remember, this is only internal configuration and it is also local to your LB server, these names will neither be ‘resolvable’ from other servers internally nor from the Internet.

Targed Architecture
Now your set up looks like this:
https://darey-io-nonprod-pbl-projects.s3.eu-west-2.amazonaws.com/project8/project8_final.png

# Congratulations!
You have just implemented a Load balancing Web Solution.

