# ADDING BASIC AUTHENTICATION TO YOUR CRUD APP
__Uses__: JavaScript (Express.js, Node.js), MongoDB

__Node.js packages__: express, ejs, mongoose, method-override, dotenv, bcrypt, express-session

__Prereq__: Have Your Basic CRUD App set up and npm initiated

### Install new node.js packages

__Install in Terminal: bycrypt, dotenv, and express-session__

[express-session](https://www.npmjs.com/package/express-session) is a module that creates cookies in the browser in an encrypted way in order to create user "sessions" after log-in. It allows us to store environmental variables specific to our computer. [Dotenv](https://www.npmjs.com/package/dotenv) is a module that loads environment variables from .env into a process.env property. The [process.env](https://nodejs.org/docs/latest/api/process.html#process_process_env) property returns an object containing the user environment. [Bcrypt](https://www.npmjs.com/package/bcrypt) is a library that hashes passwords for you, letting you safely store log in information in a database. All these packages relate to safely storing user information and creating/maintaining user sessions on your site. 

```
npm install dotenv
npm install bycrypt
npm install express-session
```
### Make Your Environment Configuration Variables Better with .env

Your .env file will store your environment configuration variables in a way that is separated from the actual code. This follows the practices outlined in [The Twelve-Factor App](https://12factor.net/config), which states there should be "strict separation of config from code. Config varies substantially across deploys, code does not." [This method also keeps your private information, like api keys and passwords, safe.](https://medium.com/codait/environment-variables-or-keeping-your-secrets-secret-in-a-node-js-app-99019dfff716)

In your .env file, list variables PORT, MONGODB_URI, and SECRET.

```
PORT = 3000
MONGODB_URI=mongodb://localhost:27017/<databasename>
SECRET=<yoursecrethere>
```
In your server.js file, authorize the new packages that you installed and switch out your static environmental variables with those in your .env file.

```
//New dependencies to include
const sessions = require('express-session');
require('dotenv').config();

//New configuration to include 
const PORT = process.env.PORT;
const mongodbURI = process.env.MONGODBURI;

//New Middleware to include
app.use(session({
  secret: process.env.SECRET,
  resave: false,
  saveUninitialized: false
}));

//Replace static environment variables with .env variable in your mongoose.connect 
mongoose.connect(
  mongodbURI, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
    useFindAndModify: false,
    useCreateIndex: true,
  },
  () => {
    console.log('the connection with mongod is established at', mongodbURI)
  }
);

//replace static environment variables with .env variable in your app.listen
app.listen(PORT, ()=> {
  console.log('Listening on port', PORT);
});
```

### Add Log In and Sign Up Buttons to Your View Pages
Create a ejs partial that will act as each ejs file's nav bar. This will include links that let the user log in or sign up. Can change this UI element depending on your web design. 

If you don't have one already, make a partials directory in your views directory. Create a nav.ejs file for this directory.

* Views
  * Partials
    * nav.ejs

Your nav should have at least two links: one to the Sign Up page and one to the Log In page. Nothing will happen when you click on them just now, but you'll be able to test them once you get each feature set up. 

```
<li><a href="/users/new">Sign Up</a></li>
<li><a href="/sessions/new">Log In</a></li>

```
Now, you can include this nav bar on any of your ejs files
```
<%- include('../partials/nav.ejs')>
```

### Create Users and Sessions in the MVC Structure

You will add users to your app by adding new files to your controllers, models, and views directories .You will add sessions to your app by adding new files to your controllers and views directories. 

This means changing how your directories are organized. 

In your controllers directory, you'll be adding two new files for users and sessions. These files will control different RESTful routes.

* Controllers
  * originalCollectionName_controller.js
  * sessions_controller.js
  * users_controller.js

You don't currently need a collection for session data, because the session will go away when you end it. You do need to add a collection for user data.

* Models
  * originalCollectionName.js
  * users.js

Finally, all the __views__ for your current CRUD app are floating around in the views directory. Put them in one folder, then create a folder for sessions and users with a 'new.ejs' file

* Views
  * originalCollectionDirectoy
    * index.ejs
    * show.ejs
    * edit.ejs
    * new.ejs
  * sessions 
    * new.ejs
  * users
    * new.ejs

### Set up your site for users to sign up

For users to sign up on your app, you need to flesh out the controller, model, and view files for user.

__The users model__ holds login information on each user. Set up your schema, connect to mongoose, and export that user data to the controller in the model file - users.js
```
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const userSchema = new Schema({
  username: {
    type: String, 
    unique: true,
    required: true
  },
  password: {
    type: String, 
    required: true
  }, 
  name: {
    type: String, 
    required: true
  }
});

const User = mongoose.model('User', userSchema);

module.exports = User;
```
Because you have a new model, you need a new controller file to interact with and encrypt the data in that model. This will be your users_controller.js file. 

__Set up users_controller file:__

```
//Dependencies
const bcrypt = require('bcrypt');
const express = require('express');
const users = express.Router();
const User = require('../models/users.js');

//Routes -- main routes will be new and create
//NEW
users.get('/new', (req, res) => {
  res.render('users/new.ejs');
});

//CREATE
users.post('/', (req, res) => {
  req.body.password = bcrypt.hashSync(req.body.password, bcrypt.genSaltSync(10))
  User.create(req.body, (err, createdUser) => {
    res.redirect('/');
  })
});

//export User 
module.exports = User;
```

The Create route above not only adds data from a form into a database, it encrypts it using the bcrypt library we installed. The create route in the example above pulls the 'password' input from the form and uses the method [_bcrypt.hashSync()_](https://www.abeautifulsite.net/hashing-passwords-with-nodejs-and-bcrypt). This hashes the password using a synchronous approach, meaning that it stops the app from handling any other requests until its finished hashing. 

__Set up new.ejs file within view/users:__

This new.ejs file will be a form with the same action/post set up as the new.ejs file in your original new.ejs file. This should render as a sign up page for users to enter their username, password, etc.

```
<form action="/users" method="POST">
<form>
```

### Set up site for users to log in: 

For users to log in to your app, you need to flesh out the controller and view files for sessions. You don't currently need a model because you're not storing session data yet. Every time a user logs in, it will create a session for them. When a user logs out, it will delete their session. 

__Set up sessions_controller.js file:__

```
//Dependencies
const bcrypt = require('bcrypt');
const express = require('express');
const sessions = express.Router();
//we are using the users.js model within this controller file
const User = require('../models/users.js');

//routes - New and Delete

//NEW
//renders log in page
sessions.get('/new', (req, res) => {
  res.render('sessions/new.ejs', {
    currentUser: req.session.currentUser
  }
});

//CREATE
//checks that password and username match document in user database
sessions.post('/', (req, res) => {
  User.findOne({username: req.body.username}, (err, foundUser) => {
    if (err) {
      console.log(err)
      //fix to render error page
      res.send('Our database ran into a problem, Our fault');
    //still testing if this works - should compare username and password
    } else if(!foundUser || !bcrypt.compareSync(req.body.password, foundUser.password)) {
      //fix to render error page
      res.send('Sorry, your username or password is incorrect);
    } else if(bcrypt.compareSync(req.body.password, foundUser.password)) {
      res.session.currentUser = foundUser;
      res.redirect('/originalCollectionName');
    } else {
      //delete if works
      res.send('combining the user and password authentication did not work, refactor')
    }
  })
});

//DELETE
//triggered on log out
sessions.delete('/', (req, res) => {
  req.sessions.destroy(() => {
    res.redirect('/')
  })
});

//export it!
module.exports = sessions;
```
__Set up new.ejs in views/sessions:__
The new.ejs for sessions should render the log in page. This form is the same as the other new.ejs forms. 
```
<form action="/sessions" method=POST>
</form>
```

### Make Things Change When Your User Logs In
Now, a user can log in and... nothing changes. That defeats the purpose of a log in feature, so the next step is changing our app depending on whether a user has started a session or not. 

We can pass on the information that a user signed in or logged out to our other files because, in our sessions_controller.js file, we set up a cookie with the key/value of the currentUser set to foundUser upon successful log in. 
```
//snippet of sessions post route
if (bcrypt.compareSync(req.body.password, foundUser.password)) {
  //set up cookie with key/value of currentUser set to foundUser
  req.session.currentUser = foundUser;
  res.redirect('/originalCollectionName')
```

__Pass currentUser onto the other routes:__

```
//value to pass in
currentUser: req.session.currentUser;

//example of how to pass value in
data.get('/', (req, res) => {
  Data.find({}, (err, allData) => {
    res.render('data/index.ejs', {
      data: allData,
      currentUser: req.session.currentUser
  })
  })
});
```

Now, we can use this info to give the user visual feedback that they have logged in successfully. Currently, we have a nav bar that just displays "Sign Up" and "Log In" all the time. You can add an if statement in your nav.ejs partial to let the user know they signed in successfully and create a new "sign out" button that will trigger the session delete route.

__Change nav upon successful user log in__

```
<% if(currentUser) {%>
<li>Welcome <%= currentUser.name%></li>
<li>
  <form action="/sessions?_method=DELETE" method="POST">
    <input type="submit" value="Log Out">
  </form>
</li>
  <% } else { %>
  <li><a href="/users/new">Sign Up</a></li>
  <li><a href="/sessions/new">Log In</a></li>
  <% } %>
</ul>
```

You can also block out whole sections of your website until your user has signed in by writing custom middleware that will authenticate whether the user logged in. [Middleware](https://expressjs.com/en/guide/using-middleware.html) functions are the foundation of Express. They have access to three things: req, res, and the next middleware function in the application (next). Middleware can execute code and decide whether to call the next middleware function in the stack with next().

__Write an authentication middleware to check whether user is logged in before allowing access to page:__
```
//middleware
const isAuthenticated = (req, res, next) => {
  if(req.session.currentUser) {
    return next();
  } else {
    res.redirect('/sessions/new')
  }
}

//how we insert the middleware 
//if user is not logged in, they are redirected to sign up page
data.get('/new', isAuthenticated, (req, res) => {
  res.render(
    'data/new.ejs, {
      currentUser: req.session.currentUser
    }
  )
})
```