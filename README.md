# 🚀 End-to-End MERN Stack Deployment on AWS EC2

- Author: Maxwell Ugochukwu
- Role: DevOps Engineer
- Date: August 14, 2025
- Reading time: ~15 minutes
---
A comprehensive step-by-step guide to deploying a **MERN Stack (MongoDB, Express.js, React.js, Node.js)** application on **AWS EC2**.  
This guide covers backend and frontend configuration, MongoDB Atlas setup, and complete deployment.

---

## 📖 Table of Contents

1. [About MERN Stack](#about-mern-stack)
2. [Architecture Overview](#architecture-overview)
3. [Prerequisites](#prerequisites)
4. [Deployment Guide](#deployment-guide)
   - [Backend Configuration](#backend-configuration)
   - [Application Code Setup](#application-code-setup)
   - [Routes](#routes)
   - [Models](#models)
   - [MongoDB Atlas Database Setup](#mongodb-atlas-database-setup)
   - [Testing Backend Code Without Frontend using RESTful API (Postman)](#testing-backend-code-without-frontend-using-restful-api-postman)
   - [Frontend Configuration](#frontend-configuration)
5. [Running the Application](#running-the-application)
6. [Final Notes](#final-notes)

---

## 📌 About MERN Stack

The **MERN Stack** is a popular JavaScript-based technology stack used for building full-stack web applications.  
It includes:

- **MongoDB** – NoSQL database for storing application data.
- **Express.js** – Web framework for Node.js to handle routes, middleware, and API requests.
- **React.js** – Frontend library for building interactive UIs.
- **Node.js** – JavaScript runtime for executing server-side logic.

This stack allows **end-to-end development** using JavaScript — from the database to the user interface.

---

## 🏗 Architecture Overview

Below is the high-level architecture of a MERN application deployed on AWS EC2:

```pgsql
                          +--------------------+
                          |     Browser        |
                          |   (User Client)    |
                          +---------+----------+
                                    |
                                    | HTTP Requests
                                    v
                          +---------+----------+
                          |   React.js Frontend|
                          +---------+----------+
                                    |
                                    | API Calls (Axios)
                                    v
                          +---------+----------+
                          | Express.js Backend |
                          +---------+----------+
                                    |
               +--------------------+--------------------+
               |                                         |
               | CRUD Operations                         | Runs on
               v                                         v
     +---------+----------+                     +--------+---------+
     |    MongoDB Atlas   |                     |   Node.js Runtime|
     +--------------------+                     +--------+---------+
                                                          |
                                                          | Hosted on
                                                          v
                                                +---------+----------+
                                                | Amazon EC2 Instance|
                                                +--------------------+
```

## 🛠 Prerequisites

Before starting deployment, ensure you have:

- ✅ An AWS Account [Sign up here](aws.amazon.com)
- ✅ Windows OS installed
- ✅ An AWS EC2 instance running Ubuntu 20.04 or later
- ✅ Ability to SSH into EC2 instance
- ✅ Basic understanding of JavaScript and Linux commands

## 🚀 Deployment Guide

### Backend Configuration

#### 1️⃣ Runtime Setup

```bash
# Update and upgrade packages
sudo apt update && sudo apt upgrade -y

# Install Node.js & npm
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify installation
node -v
npm -v
```

![Alt text describing image](images/be1.png)
![Alt text describing image](images/be2.png)
![Alt text describing image](images/be3.png)
![Alt text describing image](images/be5.png)


#### Application Code Setup

```bash
# Create project directory
mkdir Todo && cd Todo

# Initialize Node.js project
npm init
# Accept defaults by pressing Enter
```
![Alt text describing image](images/be6.png)
![Alt text describing image](images/be7.png)
![Alt text describing image](images/be8.png)

#### Install Express.js

Express is a minimal and flexible Node.js framework that provides robust features for web and mobile applications — including routing and middleware handling.

```bash
npm install express
touch index.js
sudo npm install dotenv
```

![Alt text describing image](images/ej1.png)
![Alt text describing image](images/ej2.png)
![Alt text describing image](images/ej3.png)

- **index.js** `nano index.js`
![Alt text describing image](images/ej4.png)

  - Write in the code:

    ```javascript
    const express = require('express');
    require('dotenv').config();

    const app = express();
    const port = process.env.PORT || 5000;

    app.use((req, res, next) => {
      res.header("Access-Control-Allow-Origin", "*");
      res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
      next();
    });

    app.use((req, res) => {
      res.send('Welcome to Express');
    });

    app.listen(port, () => {
      console.log(`Server running on port ${port}`);
    });
    ```

    ![Alt text describing image](images/ej5.png)

- **Start the server:**

  ```bash
  node index.js
  ```
  ![Alt text describing image](images/ej6.png)

- **✅ Create an inbound rule in AWS EC2 Security Group to allow port 5000.**

  ![Alt text describing image](images/ej7.png)

- Test: `http://<PublicIP-or-PublicDNS>:5000`

  **A quick reminder on how to get your server's Public IP or DNS name:**

  - AWS web console in Ec2 details
  - Run 
    - `curl -s http://169.254.169.254/latest/meta-data/public-ipv4`
  for Public IP address 
    - or `curl -s http://169.254.169.254/latest/meta-data/public-hostname`
  for Public DNS.

  You should see: `Welcome to Express`

  ![Alt text describing image](images/ej8.png)

### Routes

Three actions for the To-Do app:

- POST – Create new task
- GET – Retrieve tasks
- DELETE – Remove tasks

_Each task is associated with a particular endpoint and will use standard HTTP request methods: POST, GET, DELETE.
We need to create routes that define various endpoints that the To-do app will depend on. Create a folder named routes and change directory to routes_

```bash
mkdir routes && cd routes
nano api.js
cd ..
```
![Alt text describing image](images/ej9.png)
![Alt text describing image](images/ej10.png)

- create api.js file and paste the code below `nano api.js`

```javascript
const express = require('express');
const router = express.Router();

router.get('/todos', (req, res) => {});
router.post('/todos', (req, res) => {});
router.delete('/todos/:id', (req, res) => {});

module.exports = router;
```

### Models

Using Mongoose to define MongoDB schema and interact with the database. To create a schema and a model, install mongoose which is a node.js package that makes working with mongodb easier and create a new folder, change  dir into the new  folder and create a file name it todo.js

```bash
npm install mongoose
mkdir models && cd models && touch todo.js
```
![Alt text describing image](images/db1.png)
![Alt text describing image](images/db2.png)

- Open todo.js created with nano then paste the code below `nano todo.js`
  ![Alt text describing image](images/db3a.png)

- Write in and save

  ```javascript
  const mongoose = require('mongoose');
  const Schema = mongoose.Schema;

  const TodoSchema = new Schema({
    action: { type: String, required: [true, 'The todo text field is required'] }
  });

  const Todo = mongoose.model('todo', TodoSchema);
  module.exports = Todo;
  ```

  ![Alt text describing image](images/db3.png)

- Now update routes from the file `api.js` in routes dir to make use of the new model, open `api.js` with nano, delete the code inside and paste the code below, save and close.

  ```
  nano api.js
  ```

  ```javascript
  const express = require ('express');
  const router = express.Router();
  const Todo = require('../models/todo');

  router.get('/todos', (req, res, next) => {

  //this will return all the data, exposing only the id and action field to the client
  Todo.find({}, 'action')
  .then(data => res.json(data))
  .catch(next)
  });

  router.post('/todos', (req, res, next) => {
  if(req.body.action) {
  Todo.create(req.body)
  .then(data => res.json(data))
  .catch(next)
  }else {
  res.json({
  error:  "The input field is empty"
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

  ![Alt text describing image](images/db4a.png)
  ![Alt text describing image](images/db4.png)

### MongoDB Atlas Database Setup

Create a database where we will store our data. Here we will make use  of Atlas which provides MongoDB database as a service solution(DBaas).

- Sign up: https://cloud.mongodb.com
![Alt text describing image](images/db8.png)
- Pick your Cloud provider
![Alt text describing image](images/db7.png)
- Create free shared cluster
![Alt text describing image](images/db6.png)
- Allow access from anywhere
- Create DB user & password
![Alt text describing image](images/db9.png)
- Copy connection string
![Alt text describing image](images/db10.png)
- Install MongoDB

  `npm install mongodb`

- Create .env file, paste your MongDB connection string and save:

  ```
  DB=mongodb+srv://dbadmin:<password>@clustermern.mongodb.net/ retryWrites=true&w=majority
  ```
  ![Alt text describing image](images/db11.png)
  ![Alt text describing image](images/.envcorrected.png)

- Update index.js to reflect the use of .env `nano index.js` and replace with code below, save and exit:

  ```javascript
  const express = require('express');
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
  ![Alt text describing image](images/db13.png)
  ![Alt text describing image](images/db14.png)

_Using environment variable to store information is more secured and  bet  practice to separate configuration and secret data from the application, instead of writing connection strings directly inside the index.js application file_

- Start server: You should see `Database connected successfully` if rightly done 
  ```
  node index.js
  ```
  ![Alt text describing image](images/db15.png)

### Testing Backend Code Without Frontend using RESTful API (Postman)

- Click [Install Postman](postman.com) to download and install postman on your machine. Test all the API endpoints and make sure they are working
![Alt text describing image](images/tb1.png)
![Alt text describing image](images/td2.png)

1. POST → Create a task `http://<PublicIP>:5000/api/todos` Set key value as Content-Type: application/json and add a task to the To-Do list

    ```
    Body (raw JSON):

    {
      "action": "Finish Project MERN"
    }
    ```
    ![Alt text describing image](images/td3.png)
    ![Alt text describing image](images/td4.png)
    ![Alt text describing image](images/td5.png)

2. GET → Retrieve all tasks `http://<PublicIP>:5000/api/todos`
![Alt text describing image](images/td6.png)

3. DELETE → Remove a specific task `http://<PublicIP>:5000/api/todos/<task-id>`
![Alt text describing image](images/td7.png)

### Frontend Configuration

In the same root directory as your backend code Todo dir, run

```bash
npx create-react-app client
```
![Alt text describing image](images/fe1.png)

This will create a new folder in your Todo dir called client, where you will add all the react code.

- Install required dependencies needed for a React app
  - Install concurrently. It is used to run more than one command simultaneously from the same terminal window

    ```bash
    npm install concurrently --save-dev
    ```

  - Install nodemon. It is used to run and monitor the server. If there is any change in the server code, nodemon will restart it automatically and load changes.

    ```
    npm install nodemon --save-dev
    ```
    ![Alt text describing image](images/fe2.png)

- In Todo folder open the package.json file. Replace the Script part with the code below  

  ```json
  "scripts": {
      "start": "node index.js",
      "start-watch": "nodemon index.js",
      "dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
    },
  ```
  ![Alt text describing image](images/fe3.png)
  ![Alt text describing image](images/fe4.png)
  ![Alt text describing image](images/fe6.png)

- Configure Proxy in package.json: Change directory to client
`cd client` and Open the package.json file `nano package.json` Add the key value pair in the package.json file 

  ```json
  "proxy": "http://localhost:5000"
  ```
  ![Alt text describing image](images/fe8.png)

- Run `npm run dev` and ensure `port 3000` is open on Ec2
![Alt text describing image](images/fe9.png)
![Alt text describing image](images/fe8a.png)
![Alt text describing image](images/fe8b.png)

Your app should open and start running on port 3000

### Creating your React Components

One of the advantages of react is that it makes use of components, which are reusable and also makes code modular. There will be two stateful components and one stateless component for our Todo app.

- In Todo dir, run
  ```bash
  cd client/src
  ```

- Inside your src folder create another folder called components and move into the component dir.

  ```bash
  mkdir components && cd components
  ```
  ![Alt text describing image](images/z1.png)

- Inside components dir create three files

  ```bash
  touch Input.js ListTodo.js Todo.js
  ```
  ![Alt text describing image](images/z2.png)

- `nano input.js`
![Alt text describing image](images/z3a.png)

  Paste:

  ```javascript
  import React, { Component } from 'react';
  import axios from 'axios';

  class Input extends Component {

    state = {
      action: ""
    }

    addTodo = () => {
      const task = { action: this.state.action }

      if (task.action && task.action.length > 0) {
        axios.post('/api/todos', task)
          .then(res => {
            if (res.data) {
              this.props.getTodos();
              this.setState({ action: "" })
            }
          })
          .catch(err => console.log(err))
      } else {
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

  export default Input;
  ```
  ![Alt text describing image](images/z3.png)

- We will make use of axios, which is a promise based HTTP client for the browser and node.js. cd into your client from your terminal `cd ../../`

    - Install Axios

      ```bash    
      npm install axios
      ```
      ![Alt text describing image](images/z4.png)

- Go to components dir

  ```
  cd src/components
  ```

- Open your ListTodo.js file `nano ListTodo.js`
![Alt text describing image](images/z5a.png)
  - paste

    ```javascript
    import React from 'react';

    const ListTodo = ({ todos, deleteTodo}) => {

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
    </ul>
    )
    }

    export default ListTodo;
    ```
    ![Alt text describing image](images/z5.png)

- In Todo.js file `nano Todo.js` write the following code
![Alt text describing image](images/z6a.png)

  ```javascript
  import React, {Component} from 'react';
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
  ![Alt text describing image](images/z6.png)

- Move to the src folder, In `App.js` delete the logo and adjust to look like below

  ```bash
  cd ..
  nano App.js
  ```
  ![Alt text describing image](images/z7a.png)

  - Paste

    ```js  
    import React from 'react';

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
    ![Alt text describing image](images/z7.png)

- In the src dir, open the App.css `nano App.css` and write
![Alt text describing image](images/z8a.png)

  ```css
  .App {
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
      width: 100%;
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
  ![Alt text describing image](images/z8.png)

- In the src dir open the index.css `nano index.css` and enter the code below:

  ```css
  body {
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
  ![Alt text describing image](images/z9a.png)
  ![Alt text describing image](images/z9.png)

##  Running the Application
- Go to the Todo directory

```bash
cd ../../
```

- In the Todo dir. run

```bash
npm run dev
```
![Alt text describing image](images/z10.png)

**The Todo app should be ready and fully functional with the functionality discussed earlier: creating a task, deleting a task and viewing all your task. Access it in your browser `http://<Public-IP-or-Public-DNS>:3000` you should see the UI below:**

![Alt text describing image](images/z11.png)

## Final Notes

During the deployment process, several issues came up that are worth noting for future reference. Below are the problems and their solutions.

- **React App Not Rendering After Deployment:**

  - Issue: Blank page or default React logo screen after deploying.
    
  - Cause: The default `App.js` from `create-react-app` wasn’t replaced with the correct component imports.

  - Solution: Ensure `App.js` imports and renders the correct component (e.g., `<Todo />`). Remove unused default code from create-react-app.

- **Unterminated Regular Expression Error in JSX**

  - Issue: Error like Unterminated regular expression appearing during build.

  - Cause: Missing closing curly brace `}` in JSX expressions inside components.

  - Solution: Carefully check JSX expressions for correct opening `{` and closing `}` pairs. Example: In `ListTodo.js`, add the missing `}` before `</ul>`.

- **Port Conflict When Running Backend and Frontend**

  - Issue: Backend and frontend both trying to use the same port.

  - Cause: No proxy configuration or concurrent run setup.

  - Solution: Use concurrently to run both frontend and backend in one command. Add `"proxy": "http://localhost:<backend_port>"` to `client/package.json` for API requests.

- **MongoDB Connection Issues**

  - Issue: App failing to connect to MongoDB.

  - Cause: Wrong connection URI, missing `.env` file, or `MongoDB` service not running.

  - Solution: Check MongoDB URI format (especially for Atlas:  `mongodb+srv://username:password@cluster.mongodb.net/dbname`). Ensure `.env` file is loaded in `server.js` using `dotenv.config()`. Verify `MongoDB service` is running locally or network access is open for Atlas.

- **Public IP/DNS Not Working After EC2 Deployment**

  - Issue: Frontend/backend unreachable after deploying to AWS EC2.

  - Cause: Security group inbound rules not set or server bound to localhost.

  - Solution: Add inbound rule for `HTTP (port 80)` or custom port in EC2 security group. In Express, listen on `0.0.0.0 ` instead of `localhost`.


