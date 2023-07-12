# IMPLEMENT A CLIENT SERVER ARCHITECTURE USING MYSQL DATABASE MANAGEMENT SYSTEM (DBMS).

### Client-Server refers to an architecture in which two or more computers are connected together over a network to send and receive requests between one another.
### In their communication, each machine has its own role: the machine sending requests is usually referred as “Client” and the machine responding (serving) is called “Server”

## To demonstrate a basic client-server using MySQL Relational Database Management System (RDBMS), follow the below instructions;

### 1. Create and configure two Linux-based virtual servers (EC2 instances in AWS). <img width="1280" alt="Screen Shot 2023-07-12 at 9 26 00 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/cbd49b68-a4e6-4a78-a9ba-532da322736f"> 

### In my previous projects, i describe how to launch and connect to an EC2 instance but if you haven't seen any of my projects, click this link to see how to launch it. https://youtu.be/xxKuB9kJoYM 
### I'm also dropping a link on how to connect your EC2 instance to your terminal or Virtual Machine. https://youtu.be/TxT6PNJts-s


### I'd also like to change the names of both servers on my terminal so i can differentiate them. so i type 
#### sudo hostname mysql-server
### I hit enter 
#### type bash
### hit enter again and the server name changes, I repeart the same process for the client server to also apply the change. see pic <img width="573" alt="Screen Shot 2023-07-12 at 9 35 47 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/4a154e6f-d0dc-4c9b-becc-3277a757791e">

## On mysql server Linux Server install MySQL Server software.
#### sudo apt update 
### Then run
#### sudo apt install mysql-server -y 
### Then start the mysql service with
#### sudo systemctl enable mysql

## On mysql client Linux Server install MySQL Client software.
#### sudo apt update 
### Then run
#### sudo apt install mysql-client 

### By default, both of your EC2 virtual servers are located in the same local virtual network, so they can communicate to each other using local IP addresses. Use mysql server's local IP address to connect from mysql client. MySQL server uses TCP port 3306 by default, so you will have to open it by creating a new entry in ‘Inbound rules’ in ‘mysql server’ Security Groups. For extra security, do not allow all IP addresses to reach your ‘mysql server’ – allow access only to the specific local IP address of your ‘mysql client’.

### First we get the Public IP address of the Mysql-client by typing 
#### ip address on the mysql-client server to reveal the public ip see picture <img width="571" alt="Screen Shot 2023-07-12 at 9 51 26 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/1d1adbf0-9400-4a48-9abd-067dc90194e1"> 

### Then we go to the EC2 instance of the mysql-server to edit our inbound rules in  security group to include Mysql TCP port 3306. see picture 

### Notice the public ip (Highlighted) of the mysql-client <img width="1280" alt="Screen Shot 2023-07-12 at 9 59 09 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/b83a74e0-592b-4bff-9de4-5b0ab3e7d228"> 
### Now we need to configure database and user on the mysql server, so use basic mysql commands to initiate.
#### sudo mysql_secure_installation

### Now You might need to configure MySQL server to allow connections from remote hosts.
#### sudo mysql
#### CREATE USER 'remote_user'@'%' IDENTIFIED WITH mysql_native_password BY <...>
#### mysql> CREATE DATABASE <Database_name>;
#### mysql> GRANT ALL ON <Database_name>.* TO 'remote_user'@'%' WITH GRANT OPTION;
#### mysql> FLUSH PRIVILEGES;
#### mysql> exit

### Now type this
#### sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
#### Replace ‘127.0.0.1’ to ‘0.0.0.0’ like this: <img width="569" alt="Screen Shot 2023-07-13 at 12 06 10 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/37463075-162e-460c-830f-57e2823a76d4"> 
### Then restart mysql with 
#### sudo systemctl restart mysql

### From mysql client Linux Server connect remotely to mysql server Database Engine without using SSH. You must use the mysql utility to perform this action. 
### Now go to your second server which is the mysql-client and type
#### sudo mysql -u remote_user -h 172.31.28.104 -p 
### NOTE: 172.31.28.104 is the private IP of the mysql-server.
### Check that you have successfully connected to a remote MySQL server and can perform SQL queries: 
#### Show databases;

### If you see an output similar to this image, <img width="620" alt="Screen Shot 2023-07-13 at 12 22 00 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/21c1f491-e35b-4dd1-83ea-cd2a4a7edb97">  
### then you have successfully completed this project – you have deloyed a fully functional MySQL Client-Server set up.

## CONGRATULATIONS


