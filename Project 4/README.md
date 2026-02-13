### **MEAN STACK DEPLOYMENT TO UBUNTU IN AWS**

**MEAN Stack** is a combination of the following components:

1. MongoDB (Document database) – Stores and allows retrieval of data.
2. Express (Back-end application framework) – Makes requests to Database for Reads and Writes.
3. Angular (Front-end application framework) – Handles Client and Server Requests. 
4. Node.js (JavaScript runtime environment) – Accepts requests and displays results to end user



**Preparing prerequisites**

In order to complete this project you will need an AWS account and a virtual server with Ubuntu Server OS.

### **Step 1- Install NodeJs:**
Node.js is a JavaScript runtime built on Chrome’s V8 JavaScript engine. Node.js is used in this tutorial to set up the Express routes and AngularJS controllers.

**Update Ubuntu**

`sudo apt update`

![alt text](<Images/Screenshot 2026-02-12 005041.png>)

**Upgrade ubuntu**

`sudo apt upgrade`

![alt text](<Images/Screenshot 2026-02-12 005112.png>)

**Add certificates**

 `sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates`

![alt text](<Images/Screenshot 2026-02-12 005215.png>)

 `curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -`

![alt text](<Images/Screenshot 2026-02-12 012647.png>)

 **Install NodeJS**

`sudo apt install -y nodejs`

![alt text](<Images/Screenshot 2026-02-12 012716.png>)

**Verify installation:**

`nodejs -v && npm -v`

![alt text](<Images/Screenshot 2026-02-12 012754.png>)


### **Step 2- Install MongoDB:**

**NOTE:** We would be using the MongoDB 8.0 repository, which was built specifically with native support for Ubuntu 24.04.



**Add the MongoDB 8.0 Repository (For Noble)**

Run these commands one by one. This uses the 8.0 GPG key and repo, which are the most compatible with the OS:

`curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | \
   sudo gpg --batch --yes -o /usr/share/keyrings/mongodb-server-8.0.gpg \
   --dearmor`


`echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list`

**Critical Update & Install**

Run: `sudo apt-get update` , `sudo apt-get install -y mongodb-org`

![alt text](<Images/Screenshot 2026-02-12 023052.png>)

**Start The server**

Run: `sudo systemctl daemon-reload` ,
`sudo systemctl start mongod`

**Verify the Service**

To make sure it’s actually running and not just installed, Run: `sudo systemctl status mongod`

You should see "active (running)" in green text.

![alt text](<Images/Screenshot 2026-02-12 025301.png>)


**Install body-parser package**

We need ‘body-parser’ package to help us process JSON files passed in requests to the server. Run: `sudo npm install body-parser`

**Create a folder named ‘Books’**

`mkdir Books && cd Books
`

**In the Books directory, Initialize npm project:**


`npm init`, click on enter for every prompt.

![alt text](<Images/Screenshot 2026-02-12 033851.png>)

Add a file to it named server.js

`nano server.js`

Copy and paste the web server code below into the server.js file.

```java var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes')(app);
app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});
```

### **Step 3 - Install Express and set up routes to the server:**

Express is a minimal and flexible `Node.js` web application framework that provides features for web and mobile applications. We will use Express to pass book information to and from our MongoDB database.

We also will use Mongoose package which provides a straightforward, schema-based solution to model your application data. We will use Mongoose to establish a schema for the database to store data of our book register.

In your Book directory run this command:

`sudo npm install express mongoose`

In Books folder, create a folder named apps

`mkdir apps && cd apps`

Create a file named routes.js

`nano routes.js`

Copy and paste the code below into routes.js

``` java var Book = require('./models/book');

module.exports = function (app) {

  // Get all books
  app.get('/book', async function (req, res) {
    try {
      const result = await Book.find({});
      res.json(result);
    } catch (err) {
      res.status(500).json({ error: err.message });
    }
  });

  // Add new book
  app.post('/book', async function (req, res) {
    try {

      const book = new Book({
        name: req.body.name,
        isbn: req.body.isbn,
        author: req.body.author,
        pages: req.body.pages
      });

      const result = await book.save();

      res.json({
        message: "Successfully added book",
        book: result
      });

    } catch (err) {
      res.status(500).json({ error: err.message });
    }
  });

  // Delete book
  app.delete('/book/:isbn', async function (req, res) {
    try {

      const result = await Book.findOneAndDelete({
        isbn: req.params.isbn
      });

      if (!result) {
        return res.status(404).json({
          message: "Book not found"
        });
      }

      res.json({
        message: "Successfully deleted the book",
        book: result
      });

    } catch (err) {
      res.status(500).json({ error: err.message });
    }
  });

  // Catch all routes
  app.all(/.*/, function (req, res) {
    res.status(404).json({ message: "Route not found" });
  });

};
```

![alt text](<Images/Screenshot 2026-02-13 054629.png>)

In the apps folder, create a folder named models

`mkdir models && cd models`

Create a file named book.js

`nano book.js`

Copy and paste the code below into `book.js`

``` java var mongoose = require('mongoose');
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
```

![alt text](<Images/Screenshot 2026-02-13 055656.png>)

![alt text](<Images/Screenshot 2026-02-13 055559.png>)

### **Step 4 – Access The Routes With AngularJS**

AngularJS provides a web framework for creating dynamic views in your web applications. In this tutorial, we use AngularJS to connect our web page with Express and perform actions on our book register.

Change the directory back to ‘Books’

`cd ../..`

Create a folder named public :

`mkdir public && cd public`

Add a file named script.js and copy and paste the code below in it :

`nano script.js`

``` java var app = angular.module('myApp', []);
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
```
![alt text](<Images/Screenshot 2026-02-13 065112.png>)

In the public folder, create a file named index.html and paste the code below in it :

`nano index.html`

``` html <!doctype html>
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
```
![alt text](<Images/Screenshot 2026-02-13 070208.png>)

Change the directory back up to Books

`cd ..`

Start the server by running this command:

`node server.js`

![alt text](<Images/Screenshot 2026-02-13 070434.png>)

You need to open TCP port 3300 in your AWS Web Console for your EC2 Instance to view your Book register web application from the internrt with your broswer using your Public IP.

![alt text](<Images/Screenshot 2026-02-13 070645.png>)

Then open your web browser and run your Public IP add with port 3300

![alt text](<Images/Screenshot 2026-02-13 070900.png>)




