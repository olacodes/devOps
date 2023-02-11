# MERN STACK IMPLEMENTATION

## SIMPLE TO-DO APPLICATION ON MERN WEB STACK

In this project we are tasked to implement a web solution based on MERN stack in AWS Cloud.

The MERN technology stack stand for:

- M --> MongoDB
- E --> ExpressJS
- R --> ReactJS
- N --> Node.js

## DevOps Technologies used

- AWS CLI --> [Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html)
- AWS EC2 --> [Elastic Cloud Compute](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)

## DEPLOYMENT PROCESS WITH SCREEN SHOTS

### STEP 0: CREATE AN AWS EC2 INSTANCE

- Use Ubuntu OS

### STEP 1: BACKEND CONFIGURATION

#### Update and upgrade Ubuntu

```bash
   sudo apt update
   sudo apt upgrade
```

#### Get node from nodesource, install node and start todo app

```bash
# Get location of Node.js software from https://github.com/nodesource/distributions#deb
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -

# Install nodejs on the server
sudo apt-get install -y nodejs

# Check nodejs and npm version
node -v
npm -v

# Todo Application setup
mkdir Todo
cd Todo
npm init
```

#### Install ExpressJS and create routes

```bash
npm install express
# Install dotenv
npm install dotenv
# Create index.js file
touch index.js
```

#### Start nodejs server and create routes

```js
vim index.js

const express = require('express');
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

Then start the server with

```bash
node index.js
```

### NB: Edit you EC2 security group inbound rules to accept request from port 5000

Access your webpage on `http://<PublicIP-or-PublicDNS>:5000`

You can also get your AWS Public IP and Public DNS name

```bash
# For Public IP address
curl -s http://169.254.169.254/latest/meta-data/public-ipv4

# For DNS name
curl -s http://169.254.169.254/latest/meta-data/public-hostname
```

### Create Routes

There are three actions that our To-Do application needs to be able to do:

1. Create a new task
2. Display list of all tasks
3. Delete a completed task

```bash
mkdir routes
cd routes
touch api.js
vim api.js
```

Then past the following code

```js
const express = require("express");
const router = express.Router();

router.get("/todos", (req, res, next) => {});

router.post("/todos", (req, res, next) => {});

router.delete("/todos/:id", (req, res, next) => {});
module.exports = router;
```

### Create Models

Make sure you are in the Todo directory and run the following commands

```bash
npm install mongoose
mkdir models
cd models
touch todo.js
vim todo.js
```

then paste the following code

```js
const mongoose = require("mongoose");
const Schema = mongoose.Schema;

//create schema for todo
const TodoSchema = new Schema({
  action: {
    type: String,
    required: [true, "The todo text field is required"],
  },
});

//create model for todo
const Todo = mongoose.model("todo", TodoSchema);

module.exports = Todo;
```

### Updating out controller or api file

Now we need to update our routes from the file api.js in ‘routes’ directory to make use of the new model.

In Routes directory, open api.js with vim api.js, delete the code inside with :%d command and paste there code below into it then save and exit

```js
const express = require("express");
const router = express.Router();
const Todo = require("../models/todo");

router.get("/todos", (req, res, next) => {
  //this will return all the data, exposing only the id and action field to the client
  Todo.find({}, "action")
    .then((data) => res.json(data))
    .catch(next);
});

router.post("/todos", (req, res, next) => {
  if (req.body.action) {
    Todo.create(req.body)
      .then((data) => res.json(data))
      .catch(next);
  } else {
    res.json({
      error: "The input field is empty",
    });
  }
});

router.delete("/todos/:id", (req, res, next) => {
  Todo.findOneAndDelete({ _id: req.params.id })
    .then((data) => res.json(data))
    .catch(next);
});

module.exports = router;
```

### Setup MongoDB Database

Setup mongoDB [here](https://www.mongodb.com/atlas-signup-from-mlab) and create a cluster

Then get the connection string and place it in your .env like this

```bash
vim .env
DB=mongodb+srv://olacodes:<password>@cluster0.edfxehf.mongodb.net/?retryWrites=true&w=majority
```

Then connect your `index.js` to it

```js
const express = require("express");
const bodyParser = require("body-parser");
const mongoose = require("mongoose");
const routes = require("./routes/api");
const path = require("path");
require("dotenv").config();

const app = express();

const port = process.env.PORT || 5000;

//connect to the database
mongoose
  .connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log(`Database connected successfully`))
  .catch((err) => console.log(err));

//since mongoose promise is depreciated, we overide it with node's promise
mongoose.Promise = global.Promise;

app.use((req, res, next) => {
  res.header("Access-Control-Allow-Origin", "*");
  res.header(
    "Access-Control-Allow-Headers",
    "Origin, X-Requested-With, Content-Type, Accept"
  );
  next();
});

app.use(bodyParser.json());

app.use("/api", routes);

app.use((err, req, res, next) => {
  console.log(err);
  next();
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

Then run the app with

```bash
node index.js
# Then make an api post request to it to create a data
http://<PublicIP-or-PublicDNS>:5000/api/todos
{"action": "finish project 8 and 9"}

This should be successful and you can also do get, post and delete request
```

## STEP 2 – FRONTEND CREATION WITH REACT

In Todo folder run the create-react-app command like this

```bash
 npx create-react-app client

#  Install utils packages
1. [Concurrently](https://www.npmjs.com/package/concurrently): To run more than one command simultaneously from the same terminal window

npm install concurrently --save-dev

2.[Nodemon](https://www.npmjs.com/package/nodemon): It is used to run and monitor the server.

npm install nodemon --save-dev
```

In Todo folder open the todo `package.json` file change the script part to the following code.

```js
"scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
}
```

Then Add proxy rule to the client

```bash
cd client
vim package.json

# Add the key value pair in the package.json file

"proxy": "http://localhost:5000".

# The whole purpose of adding the proxy configuration above is to make it possible to access the application directly from the browser by simply calling the server url like http://localhost:5000 rather than always including the entire path like http://localhost:5000/api/todos

```

Then run `npm run dev` to start the app. This will start the app on port 3000. If you have not open port for 3000 in your EC2 security group pleas do so.

### Add frontend code

```bash
cd client
npm install axios # To make api calls
cd src
mkdir components
cd components
touch Input.js ListTodo.js Todo.js
```

Then create all the frontend from the following code [here](frontend-code.md)

Finally you should have something like this by going to `http://<PublicIP-or-PublicDNS>:3000`

[Todo page](assets/todo.png)
