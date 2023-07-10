# MERN STACK DEPLOYMENT TO UBUNTU IN AWS

## MERN Web stack consists of following components:

### MongoDB: A document-based, No-SQL database used to store application data in a form of documents.
### ExpressJS: A server side Web Application framework for Node.js.
### ReactJS: A frontend framework developed by Facebook. It is based on JavaScript, used to build User Interface (UI) components.
### Node.js: A JavaScript runtime environment. It is used to run JavaScript on a machine rather than in a browser.
## How it works:
### A user interacts with the ReactJS UI components at the application front-end residing in the browser. This frontend is served by the application backend residing in a server, through ExpressJS running on top of NodeJS.
### Any interaction that causes a data change request is sent to the NodeJS based Express server, which grabs data from the MongoDB database if required, and returns the data to the frontend of the application, which is then presented to the user.
## Task
### To deploy a simple To-Do application that creates To-Do lists.
### First i will begin by starting up an Instance on our AWS console, then connect the instance to Virtual Machince(for those using Windows) or Terminal(For MasOS). 

## STEP 1 – BACKEND CONFIGURATION
### Update ubuntu
#### sudo apt update
### Then we upgrade Ubuntu
#### sudo apt upgrade
### Lets get the location of Node.js software from Ubuntu repositories.
#### curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
### Install Node.js with the command below
#### sudo apt-get install -y nodejs
### Note: The command above installs both nodejs and npm. NPM is a package manager for Node like apt for Ubuntu, it is used to install Node modules & packages and to manage dependency conflicts.
### Verify the node installation with the command below
#### node -v
### Verify the node installation with the command below
#### npm -v
### Now we set up Application Code by Creating a new directory for your To-Do project.
#### mkdir Todo
#### ls to confirm the Todo directory
#### cd Todo
### Next, you will use the command npm init to initialise your project, so that a new file named package.json will be created.
#### npm init
### Yous should see this <img width="1223" alt="Screen Shot 2023-06-01 at 8 19 49 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/6fe2309d-615e-4b6d-95f0-fd3e52c9aa85">
### Run the command ls to confirm that you have package.json file created.

## Stage 2: INSTALL EXPRESSJS
### Express helps to define routes of your application based on HTTP methods and URLs, To use express, install it using npm:
#### npm install express
### create a file index.js
#### touch index.js
### Install the dotenv module
#### npm install dotenv
#### vim index.js
### Type the code below into it and save.
##### const express = require('express');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use((req, res, next) => {
res.send('Welcome to Express');
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});

### Save and quit with esc :wqa
### Now it is time to start our server to see if it works.
#### node index.js
##### you should see this if it works <img width="497" alt="Screen Shot 2023-06-01 at 8 32 41 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/1839206d-776b-4d01-bf6a-89f11f9c361b">
### 


