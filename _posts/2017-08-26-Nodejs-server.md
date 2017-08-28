---
layout: post
title: Node.js + Express Generator
tags: [nodejs,express,sequelize]
---

At work, I don't start brand new projects very often. At most of the jobs I have had, either the projects have already been established and tasks involve adding new features to it, or code bases are inherited from other companies/groups. Usually looking at someone else's project for the first time is accompanied with phrases such as "Why would anyone do it this way?" or "If I was building this type of application I would have done it in [x] language and with [y] framework."

Side projects on the other hand usually only consist of starting from scratch. This is both exhilarating and exhausting. At least for me, the process usually goes as follows: Read about a new language/framework, decide to build something using that technology, then spend most of my Saturday setting up my developer environment, and finally barely get through a hello world app before my free time runs out. I'm sure that I'm not the only person with this workflow, and it shows by the number of new languages/frameworks that have some sort of generator/scaffolding tool that greatly speeds up the process.

Recently I was trying to build a web app using React and needed a server that would serve the API and would also allow uploading files. I decided to use a Node.js server, but instantly had flash backs about the last time I attempted to use [Express](https://expressjs.com/) from scratch and how I had spent most of my time trying to figure out which packages to include and which order to add them to my app.js file so that the server would start up and do what I needed. This time however, it went a lot smoother. Now, Express has an [express-generotor](https://expressjs.com/en/starter/generator.html) which scaffolds out the project for you and includes all of the needed dependencies to get you off the ground and running in no time. 

Let's give it a try.

```
$npm install express-generator -g
$express myapp
$cd myapp
$npm i
$npm start
```
Done.
The project is built, installed, and running. If you go to the browser and navigate to localhost:3000, you will be greeted by a very basic page that says "Express" and a warm feeling in your stomach because you don't have to spend the next 2 hours dealing with missing/mismatched dependencies.

Now that you have the basics of the project built for you, it is time to decide how you want your project structured. You can always change you mind later, but it is a lot easier to decide now when you have only a few classes and components, than later when you have way more and way less time to change things around.

After doing a lot of searching, blog reading, blog skimming, and sifting through StackOverflow answers, I found that I really like [this](https://stackoverflow.com/a/19623507/947240) structure which is demonstrated in [this Github repo](https://github.com/focusaurus/express_code_structure). There is a lot going on there. I would recommend you taking a few minutes to read through it all. 

The main idea that I am looking for is where to put the router and model files for each component. I have worked on projects that have separated them into their own folders with all modals in one folder and all routers in another, but I like it better when everything is together in a single folder. It makes it easy to look at it and see what each component has, and what it is related to.

If you look at the project that the express-generator generated for us, you will notice that it has created a router folder. Instead of using that, create a new folder under the root directory named app, and then create another folder under that named users. This is where all of the user files will live. Feel free to create a folder for any other components that you will need. The component names should match what each endpoint your API will be serving; users, customers, vehicles, unicorns, etc. Once you have created the necessary folders, move the routes/users.js file into the new users folder and rename it to router.js. 

Currently, the app is serving [jade](http://jade-lang.com/) files to the browser. For now, I am only going to be focusing on using this application as for API endpoints. Go ahead and delete the routes folder since we won't be needing that either.

The app.js file is the main entry point for all the incoming requests. If you handled all of the routes here, the file would get out of hand faster than you can say, "spaghetti junction." To remedy this, we will only reference each components route file in the app.js. 

In app.js, change:

```javascript
var users = require('./routes/users');
```
to:

```javascript
var users = require('./users/router');
```

You will notice in app.js that there is already an entry to route the requests of /users to the correct users route.

```javascript
app.use('/users', users);
```

To see the changes, you will need to stop the node server and then restart it. You will need to do that after every change which will get very annoying very quickly. Instead, you can use something like [nodemon](https://github.com/remy/nodemon), [forever](https://github.com/foreverjs/forever), or [pm2](http://pm2.keymetrics.io/) to watch the project for changes and restart the server so that you can focus on making code changes and not getting angry when your changes aren't showing up because you forgot to restart the server. If you look in our package.json under scripts, you will notice that start is mapped to "node ./bin/www" so if you are going to use nodemon, you will need to start it with:

```
nodemon ./bin/www
```

After the server is back up and running, you should be able to curl localhost:3000/users and get less than stellar response. When was the last time you wanted an API endpoint to return static text? Instead, you usually will want to retrieve records from a database. There are a slew of database drivers that you can use with node. Today, I've chosen to use [Sequalizejs](http://docs.sequelizejs.com/). It is fairly easy to use, it has plenty of documentation to help you along your way, and it works with a few different databases. Currently I have MySQL running locally, so I am going to use that. To install:

```
npm install --save sequelize
npm install --save mysql2
```

Create a file named users-model.js in app/users. In this file, we are going to create the database connection, define the model, and add a function to save and get a list of users from the database. 

Add the following to users-model.js:

```javascript
const Sequelize = require('sequelize');
const sequelize = new Sequelize('mytestapp', 'app_user', 'supersecretpassword', {host: 'localhost', dialect: 'mysql', logging: true});

const Users = sequelize.define('users', {
    firstName: Sequelize.STRING,
    lastName: Sequelize.STRING
});

sequelize.authenticate()
    .then(() => {
    console.log('Connection has been establised to database');
Users.sync();
})
.catch(err => {
    console.error('Error connecting to database', err);
});

function upsert(user) {
    return Users
        .findOne({where: {firstName: user.firstName, lastName: user.lastName}})
        .then(function (obj) {
            if (obj) {
                obj.changed('updatedAt', true);
                user.updatedAt = new Date();
                return obj.update(user);
            }
            else {
                return Users.create(user);
            }
        });
}

function findAll(options, callback) {
    Users
        .findAll(options)
        .then(users=> {
        callback(null, users);
});
}

exports.upsert = upsert;
exports.findAll = findAll;
```

Let's walk through what is going on here. First we are create an instance of sequelize and give it the credentials to connect to our database. As your app grows and you have more components, you will want to move this out of the user specific model and into an app wide class that each model can access. For now though, this is fine. Next we define the Users model. By default, when the app starts up, it will connect to the database, and sequelize will check to see if there is a Users table with the columns firstName and lastName, if there isn't it will create on for us. This is nice because you don't have to worry about the code being out of sync, but you will need to put more thought into it later after you have been inserting data into the table for months and then you need to add/alter a column.

Next we have two functions, upsert(user) and findAll(options, callback). Upsert is used to either insert a new record or update an existing record if one already exists. This helps prevent duplicate records from getting created.

The findAll function takes in search parameters to give to sequalize to built the query. To get a full list of the options, read over the [sequalize docs](http://docs.sequelizejs.com/manual/tutorial/models-usage.html#data-retrieval-finders).

Back in app/users/router.js, remove the useless default '/' function and replace it with the following, more meaningful code:

```javascript
function create(req, res){
  try {
    let user = {
      firstName: req.body.firstName,
      lastName: req.body.lastName
    };
    users.upsert(user);
    res.status(201).send('');
  }
  catch (ex) {
    res.status(500).send('Failed create user');
  }
}

function getAll(req, res) {
  try {
    users.findAll({}, function (err, users) {
      res.json(users);
    });
  }
  catch (ex) {
    res.status(500).send('Failed to get users');
  }
}

router.post('/', create);
router.get('/', getAll);
```

These functions map the base routes to either create or getAll depending on the request method. Once all of the code is in place and saved, give it a try.

Insert a couple of users:

```
$curl -X POST -d '{"firstName":"dave", "lastName": "mcnavish"}' -H "Content-Type: application/json" localhost:3000/users
$curl -X POST -d '{"firstName":"bruce", "lastName": "wayne"}' -H "Content-Type: application/json" localhost:3000/users
```

Then retrieve the users:

```
$curl localhost:3000/users
```

There you have it. Using the express generator, we quickly created a node.js project that serves our Users API. It isn't too magical now, but we have laid the ground work for future development. We learned how to route requests, structure a project, connect to a database, and save and retrieve records from that same database. Pat yourself on the back, you have a working API.



