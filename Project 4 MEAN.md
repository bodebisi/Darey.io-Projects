# MEAN STACK DEPLOYMENT TO UBUNTU IN AWS

### The MEAN stack is a JavaScript-based framework for developing web applications. MEAN is named after MongoDB, Express, Angular, and Node, the four key technologies that make up the layers of the stack. Angular is Frontend Development Framework whereas Node.js, Express, and MongoDB are used for Backend development
### Now we are going to be implementing a simple Book Register web form using MEAN stack.

### We will start by launching an EC2 instance in AWS console and connecting it to our Virtual Machine or Terminal. See Project 1 on how to do this.

## Step 1: Install NodeJs
#### sudo apt update
#### sudo apt upgrade
### Add certificates
#### sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
#### curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
### Install NodeJS
#### sudo apt install nodejs -y

## Step 2: Install MongoDB
### For our example application, we are adding book records to MongoDB that contain book name, isbn number, author, and number of pages.
#### sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
#### echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
### Install MongoDB
#### sudo apt install mongodb -y
### Start The server
#### sudo service mongodb start
### To confirm your mongobd is installed, run:
#### sudo systemctl status mongodb
### you should see this <img width="578" alt="Screen Shot 2023-06-01 at 10 07 20 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/dc864bbe-0f22-4a6e-9112-54d477ca37b1">
### Then confirm npm is installed, by running:
#### npm -v
### if it's already install we can continue if not, run:
#### sudo apt install npm -y
### confirm after installation with nmp -v
### Let's continue. Now we Install body-parser package. We need ‘body-parser’ package to help us process JSON files passed in requests to the server
#### sudo npm install body-parser
### Then Create a folder named ‘Books’
#### mkdir Books && cd Books
### In the Books directory, Initialize npm project
#### npm init
### Follow the prompts and fill the relevant informations
### Now create file server.js
#### vi server.js
#### paste this var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes')(app);
app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});
### to save press esc :wqa
## Step 3: Install Express and set up routes to the server
### Express is a minimal and flexible Node.js web application framework that provides features for web and mobile applications. We will use Express in to pass book information to and from our MongoDB database.
### So we start by installing Mongoose package which provides a straight-forward, schema-based solution to model your application data. We will use Mongoose to establish a schema for the database to store data of our book register.
#### sudo npm install express mongoose
### Now create a folder named apps
#### mkdir apps && cd apps
### Then Create a file named routes.js
#### vi routes.js
### Copy and paste the code below into routes.js then save and exit
#### const Book = require('./models/book');

module.exports = function(app) {
  app.get('/book', function(req, res) {
    Book.find({}).then(result => {
      res.json(result);
    }).catch(err => {
      console.error(err);
      res.status(500).send('An error occurred while retrieving books');
    });
  });

  app.post('/book', function(req, res) {
    const book = new Book({
      name: req.body.name,
      isbn: req.body.isbn,
      author: req.body.author,
      pages: req.body.pages
    });
    book.save().then(result => {
      res.json({
        message: "Successfully added book",
        book: result
      });
    }).catch(err => {
      console.error(err);
      res.status(500).send('An error occurred while saving the book');
    });
  });

  app.delete("/book/:isbn", function(req, res) {
    Book.findOneAndRemove(req.query).then(result => {
      res.json({
        message: "Successfully deleted the book",
        book: result
      });
    }).catch(err => {
      console.error(err);
      res.status(500).send('An error occurred while deleting the book');
    });
  });

  const path = require('path');
  app.get('*', function(req, res) {
    res.sendFile(path.join(__dirname, 'public', 'index.html'));
  });
};
### In the ‘apps’ folder, create a folder named models
#### mkdir models && cd models
### Create a file named book.js
#### vi book.js
### Copy and paste the code below into ‘book.js’ then save and exit
#### var mongoose = require('mongoose');
var dbHost = 'mongodb://localhost:27017/test';
mongoose.connect(dbHost);
mongoose.connection;
mongoose.set('debug', true);
var bookSchema = mongoose.Schema( {
  name: String,
  isbn: {type: String, index: true},
  author: String,
  pages: Number
});
var Book = mongoose.model('Book', bookSchema);
module.exports = mongoose.model('Book', bookSchema);

## Step 4 – Access the routes with AngularJS
### AngularJS provides a web framework for creating dynamic views in your web applications. In this tutorial, we use AngularJS to connect our web page with Express and perform actions on our book register.
### Change the directory back to ‘Books’ 
### Create a folder named public
#### mkdir public && cd public
### Add a file named script.js
#### vi script.js
### Copy and paste the Code below (controller configuration defined) into the script.js file, then save and exit.
#### var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope, $http) {
  $http( {
    method: 'GET',
    url: '/book'
  }).then(function successCallback(response) {
    $scope.books = response.data;
  }, function errorCallback(response) {
    console.log('Error: ' + response);
  });
  $scope.del_book = function(book) {
    $http( {
      method: 'DELETE',
      url: '/book/:isbn',
      params: {'isbn': book.isbn}
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
  $scope.add_book = function() {
    var body = '{ "name": "' + $scope.Name + 
    '", "isbn": "' + $scope.Isbn +
    '", "author": "' + $scope.Author + 
    '", "pages": "' + $scope.Pages + '" }';
    $http({
      method: 'POST',
      url: '/book',
      data: body
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
});

### In public folder, create a file named index.html
#### vi index.html
### Copy and paste the code below into index.html file, then save and exit.
#### <!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="script.js"></script>
  </head>
  <body>
    <div>
      <table>
        <tr>
          <td>Name:</td>
          <td><input type="text" ng-model="Name"></td>
        </tr>
        <tr>
          <td>Isbn:</td>
          <td><input type="text" ng-model="Isbn"></td>
        </tr>
        <tr>
          <td>Author:</td>
          <td><input type="text" ng-model="Author"></td>
        </tr>
        <tr>
          <td>Pages:</td>
          <td><input type="number" ng-model="Pages"></td>
        </tr>
      </table>
      <button ng-click="add_book()">Add</button>
    </div>
    <hr>
    <div>
      <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>

        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>

          <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        </tr>
      </table>
    </div>
  </body>
</html>

### Now Change the directory back up to Books
### Start the server by running this command:
#### node server.js
### you should see this <img width="533" alt="Screen Shot 2023-06-01 at 12 04 54 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/c0210ccf-0958-4c67-87ce-5555213197ff">
### Now that the server is now up and running, we can connect it via port 3300. For this – you need to open TCP port 3300 in your AWS Web Console for your EC2 Instance. <img width="1278" alt="Screen Shot 2023-06-01 at 11 41 04 AM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/308a7fc3-f68e-43ae-a5de-b706e616c6d0">
#### curl -s http://localhost:3300
### It shall return an HTML page, it is hardly readable in the CLI, but we can also try and access it from the Internet. 
### Quick reminder how to get your server’s Public IP and public DNS name: You can find it in your AWS web console in EC2 details, Run curl -s http://169.254.169.254/latest/meta-data/public-ipv4 for Public IP address or curl -s http://169.254.169.254/latest/meta-data/public-hostname for Public DNS name.
### This is how your Web Book Register Application will look like in browser: <img width="1123" alt="Screen Shot 2023-06-01 at 12 00 38 PM" src="https://github.com/bodebisi/Darey.io-Projects/assets/132711315/ad19b61d-1012-403d-8d90-df23f9540ab6">








