# TOOLING WEBSITE DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION. INTRODUCTION TO JENKINS
In this project we are going to start automating part of our routine tasks with a free and open source automation server – Jenkins. It is one of the most popular CI/CD tools, it was created by a former Sun Microsystems developer Kohsuke Kawaguchi and the project originally had a named “Hudson”.

According to Circle CI, Continuous integration (CI) is a software development strategy that increases the speed of development while ensuring the quality of the code that teams deploy. Developers continually commit code in small increments (at least daily, or even several times a day), which is then automatically built and tested before it is merged with the shared repository.

In our project we are going to utilize Jenkins CI capabilities to make sure that every change made to the source code in GitHub https://github.com/<yourname>/tooling will be automatically be updated to the Tooling Website.

## Side Self Study
Read about Continuous Integration, Continuous Delivery and Continuous Deployment. https://circleci.com/continuous-integration/

## Task
Enhance the architecture prepared in Project 8 by adding a Jenkins server, configure a job to automatically deploy source codes changes from Git to NFS server.

## Step 1 – Install Jenkins server

Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it “Jenkins”

#Install JDK (since Jenkins is a Java-based application)
### sudo apt update
### sudo apt install default-jdk-headless

#Install Jenkins

wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins

#Make sure Jenkins is up and running
### sudo systemctl status jenkins

![Screen Shot 2023-08-15 at 4 58 27 AM](https://github.com/bodebisi/Darey.io-Projects/assets/132711315/2376fabb-1d80-4ab6-944b-eec505411ac6)

By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group

<img width="1277" alt="Screen Shot 2023-08-15 at 5 01 49 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/f8bbe244-35c7-43c8-a1dc-a57c958dca2b">
        
#Perform initial Jenkins setup.
From your browser access http://Jenkins-Server-Public-IP-Address-or-Public-DNS-Name:8080

You will be prompted to provide a default admin password

<img width="1279" alt="Screen Shot 2023-08-15 at 5 03 15 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/fe130e04-b112-4e4b-be6c-8b28afa4bce7">
 
Retrieve it from your server:
### sudo cat /var/lib/jenkins/secrets/initialAdminPassword

<img width="1272" alt="Screen Shot 2023-08-15 at 5 06 47 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/1394373b-ba27-4940-ba0f-db827185fc62">

Then you will be asked which plugins to install – choose suggested plugins.

Once plugins installation is done – create an admin user and you will get your Jenkins server address.

<img width="1280" alt="Screen Shot 2023-08-15 at 5 13 09 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/294370ce-dbb7-4074-a0d1-55d943426c19">
The installation is completed!

![Screen Shot 2023-08-15 at 5 17 04 AM](https://github.com/bodebisi/Darey.io-Projects/assets/132711315/5ba1132b-1f81-4746-8769-9d7e6d891f63)

##Step 2 – Configure Jenkins to retrieve source codes from GitHub using Webhooks
In this part, you will learn how to configure a simple Jenkins job/project (these two terms can be used interchangeably). This job will will be triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.

#Enable webhooks in your GitHub repository settings 

<img width="1280" alt="Screen Shot 2023-08-15 at 11 55 50 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/3194783b-ab79-4d7b-bce9-44c413507f26">
        
Go to Jenkins web console, click “New Item” and create a “Freestyle project”

<img width="1273" alt="Screen Shot 2023-08-15 at 5 32 57 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/972d2424-e257-4c6d-9e30-05d9a1715d1f">

<img width="1276" alt="Screen Shot 2023-08-15 at 5 34 49 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/92598ca1-7a7b-4476-90d4-2433023e0abf">

<img width="1280" alt="Screen Shot 2023-08-15 at 12 11 28 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/58963d81-d996-4c52-9395-dc68877f32db">
        
#To connect your GitHub repository, you will need to provide its URL, you can copy from the repository itself 

<img width="1280" alt="Screen Shot 2023-08-15 at 12 13 18 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/48223df3-7830-45b5-8654-58fb5f967995">
 
In configuration of your Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository. 

<img width="1276" alt="Screen Shot 2023-08-15 at 5 47 28 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/59134ca0-bcf8-4381-b0ba-5b9ac1147b43">

![Screen Shot 2023-08-15 at 12 24 37 PM](https://github.com/bodebisi/Darey.io-Projects/assets/132711315/41457eae-9137-4d85-b78d-bdc2884930fe)

Save the configuration and let us try to run the build. For now we can only do it manually.

Click “Build Now” button, if you have configured everything correctly, the build will be successfull and you will see it under #1 

![Screen Shot 2023-08-15 at 12 27 00 PM](https://github.com/bodebisi/Darey.io-Projects/assets/132711315/64572b62-9618-49a9-8e7a-49f2cca8c80c)

You can open the build and check in “Console Output” if it has run successfully.

![Screen Shot 2023-08-15 at 12 29 34 PM](https://github.com/bodebisi/Darey.io-Projects/assets/132711315/fc597c93-7631-471e-8417-19979aff6f0f)

If so – congratulations! You have just made your very first Jenkins build!

But this build does not produce anything and it runs only when we trigger it manually. Let us fix it.

Click “Configure” your job/project and add these two configurations
Configure triggering the job from GitHub webhook: 

<img width="1275" alt="Screen Shot 2023-08-15 at 6 12 00 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/8de1334a-7df5-45d2-8875-3d6b08b8862f">

<img width="1269" alt="Screen Shot 2023-08-15 at 6 13 14 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/bb0a2749-40f0-4570-804d-c53cec82140b">

<img width="1280" alt="Screen Shot 2023-08-15 at 6 13 51 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/49df35cb-88ff-411e-bcce-17c4339095ca">

<img width="1279" alt="Screen Shot 2023-08-15 at 6 15 56 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/56aea225-d8bf-4641-8496-def98f0d87dc">

Configure “Post-build Actions” to archive all the files – files resulted from a build are called “artifacts”.

<img width="1280" alt="Screen Shot 2023-08-15 at 6 17 40 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/770c67ba-55e6-47ae-a6a2-5556c9ede0ca">
 
Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.
You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server. 

![Screen Shot 2023-08-15 at 12 38 39 PM](https://github.com/bodebisi/Darey.io-Projects/assets/132711315/900a4bc6-3f24-4849-a3b1-1fd99d818f57)

![Screen Shot 2023-08-15 at 12 40 07 PM](https://github.com/bodebisi/Darey.io-Projects/assets/132711315/309c8469-ad8f-4486-8fea-640561cad1ec)

![Screen Shot 2023-08-15 at 12 44 39 PM](https://github.com/bodebisi/Darey.io-Projects/assets/132711315/8ef3992a-3196-4df2-8fe1-273068467718)

![Screen Shot 2023-08-15 at 12 50 51 PM](https://github.com/bodebisi/Darey.io-Projects/assets/132711315/5f3bcd75-9248-4954-b412-3cd8e7445f20)

![Screen Shot 2023-08-15 at 12 47 18 PM](https://github.com/bodebisi/Darey.io-Projects/assets/132711315/93eae549-7e01-4f0e-96c7-097bcf5d9e63)

You have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as ‘push’ because the changes are being ‘pushed’ and files transfer is initiated by GitHub). There are also other methods: trigger one job (downstreadm) from another (upstream), poll GitHub periodically and others.

By default, the artifacts are stored on Jenkins server locally

### ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/

![Screen Shot 2023-08-15 at 1 07 29 PM](https://github.com/bodebisi/Darey.io-Projects/assets/132711315/497594f2-7722-4c2c-bd81-e2d23a1783e5)

## Step 3 – Configure Jenkins to copy files to NFS server via SSH
Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory.
Jenkins is a highly extendable application and there are 1400+ plugins available. We will need a plugin that is called “Publish Over SSH”.

#Install “Publish Over SSH” plugin.
On main dashboard select “Manage Jenkins” and choose “Manage Plugins” menu item.
On “Available” tab search for “Publish Over SSH” plugin and install it 

<img width="1280" alt="Screen Shot 2023-08-15 at 1 12 16 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/f72fecf9-4bf5-4d46-912d-628db5341781">

#Configure the job/project to copy artifacts over to NFS server.
On main dashboard select “Manage Jenkins” and choose “Configure System” menu item.

Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:

Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)
Arbitrary name
Hostname – can be private IP address of your NFS server
Username – ec2-user (since NFS server is based on EC2 with RHEL 8)
Remote directory – /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server
Test the configuration and make sure the connection returns Success.(pic)
Remember, that TCP port 22 on NFS server must be open to receive SSH connections.(pic)

    
Save the configuration, open your Jenkins job/project configuration page and add another one “Post-build Action”(pic)

 
Configure it to send all files produced by the build into our previously define remote directory. In our case we want to copy all files and directories – so we use **.
If you want to apply some particular pattern to define which files to send – use this syntax.(pic)

 
Save this configuration and go ahead, change something in README.MD file in your GitHub Tooling repository.

Webhook will trigger a new job and in the “Console Output” of the job you will find something like this: (pic)

SSH: Transferred 25 file(s)
Finished: SUCCESS

To make sure that the files in /mnt/apps have been updated – connect via SSH/Putty to your NFS server and check README.MD file
### cat /mnt/apps/README.md

If you see the changes you had previously made in your GitHub – the job works as expected.

# Congratulations!
You have just implemented your first Continuous Integration solution using Jenkins CI. Watch out for advanced CI configurations in upcoming projects.


