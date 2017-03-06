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

After signing into your Digital Garage account, deploy the Vue.js Full-stack Quickstart (qs-mven). There's a detailed walkthrough of setting up an application throught the web console and command line interface on the Digital Garage documentation site.Choose the Add to Project link in the top menu bar to go to the template catalog.

![Add To Project](http://assets-digitalgarage-infra.apps.thedigitalgarage.io/images/screenshots/add_to_project.png)

In the add to project screen, choose the Vue.js Full-stack Quickstart (qs-mven) from the catalog.

![Add To Project](http://assets-digitalgarage-infra.apps.thedigitalgarage.io/images/screenshots/choose_quickstart.png)

In the template configuration page change the Git Repository URL to point to the repository that was just forked into your account. `https://github.com/johnmccawley/vuejs-ex-ex.git` You can simply accept the defaults for the remaining parameters and click "Create"

![Add To Project](http://assets-digitalgarage-infra.apps.thedigitalgarage.io/images/screenshots/quickstart-configure.png)

That's it. Digital Garage is now setting up your Vue.js application, Mongo database, and NodeJS preconfigured with Express. On the next screen click "Continue to Overview". You will be taken to the Project Overview screen where you can watch Digital Garage do the setup work for you. In just a few minutes you'll have full stack running in containers and managed through Google Kubernetes. When MongoDB and NodeJS are completely deployed, (the pod status circle is Green) simply click on the application URL in the upper right corner of the overview screen. You will be taken to a browser to see a simple "Hello World" message.

Before you actually start the tutorial, you may want to become more familiar with the features of Digital Garage that make your life as a developer easier.
Features like:

+ [A command line interface (CLI)](http://docs.thedigitalgarage.io/getting_started/developers_cli.html)
+ [Support for Github webhooks](http://docs.thedigitalgarage.io/getting_started/basic_walkthrough.html#bw-configuring-automated-builds)


## File Structure

Now that we have the MEAN Example repository forked into your account, let's take a few minutes to review the file structure for the repository. There are many ways to structure a MEAN application. I have tried to take the best-practices from several tutorials and create a simple yet expandable file structure this example project. For further reading on file structures for the MEAN Stack, [Mean.io](http://mean.io) is a good boilerplate to see best practices and how to separate file structure. For now, though, we will just use the following structure adjust as we go.

```

    - app                       <!-- holds all our files for node components (models, routes) -->
    ----- models
    ---------- any-model.js     <!-- defines the mongoose/mongodb model -->
    ----- routes.js             <!-- all routes will be handled here -->

    - config                    <!-- all our configuration will be here -->
    ----- database.js

    - public                    <!-- holds all our files for our frontend angular application -->
    ----- app.js                <!-- all angular code for our app -->
    ----- index.html            <!-- main view -->

    - package.json              <!-- npm configuration to install dependencies/modules -->
    - server.js                 <!-- Node configuration -->

```

### Installing Modules

In Node, the `package.json` file holds the configuration for our app. Node’s package manager (npm) will use this to install any dependencies or modules that we are going to use. In our case, the Digital Garage Quickstart template read the package.json file and installed all of the packages we need. As with a standard npm install, you can change the `package.json` to include or exclude modules. For this example, we will, of course, be using [Express](http://expressjs.com/) (the "E" in MEAN) and [Mongoose](http://mongoosejs.com/) (a popular object modeling module for MongoDB).

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

### Node.js Configuration

Our `package.json` file sets the "start" script for node to `server.js`. This is the main file for our Node app and where we will configure the entire application.

This is the file where we will:

*   Configure our application
*   Connect to our database
*   Set the app to listen on a port so we can view it in our browser

You may also notice that we are collecting some environment variables from our Digital Garage environment, `process.env.PORT`. This just makes our configuration easier and more portable. We will discuss these features in more detail in later tutorials. For now, it is only important to understand that environment varialbles are helping us configure the app for Express, our MongoDB database, and listening on a port.

```
// server.js

// set up ========================
var express = require('express');
var app = express(); // create our app w/ express
var fs = require('fs')
var mongoose = require('mongoose'); // mongoose for mongodb
var morgan = require('morgan'); // log requests to the console (express4)
var bodyParser = require('body-parser'); // pull information from HTML POST (express4)
var database = require('./config/database'); //load the database config

// configuration =================
app.use(express.static(__dirname + '/public')); // set the static files location /public/img will be /img for users
app.use(morgan('combined')); // log every request to the console
app.use(bodyParser.urlencoded({'extended': 'true'})); // parse application/x-www-form-urlencoded
app.use(bodyParser.json()); // parse application/json
app.use(bodyParser.json({type: 'application/vnd.api+json'})); // parse application/vnd.api+json as json

var port = process.env.PORT || process.env.OPENSHIFT_NODEJS_PORT || 8080,
    ip = process.env.IP || process.env.OPENSHIFT_NODEJS_IP || '0.0.0.0';

mongoose.connect(database.url);// connect to mongoDB database
var db = mongoose.connection;
db.on('error', console.error.bind(console, 'connection error:'));

// listen (start app with node server.js) ======================================
app.listen(port, ip);
console.log('Server running on http://%s:%s', ip, port);

module.exports = app;

```

Just with that bit of code, we now have an HTTP server courtesy of Node. We have also created an app with Express and now have access to many benefits of it. In our `app.configure` section, we are using express modules to add more functionality to our application.

### Database Setup

We will be using a local database that the Digital Garage Quickstart template has deployed for us. Digital Garage has automatically deployed a MongoDB in a Docker Container and provided us with a network address to connect to the database. The database URL is configured in `config/database.js` and used in the `mongoose.connect` command to connect to it. In the current "Hello World" example, the database connection is not being used. We will use that connection later in the tutorial. For now, as long as logs for the NodeJS application do not show you a database connection error, you are connected to the database.

## Building our Tasklist Application

### Application Flow

There are a lot of different ideas and technologies involved in this simple application. It is easy to get confused during the setup. If you just take a few minutes to understand the flow and the components of this MEAN Stack tutorial, it will be much easier.

You are going to be implementing this tutorial through a design or software architecture best practice known as a microservices architecture. There is a lot of information about the definition, benefits and pitfalls of microservices available on the web. For now, however, it is only important to know that Digital Garage is handling all of the difficult details of Microservices in the background. You simply need to configure your application to take advantage of those best practices. In the next few paragraphs, I will explain a bit of the separation of tasks and how the services tie together.

AngularJS is the framework that will handle the front-end work. It will access all the data it needs through a the Node.js RESTful API. The AngularJS application then, is considered a microservice. It is self contained and interacts with other microservices.

In this tutorial we will be implementing a small Node.js application (or microservice) that retrieves data from the database and returns that data in JSON format to our AngularJS application via a RESTful API. In this way, you can separate the frontend application from the actual API. If you want to extend the API, you can do so without affecting the front-end application. You can eventually build different microservices (or apps) on different platforms, or even different languages that simply connect to this REST-based API.

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



## Front-end Application with Angular

Let's do a quick recap of what we have accomplished. So far, we have:

+ set up our MEAN Stack
+ built our task model in MongoDB with the help of Mongoose
+ created a Node.js microservice with RESTful API's to get, create and remove tasks.

The application you have built so far could stand alone as a RESTful API serving up our task data to other applications. We're not going to stop there, though. We're going to create a frontend application in AngularJS that will use the RESTful API that we just built. The AngularJS appliction will use the **GET** endpoint to retrieve the task list, the **POST** endpoint to create a task and then retrieve the new task list and the **DELETE** endpoint to remove a task from the task list and retrieve the new task list. This will all be presented in our single page web application.

To get started we will use the `app.js` file and the `index.html` file that we used for our "Hello World" example application. With a few simple modifications we will have our new front-end application.

### Setting Up AngularJS app.js

AngularJS is what we refer to as a Model-View-Controller (MVC) framework. MVC frameworks have been around for a long time. Frameworks that use the MVC pattern are very popular for front-end web applications because they treat the data (the model), the screenflow (the controller) and the presentation of the data (the view) as seperate "concerns". In other words, it makes an application easier to maintain if you keep all of the code that moves you from one screen to the next in a file called a controller, all of the data in a model and all of the presentation code in the view. Again, you do not need to know the details of the Model-View-Controller pattern to write AngularJS applications, but if you are interested in learning more, I suggest reading this [tutorial](http://www.angularjstutorial.com/2014/02/12/angularjs-basics-part-1-the-model-view-controller-approach/)

Let’s go through our Angular setup first. We will do the following:

+ create a module,
+ create a controller,
+ and define functions to handle tasks,
+ apply to view.

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
