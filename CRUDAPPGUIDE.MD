# CREATING A CRUD APP GUIDE


Uses Express.js, MongoDB, Mongoose


## Terminal Set-Up: 

### Open 4 tabs in your terminal -- commands, mongod, mongo, node
  * In commands -- make sure you are in the correct file then…
  npm init (if you’d like to go through all options) or npm init -y (go with your previous set up)
    * Npm i express
    * Npm i ejs
    * Npm i mongoose
    * Npm i method-override
  * In MongoD -- type  “mongod --dpbath filepath (ie. ~/dev/data/db/)
  * In Mongo -- type mongo
  * In node -- type nodemon

## Basic File Structure Set-Up: 

CollectionName
----Controllers
     ------collectionNamesController.js
----Models
    ------data.js
----node_modules < -- You get this through terminal set up
----Views
     ---partials
	---put repeating HTML sections in files here (header.ejs, footer.ejs. etc.)
     ---index.ejs
     ---edit.ejs
     ---show.ejs
     ---new.ejs
     ---any other ejs routes
-----package_lock.json
-----package.json
-----server.js

## Individual File Set Up

### Home Base - server.js

server.js ← this file connects everything

Set Up:

Set up variables
```
const express = require(‘express’);
const mongoose = require(‘mongoose’);
const methodOverride = require(‘method-override’);
const collectionsController = require(‘./controllers/collectionNameController.js');
const app = express();
```

Set up middleware
```
//lets you use method override for PUT and DELETE methods in routes 
app.use(methodOverride(‘_method’));
//lets you parse your body data (?)
app.use(express.urlencoded({extended: false}));
//use the controller file
app.use(collectionsController);
```
Connect mongoose to mongodb
```
mongoose.connect(
‘mongodb://localhost:27017/dbName’, {
  useNewURLParser: true,
  useUnifiedTopology: true,
  useFindAndModify: false
},
() => {
  console.log(‘connected to mongod’);
}
);
```

Connect your app to localhost:3000 or whichever port you want to use
```
app.listen(3000, () => {
  console.log(‘listening…’);
});
```

### Controller File - collectionNameController.js :

Controllers
---- collectionsNameController.js  ←	your routes go in this file

In your server.js, you connected with the controller file through the following code: 
	const fruitsController = require(‘./controllers/fruits.js’);
	app.use(fruitsController);

In your collectionsNameController.js file -- 

Set up - 
```
const express = require(‘express’);
const Fruit = require(‘../models/fruits.js’);
const router = express.Router();

Module.exports = router;
```
----- All Your Routes Will Go Here Later -------

At the bottom, send your router out to the server.js
```
module.exports = router;
```
### Date File - collectionName.js within models directory

At the top -- declare get mongoose set up
```
const mongoose = require(‘mongoose’);
```
At the bottom -- export data
```
module.exports = CollectionName;
```

### All Views Files - ie views/index.ejs

Set up all views files with HTML document structure. If you 
Are connecting to a partial for certain elements, use : 
```
<%- include('./partials/filename.ejs' %>
```
	

## Create Main Routes 

There are 7 RESTful routes that allow you to Create, Read, Update, and Delete data. 

| # | Action  | URL          | HTTP Verb | Mongoose                                 |
|---|:-------:|:------------:|:---------:|:----------------------------------------:|
| 1 | Index   |/page/        |GET        | Data.find()                              |
| 2 | Show    |/page/:id     |GET        | Data.findById()                          |
| 3 | New     |/page/new     |GET        | -                                        |
| 4 | Create  |/page/        |POST       | Data.create()                            |
| 5 | Edit    |/page/:id/edit|GET        | Data.findById()                          |
| 6 | Update  |/page/:id/    |PUT        | Data.findByIdAndUpdate()                 |
| 7 | Destroy |/page/:id     |DELETE     | Data.findByIdAndRemove()                 |

All routes will go in the collectionName.js file. 

### Index Route
```
router.get('/', (req, res) => {
  Data.find({}, (err, allData)=> {
    res.render('index.ejs', {
      data: allData
    });
  });
});
```

### Show Route
```
router.get('/:id', (req, res) => {
  Data.findById(req.params.id, (err, foundData)=> {
    res.render('show.ejs', {
      data: foundData
    });
  });
});
```

### New Route and Create Route

* The new route is rendered with a get request
```
router.get('/new', (req, res) => {
  res.render('new.ejs);
}
```
* The create route is triggered through a post request to create new data
```
router.post('/', (req, res) => {
  Data.create(req.body, () => {
    res.redirect('/');
  })
})
```

### Edit Route and Update Route

* The edit route is rendered with a get request
```
router.get('/:id/edit', (req, res) => {
  Data.findById(req.params.id, (err, foundData) => {
    res.render(
      'edit.js',
      {
        data: foundData
      }
    );
  });
});
```
* The update route is triggered through a put request
```
router.put('/:id', (req, res) => {
  Data.findByIdAndUpdate(req.params.id, req.body, {new:true}, (err, updateModel) => {
    res.redirect('/');
  });
});
```
### Destroy Route
```
router.delete('/:id', (req, res)=> {
  Data.findByIdAndRemove(req.params.id, (err, data) => {
    res.redirect('/')
  });
});
```
## You can test certain routes with curl commands in the terminal

* curl is a command line tool that acts like a browser, and you can use it to make requests to a website. It will take a response and write it into the terminal without formatting. **Capitalization matters in curl!**

  * Test POST request : curl -X POST localhost:3000/home
  * Test POST request with data : curl -X POST -d name="kiwi" -d color="green localhost:3000/home
  * Test DELETE request : curl -X DELETE localhost:3000/home/01

## Insert Initial Data: 

### First, you should set up the collectionName.js file in your model directory: 

* Initial set up of this file should already be complete. 
* Add a new scheme that will define the shape of the documents in the collection. 
```
const dataSchema = new Schema({
  name: String, 
  age: {type: Number, required: true},
  likes: {type: Number, default: 0},
}, {
  timestamps: true
})
```
* Then, create a new data model. This will convert the schema into a model to be store in the 'data' collection. Models are fancy constructors, and an instance of a model is called a document
```
const Data = mongoose.model('Data', 'dataSchema');
```
### Now, add some initial data into your collection if you need to...

1) Insert data with post requests after creating new.ejs. 
2) Create a Seed Route : 
```
app.get('/seed', (req, res) => {
  Data.create(
    [
      {
        Name: Ellie,
        Age: 29, 
        userName: ellieRuns02
      },
      {
        Name: ---, 
        Age: ---, 
        userName: --, 
      }
    ],
    (err, data) => {
      res.redirect('/');
    }
  )
});
```
3. Add data using your server.js file (don't use nodemon with this and comment out after to not replicate a bunch of data)

* create a piece of data - make sure it matched the structure of your schema
```
const myData = {
  Name: 'Elliot',
  Age: 57,
  userName: elliot1234
}
```
* insert the piece of data into your Model
```
Data.create(myData, (error, data) => {
  if (error) =>
  {
    console.log(error);
  } else {
    console.log(data);
  }
})
```

* You can use the find method to make sure this worked
```
Data.find((err, data) => {
  console.log(data);
  db.close();
})
```

## Finish Off Your Views Files

### index.ejs

If you are rendering data on this page, remember that you've passed in all the data, so you have to do some extra work to select an individual object. 

Two Methods are -- (1) for of loop (2) regular for loop

for of loop example
```
<% for(let item of data) {%>
  <li><%=item.name %></li>
<%}%>
```

for loop example
```
<% for (let i = 0; i < data.length; i++>) {%>
  <li><%=data[i].name%></li>
<%}%>
```
### show.ejs
This page will render one object from the data collection. You can just use the variable name you passed in through server.js to get values: 

```
<h1><%= dataElement.name %></h1>
```
### new.ejs
This page should have a form that, on submit, complete a post request

```
<form action="/" method="POST">
  <input type="text" name="name"/>
  <input type="number" name="age"/>
  <input type="submit" value="Create New Data">
</form>
```

### edit.ejs
This page should also have a form, but it should send a put request instead. Make sure you have method-override installed. Unlike new.ejs, it's helpful for this page's form to include a value section, which will display the current data.
```
<form action="/<%=data.id%>?_method=PUT" method="POST">
  <input type="text" name="name" value="data.name">
  <input type="number" name="age" value="data.age">
  <input type="submit" value="Make Changes">
</form>
```

### delete button or form
This can go on any page, and it should execute a delete request. Check that you have method-override installed first. Example below would be for an index page.

```
<form action="/<%=data[i].id%>?_method=DELETE" method="POST">
  <input type="submit" value="DELETE">
</form>
```