In a [previous tutorial]() we created a task list application in the MEAN Stack. Today we will be recreating the same application in a slightly different way. This tutorial will focus on replacing the "A" in MEAN, AngularJS, with another popular MVVC framework called Vue.js. Unlike other Vue.js tutorials for beginners, this tutorial will introduce Vue.js in the context of the full stack. (MongoDB, Vue.js, Express and NodeJS)

As before, our application will be structured in 3 Microservices; a MongoDB microservice, a NodeJS API. This time however, we will replace AngularJS in favor of Vue.js in the front end. You will be using the best practices introduced earlier: Microservices, a RESTful API,  and building front-end application to use that RESTful API.

We will be creating:

+ Single page application written in Vue.js to create and complete tasks.
+ A MongoDB datastore through Mongoose to store our tasks.
+ Creating a RESTful webservice in NodeJS that gets a list of tasks, creates a task and deletes a task.
+ Set up a development environment with an integration to Github to to manage our development lifecycle.

In many ways, Vue.js is not as complicated as AngularJS. Many of the concepts and structure are the same, though.
There is a very good overview of the differences between the two frameworks on the [Vue.js website](https://vuejs.org/v2/guide/comparison.html#Angular-1).

# Vue.js Tutorial: Vue.js full-stack application with MongoDB, Express and NodeJS.

## Development Environment Setup

If you haven't set up development environment with GitHub and Digital Garage, you can follow [this link](https://github.com/join?source=header-home) to sign up for a free account on GitHub. For a free account on Digital Garage, follow [this link](https://cochera.thedigitalgarage.io/free-signup).

After you have signed up for/signed into your Github account, fork the [thedigitalgarage/vue.js](https://github.com/thedigitalgarage/vuejs-ex) repository into your own account. This repository contains some files and a file structure that will give you a quick start on your Vue.js application.

After signing into your Digital Garage account, deploy the Vue.js Full-stack Quickstart (qs-mven). There's a detailed walkthrough of setting up an application through the web console and command line interface on the [Digital Garage documentation site](http://docs.thedigitalgarage.io/getting_started/basic_walkthrough.html).

## File Structure

To keep it simple and provide a good basis for comparison, I am going to follow the same file structure that I used for the MEAN Stack tutorial.

```

    - app                       <!-- holds all our files for node components (models, routes) -->
    ----- models
    ---------- any-model.js     <!-- defines the mongoose/mongodb model -->
    ----- routes.js             <!-- all routes will be handled here -->

    - config                    <!-- all our configuration will be here -->
    ----- database.js

    - public                    <!-- holds all our files for our front-end vue.js application -->
    ----- app.js                <!-- all vue.js code for our app -->
    ----- index.html            <!-- main view -->

    - package.json              <!-- npm configuration to install dependencies/modules -->
    - server.js                 <!-- Node configuration -->

```

## Installing Modules

In Node, the `package.json` file holds the configuration for our app. Node’s package manager (npm) will use this to install any dependencies or modules that we are going to use. In our case, the Digital Garage Quickstart template read the package.json file and installed all of the packages we need. As with a standard npm install, you can change the `package.json` to include or exclude modules. Again, we will be using [Express](http://expressjs.com/) and [Mongoose](http://mongoosejs.com/) (a popular object modeling module for MongoDB). We have the option here of installing Vue.js as a package via npm. For simplicity, I am loading the Vue.js javascript file in our index.html page.

```

{
  "name"         : "mean-hello-world",
  "version"      : "0.0.1",
  "description"  : "Hello World Example Application on the MEAN stack for Digital Garage",
  "main"         : "server.js",
  "author"       : "John McCawley",
  "dependencies" : {
    "body-parser"    : "^1.4.3",
    "express"        : "^4.13.4",
    "mongoose"       : "^4.4.12",
    "morgan"         : "^1.1.1"
  },
  "scripts": {
    "start": "node server.js"
  }
}

```

## Node.js Configuration

The `server.js` is the main file for our Node app and where we will configure the entire application.

In this file we will:

*   Configure our application
*   Connect to our database
*   Set the app to listen on a port so we can view it in our browser

```
// server.js

// set up ========================
var express = require('express');
var app = express(); // create our app w/ express
var fs = require('fs')
var morgan = require('morgan'); // log requests to the console (express4)
var bodyParser = require('body-parser'); // pull information from HTML POST (express4)
var mongoose = require('mongoose'); // mongoose for mongodb
var database = require('./config/database'); //load the database config

// configuration =================
app.use(express.static(__dirname + '/public')); // set the static files location /public/img will be /img for users
app.use(morgan('combined')); // log every request to the console
app.use(bodyParser.urlencoded({'extended': 'true'})); // parse application/x-www-form-urlencoded
app.use(bodyParser.json()); // parse application/json
app.use(bodyParser.json({type: 'application/vnd.api+json'})); // parse application/vnd.api+json as json

var port = process.env.PORT || process.env.OPENSHIFT_NODEJS_PORT || 8080,
    ip = process.env.IP || process.env.OPENSHIFT_NODEJS_IP || '0.0.0.0';

// connect to MongoDB database
mongoose.connect(database.url);
var db = mongoose.connection;
db.on('error', console.error.bind(console, 'connection error:'));

// listen (start app with node server.js) ======================================
app.listen(port, ip);
  console.log('Server running on http://%s:%s', ip, port);

module.exports = app;

```
This file looks very similar to the `server.js` file we created for the MEAN Stack. That makes sense because the front-end application will be in Vue.js but the back end services will still be NodeJS, Express and MongoDB. If want more details on setting up the back-end services, including MongoDB and Mongoose, take a look at my previous post on the MEAN Stack. The back-end is exactly the same.

## Vue.js Tutorial: The Tasklist Application

A quick note about application flow: a primary goal in this tutorial is to provide a basis of comparison between Vue.js and AngularJS in full-stack applications. Because of that, I have decided to keep the application flow exactly the same as the previous MEAN Stack tutorial. I will republish some of the steps that we took to set up the RESTful API and the database model in summary form. As with previous steps, if you want the details, the best resource is either my previous post or the Digital Garage documentation site.

### Creating Our RESTful API on Node.js

Before we get to the front-end application, we need to create our RESTful API. This will allow us to have an api that will **get all tasks**, **create a task**, and **complete and delete a task**. It will return all this information in JSON format.

#### Task Model

First we will define our database model for our tasks. This is going to be a simple model. Create a new folder in the app folder named models. In the models folder we will create a new file named `task.js`. In the task.js file we will define our model.

```
// app/routes.js

// get the task model
var Task = require('./models/tasks');

// make the routes available to our application with module.exports
module.exports = function(app) {

        // routes ======================================================================

        // api ---------------------------------------------------------------------
        // get the task list
        app.get('/api/tasks', function(req, res) {

            // mongoose has built-in functions that help us interact with MongoDB
            // use moongoose to get our tasks
            Task.find(function(err, tasks) {

                // if there is an error retrieving, send the error. nothing after res.send(err) will execute
                if (err)
                    res.send(err)

                res.json(tasks); // return all tasks in JSON format
            });
        });

        // create a task and get the task list after creation
        app.post('/api/tasks', function(req, res) {

            // create a task - again, mongoose helps us
            Task.create({
                text: req.body.text,
                done: false
            }, function(err, task) {
                if (err)
                    res.send(err);

                // you have seen this before
                Task.find(function(err, tasks) {
                    if (err)
                        res.send(err)
                    res.json(tasks);
                });
            });

        });

        // delete a task
        app.delete('/api/tasks/:task_id', function(req, res) {
        Task.remove({
            _id : req.params.task_id
        }, function(err, task) {
            if (err)
                res.send(err);

            // get the task list after deletion
            Task.find(function(err, tasks) {
                if (err)
                    res.send(err)
                res.json(tasks);
            });
        });
    });

    // application -------------------------------------------------------------
    // the default route for our application that serves the index.html
    app.get('*', function(req, res) {
        res.sendFile('./public/index.html', { root: __dirname });
    });

};

```
Inside of each of our API routes, we use the Mongoose functions to help us interact with our database. You remember that Mongoose helped us define our task data model earlier by declaring `var Task = mongoose.model` and now we can use that to **find**, **create**, and **remove**. There are many more things you can do with Mongoose. I would suggest looking at the official [docs](http://mongoosejs.com/docs/guide.html) to learn more.

Now that we’ve defined our routes, we update the `server.js` to `require` our routes file and pass our app variable to the function. This gives `routes` access to the functions of our app.

```
// server.js
...

    // load the routes
    require('./app/routes')(app);

...
```



## Front-end Application with Vue.js

Let's do a quick recap of what we have accomplished. So far, we have:

+ set up our stack in a Digital Garage development environment.
+ built our task model in MongoDB with the help of Mongoose
+ created a Node.js microservice with RESTful API's to get, create and remove tasks.

The application you have built so far could stand alone as a RESTful API serving up our task data to other applications. Now we'll create a front-end application in Vue.js that will use the RESTful API. This will all be presented in a single page web application.

### Setting Up Vue.js

Unlike AngularJS which is closer to the MVC (Model-View-Controller) pattern, Vue.js follows the MVVM or Model-View-View Model pattern. I am not going to get into the details of the debate between the two models in this tutorial, but rather, I will focus on how the two frameworks differ because they follow different patterns.

Let’s go through our Vue.js setup first. We will do the following:

+ declare a view model and create a Vue instance,
+ define functions to handle tasks in our view model,
+ use directives to bind our view to our view model.

In the `app.js` file we will create a module `helloTaskList` and a controller `MainCtrl`. Since we are starting with the `app.js` file from our "Hello World" example, the easiest thing to do is cut and paste the code below to replace the existing module and controller in `app.js`. However, you will get a better understanding of the syntax if you move through the file and replace the code line by line.

```
// public/app.js
var helloTaskList = angular.module('helloTaskList', [])

    .controller('MainCtrl', function($scope, $http) {
            $scope.formData = {};

            // when landing on the page, get all tasks and show them
            $http.get('/api/tasks')
                .then(
                    function(response) {
                        // success callback
                        $scope.tasks = response.data;
                        console.log(response.data);
                    },
                    function(error) {
                        // failure call back
                        console.log('Error: ' + error);
                    }
                );

            // when submitting the add form, send the text to the node API
            $scope.createTask = function() {
                $http.post('/api/tasks', $scope.formData)
                    .then(
                        function(response) {
                            $scope.formData = {}; // clear the form so our user is ready to enter another
                            $scope.tasks = response.data;
                            console.log(response.data);
                        },
                        function(error) {
                            console.log('Error: ' + error);
                        }
                    );
            };

            // delete a task after checking it
            $scope.deleteTask = function(id) {
                $http.delete('/api/tasks/' + id)
                    .then(
                        function(response) {
                            // success callback
                            $scope.tasks = response.data;
                            console.log(response.data);
                        },
                        function(error) {
                            // failure call back
                            console.log('Error: ' + error);
                        }
                    );
            };
        });
```

We also create our functions to:

+ get all tasks,
+ create a task,
+ and delete a task.

As you can see, all of our functions are using the RESTful API we just created. On page load, we will retrieve a task list through `GET /api/tasks` and bind the JSON we receive from the API to `$scope.tasks`. We will then loop over these in our view to make our tasks. We’ll follow a similar pattern for creating and deleting. (execute a function and refresh our task list.)

### Frontend View index.html

Here we will keep it simple. This is the HTML needed to interact with Angular. We will:

*   Assign Angular module and controller
*   Initialize the page by getting all tasks
*   Loop over the tasks
*   Have a form to create tasks
*   Delete tasks when they are checked

```
<!-- index.html -->
<!doctype html>

<html>

<head>
    <!-- META -->
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <!-- shameless self promotion -->
    <title>Digital Garage Quickstart Angular App</title>

    <!-- get bootstrap -->
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">

    <style>
        .jumbotron {
            background-color: #65bc45;
            /* Digital Garage Green */
            color: #ffffff;
            padding: 100px 25px;
        }
        .container-fluid {
            padding: 60px 50px;
        }
        .state-icon {
          left: -5px;
        }
        .list-group-item-primary {
          color: rgb(255, 255, 255);
          background-color: rgb(66, 139, 202);
        }
        /* DEMO ONLY - REMOVES UNWANTED MARGIN */
        .well .list-group {
          margin-bottom: 0px;
        }
    </style>

    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>

    <!-- get angularjs -->
    <script src="//ajax.googleapis.com/ajax/libs/angularjs/1.6.1/angular.min.js"></script>

    <!-- get our app -->
    <script src="app.js"></script>

</head>

<!-- add our angularjs module and controller -->

<body ng-app="helloTaskList" ng-controller="MainCtrl">

    <!-- page header and task count -->
    <div class="jumbotron text-center">
        <h1>You have {{ tasks.length }} items in your task list</h1>
    </div>
    <div class='container'>
        <!-- TASK LIST -->
        <div id="task-list" class="row">
            <div class="col-sm-8 col-sm-offset-3">
                  <div class="checkbox" ng-repeat="task in tasks">
                    <label>
                      <input type="checkbox" ng-click="deleteTask(task._id)">
                      {{ task.text }}
                    </label>
                  </div>
            </div>
        </div>

        <!-- FORM TO CREATE TASKS -->
        <div id="task-form" class="row">
            <div class="col-sm-8 col-sm-offset-2 text-center">
                <form>
                    <div class="form-group">

                        <!-- BIND THIS VALUE TO formData.text IN ANGULAR -->
                        <input type="text" class="form-control input-lg text-center"
                               placeholder="Pick up dry cleaning" ng-model="formData.text">
                    </div>

                    <!-- createTask() WILL CREATE NEW TASKS -->
                    <button type="submit" class="btn btn-primary btn-lg"
                                  style="background-color: #65bc45; border-color: green;"
                                  ng-click="createTask()">Add</button>
                </form>
            </div>
        </div>
    </div>
</body>

</html>

```

OK, let's commit our changes to our GitHub repository, and rebuild the application. After the build is complete, we should be able to see the following screen.

![MEAN stack task list application](http://assets-digitalgarage-infra.apps.thedigitalgarage.io/images/screenshots/tasklist_complete.png)

## Conclusion

Congratulations! You now we have a fully working application that will show, create, and delete tasks all via API (that we built!). That was quite a day. We’ve done so much. Just an overview of what we’ve accomplished:

*   RESTful Node.js API using Express
*   MongoDB interaction using mongoose
*   Single page application AngularJS application that does not require browser refreshes
*   built a set of microservices that are deployed in an extensible development environment
