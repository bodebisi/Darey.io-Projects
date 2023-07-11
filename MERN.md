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
### You will be asked to input some details like i have done here <img width="1223" alt="Screen Shot 2023-06-01 at 8 19 49 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/6fe2309d-615e-4b6d-95f0-fd3e52c9aa85">
### Run the command ls to confirm that you have package.json file created.

### INSTALL EXPRESSJS
#### Express helps to define routes of your application based on HTTP methods and URLs, To use express, install it using npm:
#### npm install express
#### create a file index.js
#### touch index.js
#### Install the dotenv module
#### npm install dotenv
#### vim index.js
#### Type the code below into it and save.
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

#### Save and quit with esc :wqa
#### cat index.js to confirm the content
#### Notice that we have specified to use port 5000 in the code. This will be required later when we go on the browser.
#### Now it is time to start our server to see if it works.
#### node index.js
##### you should see this if it works <img width="497" alt="Screen Shot 2023-06-01 at 8 32 41 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/1839206d-776b-4d01-bf6a-89f11f9c361b">
### As we can see it works, so we need to go open our port 5000 in our security group in our AWS instance. see picture below 
#### <img width="1280" alt="Screen Shot 2023-07-10 at 7 25 10 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/b5e6a4ce-b83e-4a96-8533-54214981ac43">
### Now we open port 5000 with our public ip on our browser
#### <img width="1280" alt="Screen Shot 2023-07-10 at 7 20 56 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/5cf290e8-69d7-49df-9158-3f71aa462202">
#### Since our <publicip:5000> is opening on our browser. We can now move to our routes.
#### There are three actions that our To-Do application needs to be able to do:
### Create a new task
### Display list of all tasks
### Delete a completed task
#### Each task will be associated with some particular endpoint and will use different standard HTTP request methods: POST, GET, DELETE. For each task, we need to create routes that will define various endpoints that the To-do app will depend on. We start by creating a Routes directory.
#### mkdir routes
#### Then cd routes
#### Now, create a file api.js with the command below
#### touch api.js
#### Open the file with the command below
#### vim api.js
#### Copy below code in the file
#### const express = require ('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

})

module.exports = router;
#### You can confirm this with 
#### cat api.js

#### Now we can move to step 3 which will involve creating models.
#### Since the app is going to make use of Mongodb which is a NoSQL database, we need to create a model. A Model is at the heart of JavaScript based applications, and it is what makes it interactive.
#### We will also use models to define the database schema . This is important so that we will be able to define the fields stored in each Mongodb document. n essence, the Schema is a blueprint of how the database will be constructed, including other data fields that may not be required to be stored in the database. To create a Schema and a model, install mongoose which is a Node.js package that makes working with mongodb easier. 
#### Change directory back Todo folder with cd ..(make sure you are in the Todo folder by typing pwd and hitting enter) and install Mongoose
#### npm install mongoose
#### After the installation, we need to create the models folder.
#### mkdir models
#### Change directory into the newly created ‘models’ folder with
#### cd models
#### Inside the models folder, create a file and name it todo.js
#### touch todo.js
#### Open the file created with vim todo.js then paste the code below in the file:
#### const mongoose = require('mongoose');
const Schema = mongoose.Schema;

//create schema for todo
const TodoSchema = new Schema({
action: {
type: String,
required: [true, 'The todo text field is required']
}
})

//create model for todo
const Todo = mongoose.model('todo', TodoSchema);

module.exports = Todo;
#### Remember to type i to insert the code before you paste it and :wqa to exit.
#### Now we need to update our routes from the file api.js in ‘routes’ directory to make use of the new model.
#### In Routes directory, open api.js with vim api.js, delete the code inside with :%d command and paste there code below into it then save and exit
#### const express = require ('express');
const router = express.Router();
const Todo = require('../models/todo');

router.get('/todos', (req, res, next) => {

//this will return all the data, exposing only the id and action field to the client
Todo.find({}, 'action')
.then(data => res.json(data))
.catch(next)
});

router.post('/todos', (req, res, next) => {
if(req.body.action){
Todo.create(req.body)
.then(data => res.json(data))
.catch(next)
}else {
res.json({
error: "The input field is empty"
})
}
});

router.delete('/todos/:id', (req, res, next) => {
Todo.findOneAndDelete({"_id": req.params.id})
.then(data => res.json(data))
.catch(next)
})

module.exports = router;
#### The next piece of our application will be the MongoDB Database
### MONGODB DATABASE
#### We need a database where we will store our data. For this we will make use of mLab. mLab provides MongoDB database as a service solution (DBaaS)
### sign up to MongoDB.com <img width="911" alt="Screen Shot 2023-07-10 at 8 22 08 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/a4c7020f-af4d-40b1-86d9-5adcf507a817">

## Step 1
#### Create a database <img width="903" alt="Screen Shot 2023-07-10 at 8 22 40 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/6cc44719-fcad-4f1f-9775-e11a5ab3bccb">
#### By creating a cluster (I'd advise you choose a free cluster), select AWS as the cloud provider and choose a region near you see picture below. <img width="1278" alt="Screen Shot 2023-07-10 at 8 25 15 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/8058b2ed-9685-4dec-8b67-2abc9f217f79">
#### You'd also be required to authenticate your connection by creating a username & password or by a certificate see picture ![Screen Shot 2023-07-10 at 8 27 01 PM](https://github.com/bodebisi/Darey.io-Projects/assets/132711315/c96975ec-d5d8-4823-b5e5-b99c9ac6ecef)
### finish and close.
## step 2
### Then you have to select where you'd like to connect from? I chose anywhere, and remember to extend the time your entry will be deleted from 6hours to 1 week see picture. <img width="900" alt="Screen Shot 2023-07-10 at 8 48 25 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/a0748f32-c018-4118-8f63-9c9b5001742d"> 
#### Click Confirm
## Note: Allowing access to your database from anywhere is not recommended, usually it should be restricted to specified address. But since this is for developement and testing, that's why we are doing allowing.

## Step 3 Create a database
#### Go to database <img width="899" alt="Screen Shot 2023-07-10 at 9 04 42 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/36643817-31d7-4c44-a682-57f5c2a9a751">
#### Then browse Collections <img width="893" alt="Screen Shot 2023-07-10 at 9 06 20 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/09c684d6-fe73-4bdb-9f34-e24b1136f996">
#### Under collections, Add your data <img width="897" alt="Screen Shot 2023-07-10 at 9 10 42 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/0ff1ccf0-6c6b-408d-9899-bd9e62261e06">
#### Now create your database name and collection name <img width="887" alt="Screen Shot 2023-07-10 at 8 58 58 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/e5d16b67-f4dc-4e2c-927e-af29c6e34b6d">
## since we are done creating our database, we can now go back to our Terminal to continue the project.
### In the index.js file, we specified process.env to access environment variables, but we have not yet created this file. So we need to do that now.
### Create a file in your Todo directory and name it .env.
#### touch .env
#### vi .env
### Go to your mongoDB, connect your cluster <img width="902" alt="Screen Shot 2023-07-10 at 9 40 52 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/cb267046-5ab4-46d7-a3f4-8f2267f034b1">
#### <img width="798" alt="Screen Shot 2023-07-10 at 9 43 01 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/c8d1bbf2-349e-4579-ba12-dc2be35c5fe7">
#### Add the connection string to access the database in it, just as below: <img width="800" alt="Screen Shot 2023-07-10 at 9 47 17 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/4b1ccab6-5a4e-495f-b4f8-4be3558cb2a0">
#### Ensure to update,password and database according to your setup

### Now we need to update the index.js to reflect the use of .env so that Node.js can connect to the database.
#### Simply delete existing content in the file, and update it with the entire code below.
#### To do that using vim, follow below steps
#### open the file with vim index.js
#### Press esc
#### Type :
#### Type %d
#### Hit ‘Enter’
#### Press i to enter the insert mode in vim
#### Now, paste the entire code below in the file.

#### const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

//connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
.then(() => console.log(`Database connected successfully`))
.catch(err => console.log(err));

//since mongoose promise is depreciated, we overide it with node's promise
mongoose.Promise = global.Promise;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use(bodyParser.json());

app.use('/api', routes);

app.use((err, req, res, next) => {
console.log(err);
next();
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});

### Start your server using the command:
#### node index.js
### You shall see a message ‘Database connected successfully’, if so – we have our backend configured.
#### <img width="572" alt="Screen Shot 2023-07-10 at 10 01 48 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/1e299278-9fa9-4611-aeaa-eb1c9c8df0e8">

#### So far we have written backend part of our To-Do application, and configured a database, but we do not have a frontend UI yet. We need ReactJS code to achieve that. But during development, we will need a way to test our code using RESTfull API. Therefore, we will need to make use of some API development client to test our code.

#### In this project, we will use Postman to test our API.
#### Click Install Postman to download and install postman on your machine.
### Now open your Postman, create a POST request to the API http://<PublicIP-or-PublicDNS>:5000/api/todos. This request sends a new task to our To-Do list so the application could store it in the database.
#### Note: make sure your set header key Content-Type as application/json see picture <img width="911" alt="Screen Shot 2023-07-10 at 10 26 05 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/8c85eb18-15e6-4839-b054-e3f1ad1f1d9c">
#### Then go to body and type your message and click send like this <img width="876" alt="Screen Shot 2023-07-10 at 10 25 32 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/1115be87-28cb-4e67-bbba-e9af8ff53cd8">
#### Create a GET request to your API on http://<PublicIP-or-PublicDNS>:5000/api/todos.repeat the process you did in the post request.  <img width="923" alt="Screen Shot 2023-07-10 at 10 34 34 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/8c8a47ab-0a7a-406e-bffb-48f4c597fa3e">
#### This request (The GET request) retrieves all existing records from out To-do application (backend requests these records from the database and sends it us back as a response to GET request). 

## STEP 2 – FRONTEND CONFIGURATION

#### Since we are done with the functionality we want from our backend and API, it is time to create a user interface for a Web client (browser) to interact with the application via API. To start out with the frontend of the To-do app, we will use the create-react-app command to scaffold our app.
#### In the same root directory as your backend code, which is the Todo directory, run:
####  npx create-react-app client
### This will create a new folder in your Todo directory called client, where you will add all the react code
### Before testing the react app, there are some dependencies that need to be installed

### Install concurrently. It is used to run more than one command simultaneously from the same terminal window.
#### npm install concurrently --save-dev

### Install nodemon. It is used to run and monitor the server. If there is any change in the server code, nodemon will restart it automatically and load the new changes.
#### npm install nodemon --save-dev

### Now in Todo folder open the package.json file and replace the scripts with the code below.
#### "scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
},

### Configure Proxy in package.json
#### Change directory to ‘client’
#### cd client
#### Open the package.json file
#### vi package.json
#### Add the key value pair in the package.json file "proxy": "http://localhost:5000" like this <img width="571" alt="Screen Shot 2023-07-10 at 11 02 29 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/b7afdb74-adcb-455a-92eb-12d8fc67c30a">
### The whole purpose of adding the proxy configuration in number 3 above is to make it possible to access the application directly from the browser by simply calling the server url like http://localhost:5000 rather than always including the entire path like http://localhost:5000/api/todos

#### Now, ensure you are inside the Todo directory, and simply do:
#### npm run dev
#### Your app should open and start running on localhost:3000
#### Go to your security group in your AWS, edit inbound rules and open port 3000 see picture <img width="1233" alt="Screen Shot 2023-07-10 at 11 19 16 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/6e7e0470-7f43-4b39-97b0-571132ebd76e"> 

#### Your app should open and start running on localhost:3000 <img width="1275" alt="Screen Shot 2023-07-10 at 11 08 26 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/27e8737f-4086-4ab2-bee5-60d2adee436a">

### Creating your React Components
#### One of the advantages of react is that it makes use of components, which are reusable and also makes code modular. For our Todo app, there will be two stateful components and one stateless component.
### From your Todo directory run
#### cd client
#### move to the src directory
#### cd src
### Inside your src folder create another folder called components
#### mkdir components
### Move into the components directory with
#### cd components
### Inside ‘components’ directory create three files Input.js, ListTodo.js and Todo.js.
#### touch Input.js ListTodo.js Todo.js
#### Open Input.js file
#### vi Input.js
### Copy and paste the following

#### import React, { Component } from 'react';
import axios from 'axios';

class Input extends Component {

state = {
action: ""
}

addTodo = () => {
const task = {action: this.state.action}

    if(task.action && task.action.length > 0){
      axios.post('/api/todos', task)
        .then(res => {
          if(res.data){
            this.props.getTodos();
            this.setState({action: ""})
          }
        })
        .catch(err => console.log(err))
    }else {
      console.log('input field required')
    }
}
handleChange = (e) => {
this.setState({
action: e.target.value
})
}
render() {
let { action } = this.state;
return (
<div>
<input type="text" onChange={this.handleChange} value={action} />
<button onClick={this.addTodo}>add todo</button>
</div>
)
}
}
export default Input

### To make use of Axios, which is a Promise based HTTP client for the browser and node.js, you need to cd into your client from your terminal and run yarn add axios or npm install axios.
### Move to the src folder
#### cd ..
### Move to clients folder
#### cd ..
### Install Axios
#### npm install axios 

### Go to ‘components’ directory
#### cd src/components
### After that open your ListTodo.js
#### vi ListTodo.js
### in the ListTodo.js copy and paste the following code

#### import React from 'react';
const ListTodo = ({ todos, deleteTodo }) => {
return (
<ul>
{
todos &&
todos.length > 0 ?
(
todos.map(todo => {
return (
<li key={todo._id} onClick={() => deleteTodo(todo._id)}>{todo.action}</li>
)
})
)
:
(
<li>No todo(s) left</li>
)
}
</ul>
)
}
export default ListTodo

### Then in your Todo.js file you write the following code

### import React, {Component} from 'react';
import axios from 'axios';
import Input from './Input';
import ListTodo from './ListTodo';
class Todo extends Component {
state = {
todos: []
}
componentDidMount(){
this.getTodos();
}
getTodos = () => {
axios.get('/api/todos')
.then(res => {
if(res.data){
this.setState({
todos: res.data
})
}
})
.catch(err => console.log(err))
}
deleteTodo = (id) => {

    axios.delete(`/api/todos/${id}`)
      .then(res => {
        if(res.data){
          this.getTodos()
        }
      })
      .catch(err => console.log(err))
}
render() {
let { todos } = this.state;
    return(
      <div>
        <h1>My Todo(s)</h1>
        <Input getTodos={this.getTodos}/>
        <ListTodo todos={todos} deleteTodo={this.deleteTodo}/>
      </div>
    )
}
}
export default Todo;

### We need to make little adjustment to our react code. Delete the logo and adjust our App.js to look like this.

### Move to the src folder
### cd ..
### Make sure that you are in the src folder and run
#### vi App.js
### Copy and paste the code below into it

### import React from 'react';
import Todo from './components/Todo';
import './App.css';
const App = () => {
return (
<div className="App">
<Todo />
</div>
);
}
export default App;

### After pasting, exit the editor.
### In the src directory open the App.css
#### vi App.css
### Then paste the following code into App.css:

### .App {
text-align: center;
font-size: calc(10px + 2vmin);
width: 60%;
margin-left: auto;
margin-right: auto;
}
input {
height: 40px;
width: 50%;
border: none;
border-bottom: 2px #101113 solid;
background: none;
font-size: 1.5rem;
color: #787a80;
}
input:focus {
outline: none;
}
button {
width: 25%;
height: 45px;
border: none;
margin-left: 10px;
font-size: 25px;
background: #101113;
border-radius: 5px;
color: #787a80;
cursor: pointer;
}
button:focus {
outline: none;
}
ul {
list-style: none;
text-align: left;
padding: 15px;
background: #171a1f;
border-radius: 5px;
}
li {
padding: 15px;
font-size: 1.5rem;
margin-bottom: 15px;
background: #282c34;
border-radius: 5px;
overflow-wrap: break-word;
cursor: pointer;
}
@media only screen and (min-width: 300px) {
.App {
width: 80%;
}
input {
width: 100%
}
button {
width: 100%;
margin-top: 15px;
margin-left: 0;
}
}
@media only screen and (min-width: 640px) {
.App {
width: 60%;
}
input {
width: 50%;
}
button {
width: 30%;
margin-left: 10px;
margin-top: 0;
}
}
Exit

### In the src directory open the index.css
#### vim index.css
### Copy and paste the code below:

### body {
margin: 0;
padding: 0;
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen",
"Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
sans-serif;
-webkit-font-smoothing: antialiased;
-moz-osx-font-smoothing: grayscale;
box-sizing: border-box;
background-color: #282c34;
color: #787a80;
}
code {
font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New",
monospace;
}

### Go to the Todo directory
#### cd ../..
### When you are in the Todo directory run:
#### npm run dev

### Assuming no errors when saving all these files, our To-Do app should be ready and fully functional with the functionality discussed earlier: creating a task, deleting a task and viewing all your tasks
<img width="1279" alt="Screen Shot 2023-07-11 at 8 38 00 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/73e49752-bce8-4475-a5d8-b520da65f0d5">

### Congratulations
### In this Project #3 you have created a simple To-Do and deployed it to MERN stack. You wrote a frontend application using React.js that communicates with a backend application written using Expressjs. You also created a Mongodb backend for storing tasks in a database.













