<!--
Creator: Alex White
Market: SF
Adapted By: Zeb Girouard
Market: DEN
-->

![](https://ga-dash.s3.amazonaws.com/production/assets/logo-9f88ae6c9c3871690e33280fcf557f33.png)

<!--10:30 5 minutes -->

<!-- Hook: (Show hands) How many of you know about SQL?  How many of you love SQL?  How many of you have had a SQL syntax problem that drove you crazy?  This is the reason that so many frameworks and modelers have been developed.  

(Show hands) Who feels Mongo is better than plain Javascript?

-->

#Intro Mongoose

## Learning Objectives

By the end of this lesson you should be able to...

* **Compare** and **contrast** Mongoose with Mongo
* **Create** Mongoose schemas & models
* **Integrate** Mongoose with Express
* **Build out** `#index`, `#new`, and `#create` routes with Mongoose & Express

## Why

Mongoose is an ODM, an **Object Document Mapper**. It *maps* documents in a database to JavaScript Objects, making modifying the database possible with easy to use JavaScript helper methods.

>"Let's face it, writing MongoDB validation, casting and business logic boilerplate is a drag. That's why we wrote Mongoose."

>—Creators of Mongoose.

<!--10:35 10 minutes -->

## Terminology
- **Schema**: Similar to an object constructor, a Schema is a diagram or blueprint for what every object in the noSQL database will contain.  Here's an example of a simple Address Book noSQL database schema:  

```javascript
  var ContactSchema = new Schema({
    firstName: String,
    lastName: String,
    address: String
    phoneNumber: Number,
    email: String
    professionalContact: Boolean
  });
```

*With the above Schema, we can expect all of our Address Book entries would have a first name, last name, address, and email address in the form of Strings.  We can count on the phoneNumber to always be accepted, stored, and returned as a number.  Lastly, the boolean value of Professional Contact will always be a `true` or `false`*

- **Model**: A model is a Schema that has been 'activated' with real data and is performing actions such as reading, saving, updating, etc.

```javascript
  var Contact = mongoose.model('Contact', ContactSchema);
```

####Schema vs. Model

>"In mongoose, a schema represents the structure of a particular document, either completely or just a portion of the document. It's a way to express expected properties and values as well as constraints and indexes. A model defines a programming interface for interacting with the database (read, insert, update, etc). So a schema answers "what will the data in this collection look like?" and a model provides functionality like "Are there any records matching this query?" or "Add a new document to the collection". ""

<!-- Analogy on the board: schema -> skeleton, model -> body -->

<!--CFU: Catch-phrase with Schema, Model, Mongo, and Mongoose in pairs -->

-[Peter Lyons Apr 8 '14 at 23:53](http://stackoverflow.com/a/22950402)

<!--10:45 15 minutes -->
## Example (I do)

From the console:  

```
mkdir mongoose_project
cd mongoose_project
npm init
npm install --save mongoose
```

We need to make sure MongoDB is running.  From the console, enter this command:

```
mongod
```

Create a new Javascript file by typing `touch index.js` into the terminal.

Let's require mongoose and connect to our database.

```javascript
var mongoose = require("mongoose");
mongoose.connect("mongodb://localhost/test");
```

###Modeling

Let's create `Book` model. A `Book` has a few different characteristics: `title`, `author`, and `description`.

To create a `Book` model we have to use a Schema:

```javascript
var Schema = mongoose.Schema;
var BookSchema = new Schema({
    title: String,
    author: String,
    description: String
});
```
and finally create the model

```javascript
var Book = mongoose.model('Book', BookSchema);
```

Check the docs to see all the different [datatypes](http://mongoosejs.com/docs/schematypes.html) we can use in a Schema.

###Create: Building and Creating Documents

If you want to build a new `Book` in memory you can just do the following:

```javascript
var book = new Book({title: "Alice's Adventures In Wonderland"});
```

Then you can still edit it before saving.

```javascript
book.author = "Lewis Carroll";
```

Once you're done building you can save the book.

```javascript
book.save()
```
>Note: you can pass save a function that will be called (aka a callback) once the `book` is done being saved.

If you want to build & save in one step you can use `.create`. Also we'll pass it a callback to execute once it's done creating the book.

```javascript
Book.create({title: "The Giver"}, function (err, book) {
  console.log(book);
});
```

###Read

We can find books by any field, including `author`:

```javascript
  Book.find({author: "Lewis Carroll"}, function (err, books) {
    console.log(books);
  });
```

We can find ALL by specifying any `fields` we're filtering for, aka an empty object:

```javascript
Book.find({}, function(err, books){
  console.log(books);
});
```

Try out some of the other find methods.

```javascript
.findOne();
.findById();
```
Reference the [docs](http://mongoosejs.com/docs/guide.html) for more info on what you can do with Mongoose queries and models (use the left-hand side-bar).

###Destroy
Removing a Document is as simple as Building and Creating.

Using the remove method:

```js
Book.remove({ title: "The Giver" }, function(err, book) {
  if (err) { return console.log(err) };
  console.log("removal of " + book.title + " successful.");
});
```
Other removal methods include:

```js
findByIdAndRemove();
findOneAndRemove();
```

<!--11:00 15 minutes -->

## Integrating Mongoose into Express

Well, that's nice. But let's see how mongoose plays with express by building a reminders app. called [reminders](https://github.com/ga-dc/reminders_mongo)

Lets create a brand new express application and grab up all the dependencies we'll need

```bash
$ mkdir reminders
$ cd reminders
$ npm init
$ npm install --save express ejs body-parser mongoose
$ touch index.js
```

The 5 dependencies we'll be using for this app are:

  1. `express` - web Frameworks
  1. `ejs` - view engine
  1. `body-parser` - allows us to get parameter values from forms
  1. `mongoose` - our Mongo ODM

Let's first start by defining our schema, models and creating some seed data.

Folders/files:

```bash
$ mkdir db
$ mkdir models
$ touch models/reminder.js
$ touch db/seed.js
```

We will define the structure of our database using schemas

In `models/reminder.js`:

```js
// requiring mongoose dependency
var mongoose = require('mongoose');

// defining schema for reminders
var ReminderSchema = new mongoose.Schema({
  title: String,
  body: String,
  createdAt: { type : Date, default: new Date() }
});
// define the model
var Reminder = mongoose.model("Reminder", ReminderSchema);
// export the model to any files that `require` this one
module.exports = Reminder;
```

Great now that we have an interface for our models, let's create a seed file so we have some data to work with in our application.

In `db/seed.js`:

```js
var mongoose = require('mongoose');
var conn = mongoose.connect('mongodb://localhost/reminders');
var Reminder = require("../models/reminder");
Reminder.remove({}, function(err) {
  if (err) {
    console.log("ERROR:", err);
  }
})

var reminders = [
  {
    title: "Cat",
    body: "Figure out his halloween costume for next year"
  },
  {
    title: "Laundry",
    body: "Color-code socks"
  },
  {
    title: "Spanish",
    body: "Learn to count to ten to impress the ladies"
  }
];

Reminder.create(reminders, function(err, docs) {
  if (err) {
    console.log("ERROR:", err);
  } else {
    console.log("Created:", docs)
    mongoose.connection.close();
  }
});
```

Now run the seed file in order to add these default values to our Database, by typing ```node db/seed.js``` in the terminal.

<!-- 11:15 10 minutes -->

## Server Setup

Now that we've got all of our models and seed data set. Let's start building out the reminders application. Let's update our main application file to include the dependencies we'll need.

In `index.js`:

```js
// Dependencies
var express = require('express');
var app = express();
var mongoose = require('mongoose');
var bodyParser = require('body-parser');

// Configuration
mongoose.connect('mongodb://localhost/reminders');
process.on('exit', function() { mongoose.disconnect() }); // Shutdown Mongoose correctly
app.set("view engine", "ejs");  // sets view engine to handlebars
app.use(bodyParser.json());  // allows for parameters in JSON and html
app.use(bodyParser.urlencoded({extended:true}));
app.use(express.static(__dirname + '/public'));  // looks for assets like stylesheets in a `public` folder
var port = 3000;  // define a port to listen on

// Controllers
var remindersController = require("./controllers/remindersController");

// Routes
app.get("/reminders", remindersController.index);

// Start server
app.listen(port, function() {
  console.log("app is running on port:", port);
});
```

<!--11:25 10 minutes -->

## Reminders Index

As we can see on the last line of the code above, we have just one route. It's using `remindersController.index ` as a callback, but it hasn't been defined yet. Let's define it now.

```bash
$ mkdir controllers
$ touch controllers/remindersController.js
```

```js
var Reminder = require("../models/reminder")

var remindersController = {
  index: function(req, res) {
    Reminder.find({}, function(err, docs) {
      res.render("reminders/index", {reminders: docs});
    });
  }
}

module.exports = remindersController;
```

We're rendering a ejs view that doesn't exist yet. Lets create it now as well as our layout view.

```bash
$ mkdir views
$ touch views/layout.ejs
$ mkdir views/reminders
$ touch views/reminders/index.ejs
```

In `views/layout.ejs`:

```html
<!doctype html>
<html>
  <head>
    <link rel="stylesheet" type="text/css" href="/css/styles.css">
  </head>
  <body>
    <header>
      <h1><a href="/reminders">Reminder.ly</a></h1>
      <a href="/reminders/new">+ Reminder</a>
    </header>
    <hr>
    <!-- the below `triple-stash`, avoid handlebars escaping any html -->
   <%- body %>
  </body>
</html>
```

In `views/reminders/index.ejs`:

```html
<ul>
  <% for(var i=0; i< reminders.length; i++) {%>
    <li class="reminder">
      <a class="reminder-title" href="/reminders/{{_id}}"><%= reminders[i].title %>:</a>
      <span class="reminder-body"><%= reminders[i].body %></span>
    </li>
  <% } %>
</ul>
```

<!--11:35 10 minutes -->

## New Reminder

We'll need to update our routes in `index.js` to add `#new` and `#create` actions.

```js
app.get("/reminders", remindersController.index);
app.get("/reminders/new", remindersController.new);
app.post("/reminders", remindersController.create);
```

Our views/reminders/new.ejs should look something like this:

```html
<h2>Create a New Reminder</h2>
<form action="/reminders" method="post">
  <label for="reminder-title">Title:</label>
  <input id="reminder-title" type="text" name="title">
  <label for="reminder-body">Body:</label>
  <input id="reminder-body" type="text" name="body">
  <input type="submit">
</form>
```

Finally our updated `remindersController` will be:

```js
var Reminder = require("../models/reminder")

var remindersController = {
  index: function(req, res) {
    Reminder.find({}, function(err, docs) {
      res.render("reminders/index", {reminders: docs});
    });
  },
  new: function(req, res) {
    res.render("reminders/new")
  },
  create: function(req, res) {
    // strong params
    var title = req.body.title;
    var body = req.body.body;
    Reminder.create({title: title, body: body}, function(err, doc) {
      // if there there is an error: redirect to reminders#new; else: redirect to reminders#index
      err ? res.redirect("/reminders/new") : res.redirect("/reminders");
    })
  }
}

module.exports = remindersController;
```

![congrats](images/elfCongrats.gif)

<!-- 11:45 5 minutes -->

## Show Page (You do)

* User should be able to click on any reminder's title on the `reminders` page and be directed to the specific `reminder` page that displays more specific information about that reminder.

<!--CFU: Think-pair-share: What is the most helpful thing about Mongoose in your opinion? -->

<!-- Closing

Mongoose is an extension of the logic that made Node and Express grow in popularity: if we can write it all in Javascript, we have no major miscommunication issues in the stack.  For tomorrow, think about a specific thing you'd like to do in pure Javascript to incorporate into your app.  Chances are, there's already a library for it.  If not, you'll have a really fun project idea. -->

## Bonus

#### CRUDing in the Console
Checkout the `solution-code` directory and examine the file `db/console.js`. It drops you into a REPL session with the database and all your models pre-loaded. You can run `node db/console.js` to see for yourself. CRUD some data in this REPL.
>Note: If for some reason it's not working, try restarting your `mongod` process.

#### Refactor
* How would you refactor your `index.js` so that all the configuration logic lived in a file `config/config.js`?
* How about refactoring the code so that your routes live in `config/routes.js`?
