In this Project, we want to introduce a set of DevOps tools that will help our team in day to day activities in managing, developing, testing, deploying and monitoring different projects.

The tools we want our team to be able to use are well known and widely used by multiple DevOps teams, so we will introduce a single DevOps Tooling Solution that will consist of:

Jenkins – free and open source automation server used to build CI/CD pipelines.
Kubernetes – an open-source container-orchestration system for automating computer application deployment, scaling, and management.
Jfrog Artifactory – Universal Repository Manager supporting all major packaging formats, build tools and CI servers. Artifactory.
Rancher – an open source software platform that enables organizations to run and manage Docker and Kubernetes in production.
Grafana – a multi-platform open source analytics and interactive visualization web application.
Prometheus – An open-source monitoring system with a dimensional data model, flexible query language, efficient time series database and modern alerting approach.
Kibana – Kibana is a free and open user interface that lets you visualize your Elasticsearch data and navigate the Elastic Stack.

Side Self Study
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
Spin up a new EC2 instance with RHEL Linux 8 Operating System.

Based on your LVM experience from Project 6, Configure LVM on the Server.
Instead of formating the disks as ext4 you will have to format them as xfs
Ensure there are 3 Logical Volumes. lv-opt lv-apps, and lv-logs

Create mount points on /mnt directory for the logical volumes as follow:
### Mount lv-apps on /mnt/apps – To be used by webservers
### Mount lv-logs on /mnt/logs – To be used by webserver logs
### Mount lv-opt on /mnt/opt – To be used by Jenkins server in Project 8

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

    
