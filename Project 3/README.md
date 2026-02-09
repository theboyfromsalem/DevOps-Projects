### SIMPLE TO-DO APPLICATION ON MERN WEBSTACK 
#### MERN Web stack consists of following components:

- **MongoDB:** A document-based, No-SQL database used to store application data in a form of documents.
- **ExpressJS:** A server-side Web Application framework for Node.js.
- **ReactJS:** A frontend framework developed by Facebook. It is based on JavaScript, used to build User Interface (UI) components.
- **Node.js:** A JavaScript runtime environment. It is used to run JavaScript on a machine rather than in a browser.
#### This document describes the steps used to implement MERN WEB STACK on an AWS EC2 instance. The screenshots taken during the process are included below in the order they were used to perform the setup.

#### Preparing prerequisites

- Create an EC2 instance of t2.micro family with Ubuntu Server of 22.04 LTS (HVM) image.

### STEP 1 - BACKEND CONFIGURATION.

Update Ubuntu, RUN

  `sudo apt update`

Ubuntu Upgrade, RUN

  `sudo apt upgrade`

In order to get the location of Node.js software from Ubuntu repositories run:

  `curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -`



![alt text](<Images/Screenshot 2026-02-06 153950.png>)

Install Node.js on the server with the command below:

 `sudo apt-get install -y nodejs`

![alt text](<Images/Screenshot 2026-02-06 160045.png>)
 
 **NOTE:** The command above installs both nodejs and npm. NPM is a package manager for Node like apt for Ubuntu, it is used to install Node modules & packages and to manage dependency conflicts.


Verify the node installation with the command below:

`node -v` 

Verify the node installation with the command below:

`npm -v` 

![alt text](<Images/Screenshot 2026-02-06 160742.png>)

Application Code Setup

Create a new directory for your To-Do project run: `mkdir Todo`

check to verify that the directory you created actually worked, run:
`ls`

- TIP: In order to see some more useful information about files and directories, you can use following combination of keys `ls -lih` – it will show you different properties and size in human readable format. You can learn more about different useful keys for ls command with `ls --help`.

Now change your current directory to the newly created one:

`cd Todo`

Next, you will use the command npm init to initialise your project, so that a new file named package.json will be created. This file will normally contain information about your application and the dependencies that it needs to run. Follow the prompts after running the command. You can press Enter several times to accept default values, then accept to write out the package.json file by typing yes.

`npm init`

![alt text](<Images/Screenshot 2026-02-06 163634.png>)

Run the command `ls` to confirm that you have package.json file created

![alt text](<Images/Screenshot 2026-02-06 163742.png>)

### INSTALL EXPRESSJS

To use **Express**, install it using npm:

`npm install express`

Now create a file index.js with the command below

`touch index.js`

Run `ls` to confirm that your index.js file is successfully created.

Install the dotenv module by running the command below:

`npm install dotenv`

![alt text](<Images/Screenshot 2026-02-06 164628.png>)

Open the index.js file with the command below:

`vim index.js`

Type the code below into it and save. Do not get overwhelmed by the code you see. For now, simply paste the code into the file.

``` java const express = require('express');
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
```

Notice that we have specified to use port 5000 in the code. This will be required later when we go on the browser.

Now it is time to start our server to see if it works. Open your terminal in the same directory as your index.js file and type:

`node index.js`

![alt text](<Images/Screenshot 2026-02-06 171450.png>)

***If everything goes well, you should see Server running on port 5000 in your terminal***

Now we need to open this port in EC2 Security Groups.

![alt text](<Images/Screenshot 2026-02-06 172635.png>)

Open up your browser and try to access your server’s Public IP or Public DNS name followed by port 5000:

http://PublicIP-or-PublicDNS:5000

![alt text](<Images/Screenshot 2026-02-06 175438.png>)

**ROUTES**

There are three actions that our To-Do application needs to be able to do:
- Create a new task
- Display list of all tasks
- Delete a completed task

Each task will be associated with some particular endpoint and will use different standard HTTP request methods: **POST**, **GET**, **DELETE**.

For each task, we need to create routes that will define various endpoints that the To-do app will depend on. So let us create a folder routes:

`mkdir routes`

Change directory to routes folder.

`cd routes`

Now, create a file api.js with the command below

`touch api.js`

Open the file with the command below

`nano api.js`

 Copy below code into the file.

 ``` java const express = require ('express');
const router = express.Router();
 
router.get('/todos', (req, res, next) => {
 
});
 
router.post('/todos', (req, res, next) => {
 
});
 
router.delete('/todos/:id', (req, res, next) => {
 
})
 
module.exports = router;
```

![alt text](<Images/Screenshot 2026-02-06 181355.png>)


MODELS
Now comes the interesting part, since the app is going to make use of Mongodb which is a NoSQL database, we need to create a model.
A model is at the heart of JavaScript based applications, and it is what makes it interactive.
We will also use models to define the database schema . This is important so that we will be able to define the fields stored in each Mongodb document. (Seems like a lot of information, but not to worry, everything will become clear to you over time. I promise!!!)
In essence, the Schema is a blueprint of how the database will be constructed, including other data fields that may not be required to be stored in the database. These are known as virtual properties
To create a Schema and a model, install mongoose which is a Node.js package that makes working with mongodb easier.
Change directory back Todo folder with `cd ..` and install Mongoose

`npm install mongoose`

Create a new folder models:

`mkdir models`

Change directory into the newly created ‘models’ folder with
`cd models`

Inside the models folder, create a file and name it todo.js

`touch todo.js`

Open the file created with `vim todo.js` then paste the code below in the file:

``` java const mongoose = require('mongoose');
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
```

![alt text](<Images/Screenshot 2026-02-06 182600.png>)

Now we need to update our routes from the file api.js in ‘routes’ directory to make use of the new model.
In Routes directory, open ***api.js*** with `vim api.js`, update the code in it and paste the code below into it, then save and exit.

``` java const express = require ('express');
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
```
The next piece of our application will be the MongoDB Database!

#### MONGODB DATABASE
We need a database where we will store our data. For this we will make use of mLab. mLab provides MongoDB database as a service solution (DBaaS), so to make life easy, you will need to sign up for a shared clusters free account, which is ideal for our use case. Sign up here; https://www.mongodb.com/cloud/atlas/register.

 Follow the sign up process, select AWS as the cloud provider, and choose a region near you.

Build a cluster and follow the steps in the images below :

![alt text](<Images/Screenshot 2026-02-06 212644.png>)

![alt text](<Images/Screenshot 2026-02-06 214423.png>)

![alt text](<Images/Screenshot 2026-02-06 220019.png>)

![alt text](<Images/Screenshot 2026-02-06 220500.png>)

![alt text](<Images/Screenshot 2026-02-06 220720.png>)

![alt text](<Images/Screenshot 2026-02-06 225856.png>)

![alt text](<Images/Screenshot 2026-02-06 225942.png>)

![alt text](<Images/Screenshot 2026-02-06 230226.png>)

In the `index.js` file, we specified `process.env` to access environment variables, but we have not yet created this file. So we need to do that now.

Create a file in your `Todo` directory and name it `.env`. by running the command below:

`touch .env`

`vi .env`

Add the connection string to access the database in it, just as below:

`DB = 'mongodb+srv://<username>:<password>@<network-address>/<dbname>?retryWrites=true&w=majority'`

Ensure to update `username`, `password`, `network-address` and `database` according to your setup

Here is how to get your connection string:

![alt text](<Images/Screenshot 2026-02-06 232615.png>)

![alt text](<Images/Screenshot 2026-02-06 232713.png>)

![alt text](<Images/Screenshot 2026-02-06 233131.png>)

Now we need to update the `index.js` to reflect the use of `.env` so that `Node.js` can connect to the database.

Simply delete existing content in the file, and update it with the entire code below by using vim or nano:

``` java const express = require('express');
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
```
![alt text](<Images/Screenshot 2026-02-06 233808.png>)

**NOTE:** Using environment variables to store information is considered more secure and best practice to separate configuration and secret data from the application, instead of writing connection strings directly inside the index.js application file.

Start your server using the command:

`node index.js` You shall see a message **‘Database connected successfully'**

![alt text](<Images/Screenshot 2026-02-07 011301.png>)

#### TESTING OUR BACKEND CODE

So far, we have built the backend of our To-Do application. This backend handles things like saving tasks, reading tasks, updating them, and deleting them from the database. We have also successfully connected our app to a database.

However, we do not have a frontend yet. That means there is no website or user interface where a user can click buttons or type tasks.

Because of this, we still need a way to check if our backend is working correctly during development.

To do that, we use a tool called an API development client. This type of tool allows us to send requests (like GET, POST, PUT, DELETE) directly to our backend and see the responses, without needing a frontend.

In this project, we will use **Postman**

Click the link below to downnload Postman and install postman on your machine.

https://www.postman.com/downloads

**Note:** Make sure your set header key `Content-Type` as `application/json`

![alt text](<Images/Screenshot 2026-02-07 021844.png>)

create a POST request to the API http://PublicIP-or-PublicDNS:5000/todos. This request sends a new task to our To-Do list so the application could store it in the database.

![alt text](<Images/Screenshot 2026-02-07 021940.png>)

Create a GET request to your API on http://PublicIP-or-PublicDNS:5000/todos. This request retrieves all existing records from out To-do application (backend requests these records from the database and sends it us back as a response to GET request).

![alt text](<Images/Screenshot 2026-02-07 021940.png>)

#### FRONTEND CREATION
Since we are done with the functionality we want from our backend and API, it is time to create a user interface for a Web client (browser) to interact with the application via API.

To start out with the frontend of the To-do app, we will use the create-react-app command to scaffold our app.

In the same root directory as your backend code, which is the Todo directory, run:
 `npx create-react-app client`

![alt text](<Images/Screenshot 2026-02-07 233303.png>)

 Install Concurrently. It is used to run more than one command simultaneously from the same terminal window.
 
 `npm install concurrently --save-dev`

 Install nodemon. It is used to run and monitor the server. If there is any change in the server code, nodemon will restart it automatically and load the new changes.

`npm install nodemon --save-dev`

![alt text](<Images/Screenshot 2026-02-07 174146.png>)

In `Todo` folder open the `package.json` file. Change the highlighted part of the below screenshot and replace with the code below.

```"scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
```
![alt text](<Images/Screenshot 2026-02-07 234554.png>)

Change directory to ‘client’

`cd client`

Open the package.json file

`nano package.json`

![alt text](<Images/Screenshot 2026-02-08 022047.png>)

Add the key value pair in the package.json file "proxy": "http://localhost:5000"

![alt text](<Images/Screenshot 2026-02-08 021920.png>)

The whole purpose of adding the proxy configuration in number 3 above is to make it possible to access the application directly from the browser by simply calling the server url like http://localhost:5000 rather than always including the entire path like http://localhost:5000/api/todos

Now, ensure you are inside the Todo directory, and simply do:

![alt text](<Images/Screenshot 2026-02-08 022242.png>)

Your app should open and start running on localhost:3000

![alt text](<Images/Screenshot 2026-02-08 022903.png>)

***Important note:*** In order to be able to access the application from the Internet you have to open TCP port 3000 on EC2 by adding a new Security Group rule. You already know how to do it.

From your Todo directory run:

`cd client`

move to the src directory

`cd src`

Inside your src folder create another folder called components

`mkdir components`

Move into the components directory with:

`cd components`

Inside ‘components’ directory create three files Input.js, ListTodo.js and Todo.js. by running this command below:

`touch Input.js ListTodo.js Todo.js`

![alt text](<Images/Screenshot 2026-02-08 025737.png>)

Open Input.js file

`nano Input.js`

Copy and paste the following:

``` java import React, { Component } from 'react';
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
```

To make use of Axios, which is a Promise based HTTP client for the browser and node.js, you need to cd into your client from your terminal and run yarn add axios or npm install axios.

Move your directory to client and paste the command below:

`npm install axios`

![alt text](<Images/Screenshot 2026-02-09 054349.png>)

Go to ‘components’ directory

cd src/components

After that open your ListTodo.js

nano ListTodo.js

![alt text](<Images/Screenshot 2026-02-09 054621.png>)

in the ListTodo.js copy and paste the following code 

``` java import React from 'react';
 
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
```

![alt text](<Images/Screenshot 2026-02-09 054642.png>)

Then in your Todo.js file, write the following code:

``` java import React, {Component} from 'react';
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
```

![alt text](<Images/Screenshot 2026-02-09 054830.png>)

We need to make little adjustment to our react code. Delete the logo and adjust our App.js look. Move to the src folder cd .. Make sure that you are in the src folder and run:
`nano App.js`

Copy and paste the code below into it:

``` java import React from 'react';
 
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
```
![alt text](<Images/Screenshot 2026-02-09 062602.png>)

In the src directory open the App.css

`nano App.css`

Then paste the following code into App.css:

```.App {
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
```
![alt text](<Images/Screenshot 2026-02-09 070948.png>)

In the src directory open the index.css

`nano index.css`

Copy and paste the code below:

``` body {
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
```
![alt text](<Images/Screenshot 2026-02-09 071107.png>)

Go to the Todo directory

`cd ../..`

When you are in the Todo directory run:

`npm run dev`

![alt text](<Images/Screenshot 2026-02-09 071311.png>)

Assuming no errors when saving all these files, our To-Do app should be ready and fully functional with the functionality discussed earlier: creating a task, deleting a task and viewing all your tasks.

![alt text](<Images/Screenshot 2026-02-09 071411.png>)