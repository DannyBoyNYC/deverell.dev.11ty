---
title: ExpressJS
date: "2021-07-07"
description: Notes on ExpressJS
---

## Objective

- Build the backend for a [simple recipes app](https://recipes-spring.herokuapp.com) using ExpressJS, Mongoose, Docker and MongoDB.

## Scaffolding Our Server

1. In a new folder, run `$ npm init -y`
<!-- and edit package.json to specify `"main": "server.js",` as the entry for main -->
2. Install dependencies: `npm i -S express mongoose`
3. Install developmental dependencies: `npm i -D nodemon`
4. Create an npm script for nodemon in package.json:

```js
"scripts": {
  "start": "node server.js",
  "dev": "nodemon server.js"
},
```

### Express Routes

In `server.js`:

```js
const express = require("express");
const app = express();

// our first route
app.get("/", function (req, res) {
  res.send("Hello from the backend.");
});

const PORT = 3000;

app.listen(PORT, () => console.log(`Server running at port ${PORT}`));
```

Run this using `npm start`.

You should be able to view the [output](http://localhost:3000) at `http://localhost:3000`.

Create `/static/index.html` and edit `server.js`:

```js
const express = require("express");
const app = express();

app.get("/", function (req, res) {
  res.sendFile(__dirname + "/static/index.html");
});

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => console.log(`Server running at port ${PORT}`));
```

Instead of using `res.send` we are using `res.sendFile`. `__dirname` is a special Node global that gives us the current directory.

## Docker

We will use a Docker container that runs MongoDB locally.

1. Install the Docker [Desktop App](https://www.docker.com/products/docker-desktop)
1. Run in a Docker container on localhost:

```sh
$ docker run --name recipes-mongo -dit -p 27017:27017 --rm mongo:4.4.1
$ docker exec -it recipes-mongo mongo
$ show dbs
```

At this point you should be dropped into an interactive MongoDB shell.

`show dbs` allows you to see all the existing databases. In order to start using one, you do `$ use <database name>`.

Run `use recipesApp`.

If you run `db` now it'll show you're using the recipes database.

Let's play with the db a bit in the console.

Make a collection and a database within it. Run:

```sh
> db.recipes.insertOne({name: "Tuna", type: "sandwich", difficulty: "Easy"})
> db.recipes.count()
> db.recipes.findOne()
> db.recipes.findOne({ type: "sandwich" })
> db.recipes.updateOne( {name: "Tuna", type: "sandwich", difficulty: "Easy"}, { $set: { author: "Daniel" } } )
```

## Mongoose

Earlier we installed [Mongoose](https://mongoosejs.com), a schema builder for MongoDB, using npm.

First, import mongoose in server.js:

```js
const mongoose = require("mongoose");
```

We connect to a Mongo DB through the Mongoose's connect method, `mongoose.connect(URL, { options });`, and pass any configuration options in using an object.

Store the database connection string in a variable:

```js
const dataBaseURL = process.env.DB_URL || "mongodb://localhost:27017";
```

Call mongoose's connect method, passing it the URL.

Store the database URL in a variable:

```js
mongoose
  .connect(dataBaseURL, { useNewUrlParser: true })
  .then(() => console.log("MongoDb connected"))
  .catch((err) => console.log(err));
```

Mongoose's connect method returns a promise which we are using to log to the console (the terminal) and show any errors.

### Mongoose Schema

Mongoose uses [schemas](https://mongoosejs.com/docs/guide.html#definition) to define data and provides methods to add, remove, delete and more.

Create an instance of a Mongoose schema, RecipeSchema:

Add to `server.js`:

```js
const RecipeSchema = new mongoose.Schema({
  title: String,
  description: String,
  image: String,
});

const Recipe = mongoose.model("Recipe", RecipeSchema);
```

Models are defined by passing a Schema instance to mongoose.model. Here we are saving the model to a variable `Recipe`.

Once you have a model you can call methods on it. The actual interaction with the data happens with the Model.

Create a route in `server.js` that displays recipes:

```js
app.get("/api/recipes", function (req, res) {
  Recipe.find({}, function (err, results) {
    return res.send(results);
  });
});
```

Note the path: `/api/recipes`. Go to that endpoint in your browser to see the (empty) data.

### Import Data

We will create a new endpoint that populates our database with a starter data set using the `model.create()` method.

Pass some data to the database using [`model.create()`](https://mongoosejs.com/docs/api.html#model_Model.create), a shortcut for saving one or more documents to the database:

```js
app.get("/api/import", (req, res) => {
  Recipe.create(
    {
      title: "Lasagna",
      description:
        "Lasagna noodles piled high and layered full of three kinds of cheese to go along with the perfect blend of meaty and zesty, tomato pasta sauce all loaded with herbs.",
      image: "lasagna.png",
    },
    {
      title: "Pho-Chicken Noodle Soup",
      description:
        'Pho (pronounced "fuh") is the most popular food in Vietnam, often eaten for breakfast, lunch and dinner. It is made from a special broth that simmers for several hours infused with exotic spices and served over rice noodles with fresh herbs.',
      image: "pho.png",
    },
    {
      title: "Guacamole",
      description:
        "Guacamole is definitely a staple of Mexican cuisine. Even though Guacamole is pretty simple, it can be tough to get the perfect flavor - with this authentic Mexican guacamole recipe, though, you will be an expert in no time.",
      image: "guacamole.png",
    },
    {
      title: "Hamburger",
      description:
        "A Hamburger (often called a burger) is a type of sandwich in the form of  rounded bread sliced in half with its center filled with a patty which is usually ground beef, then topped with vegetables such as lettuce, tomatoes and onions.",
      image: "hamburger.png",
    }
  );
});
```

Now go to the import endpoint (note that the page loads indefinitely) and then return to the `http://localhost:3000/api/recipes` endpoint to see the data.

The page loads indefinitely because the endpoint never actually returns anything to the browser.

Let's return an HTTP status:

```js
    {
      title: 'Hamburger',
      description:
        'A Hamburger (often called a burger) is a type of sandwich in the form of  rounded bread sliced in half with its center filled with a patty which is usually ground beef, then topped with vegetables such as lettuce, tomatoes and onions.',
      image: 'hamburger.png',
    },
    function(err) {
      if (err) return console.log(err);
      return res.sendStatus(201);
    },
```

Travelling to `http://localhost:3000/api/import` will import the data again but, this time, since we return something to the browser it will not be stuck loading.

### Aside - Status Codes

`sendStatus` communicates with the front end by returning a standard [http status code](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes). As the backend developer it is up to you to return appropriate status codes.

## Express Static Files

To serve [static files](https://expressjs.com/en/starter/static-files.html) such as images, CSS files, and JavaScript files, we use the `express.static` built-in function in Express.

Add the following to server.js:

```js
app.use(express.static("static"));
```

## Front End

Let's output the data using JavaScript in a simple index page.

In the body tag of `index.html`:

```html
<h1>Recipes!</h1>
<ul>
  <li><a href="http://localhost:3012/api/recipes">View recipes JSON</a></li>
  <li><a href="http://localhost:3012/api/import">Import recipes</a></li>
  <li>
    <a href="http://localhost:3012/api/kill-all">Delete all recipes</a>
  </li>
</ul>
<div class="recipes"></div>

<script src="scripts.js"></script>
```

And in the head:

```html
<link rel="stylesheet" href="styles.css" />
```

Use the browser's fetch API to call our api endpoint:

```js
fetch(`api/recipes`)
  .then((response) => response.json())
  .then((recipes) => console.log(recipes));
```

Examine the browser's console.

Render some HTML to the DOM:

```js
function getRecipes() {
  document.querySelector(".recipes").innerHTML = ``;
  fetch(`api/recipes`)
    .then((response) => response.json())
    .then((recipes) => renderRecipes(recipes));
}

function renderRecipes(recipes) {
  recipes.forEach((recipe) => {
    recipeEl = document.createElement("div");
    recipeEl.innerHTML = `
      <img src="img/${recipe.image}" />
      <h3>${recipe.title}</h3>
      <p>${recipe.description}</p>
    `;
    document.querySelector(".recipes").append(recipeEl);
  });
}

getRecipes();
```

## Reorganizing with CommonJS

Before we get any further we are going to reorganize our code using CommonJS.

## DB

db.js:

```js
const mongoose = require("mongoose");

const dataBaseURL = process.env.DB_URL || "mongodb://localhost:27017";

const connect = mongoose
  .connect(dataBaseURL, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log("MongoDb connected"))
  .catch((err) => console.log(err));

module.exports = connect;
```

Import it into `server.js`:

```js
const connect = require("./db");
```

### Controllers

Create a new folder `api` and a file inside called `recipe.controllers.js`. We'll export each handler and create the functions in this file one by one. They are just empty functions for the moment.

Add the following to `recipe.controllers.js`:

```js
const mongoose = require("mongoose");
const Recipe = mongoose.model("Recipe");

exports.getOne = async (req, res) => {
  res.send("hi");
};

exports.getMany = async (req, res) => {
  res.send("hi");
};

exports.createOne = async (req, res) => {
  res.send("hi");
};

exports.updateOne = async (req, res) => {
  res.send("hi");
};

exports.removeOne = async (req, res) => {
  res.send("hi");
};

exports.import = async (req, res) => {
  res.send("hi");
};

exports.deleteAll = async (req, res) => {
  res.send("hi");
};
```

`exports` allows the functions to be available for import elsewhere in our application.

Update `server.js` to require our controllers:

```js
const recipeControllers = require("./api/recipe.controllers");
```

Now we can call the functions in `recipe.controllers`.

Delete the find route from server.js. and add the following to `server.js`:

```js
app.get("/api/recipes", recipeControllers.getMany);
app.get("/api/recipes/:id", recipeControllers.getOne);
app.post("/api/recipes", recipeControllers.createOne);
app.put("/api/recipes/:id", recipeControllers.updateOne);
app.delete("/api/recipes/:id", recipeControllers.removeOne);
app.get("/api/import", recipeControllers.import);
app.get("/api/kill-all", recipeControllers.deleteAll);
```

Update getMany's definition in `recipe.controllers.js` to send a json snippet:

```js
exports.getMany = async (req, res) => {
  res.send([
    {
      title: "Lasagna",
      description:
        "Lasagna noodles piled high and layered full of three kinds of cheese to go along with the perfect blend of meaty and zesty, tomato pasta sauce all loaded with herbs.",
      image: "lasagna.png",
    },
  ]);
};
```

And test the API endpoint.

You should see the recipe in the browser and, at the specified route `/api/recipes'`, you should see the json in the browser.

### Recipe Model

Add a new file `recipe.model.js` to `api` for our Recipe Model.

Require Mongoose in this file, and create a new Schema object:

```js
const mongoose = require("mongoose");
const Schema = mongoose.Schema;

const RecipeSchema = new Schema({
  title: String,
  description: String,
  image: String,
});

module.exports = mongoose.model("Recipe", RecipeSchema);
```

Delete the model in server.js and import `recipe.model.js`. Order is important. In `server.js` we must require the model _before_ the controllers.

```js
const recipeModel = require("./api/recipe.model");
const recipeControllers = require("./api/recipe.controllers");
```

Update the `getMany()` function in `recipe.controllers` to query Mongo with the `find()` method.

```js
exports.getMany = async (req, res) => {
  Recipe.find({}, (err, results) => {
    return res.send(results);
  });
};
```

Check that the server is still running and then visit the API endpoint for all recipes [localhost:3000/api/recipes](localhost:3000/api/recipes). You'll get JSON data back from the database - albeit an empty array `[]` st this point.

## Mongoose Model.create

We will use the Mongoose method `Model.create` to import data into our application.

Delete the import route in server.js and define it in `recipe.controllers.js`:

```js
exports.import = function (req, res) {
  Recipe.create(
    {
      title: "Lasagna",
      description:
        "Lasagna noodles piled high and layered full of three kinds of cheese to go along with the perfect blend of meaty and zesty, tomato pasta sauce all loaded with herbs.",
      image: "lasagna.png",
    },
    {
      title: "Pho-Chicken Noodle Soup",
      description:
        'Pho (pronounced "fuh") is the most popular food in Vietnam, often eaten for breakfast, lunch and dinner. It is made from a special broth that simmers for several hours infused with exotic spices and served over rice noodles with fresh herbs.',
      image: "pho.png",
    },

    {
      title: "Guacamole",
      description:
        "Guacamole is definitely a staple of Mexican cuisine. Even though Guacamole is pretty simple, it can be tough to get the perfect flavor - with this authentic Mexican guacamole recipe, though, you will be an expert in no time.",
      image: "guacamole.png",
    },

    {
      title: "Hamburger",
      description:
        "A Hamburger (often called a burger) is a type of sandwich in the form of  rounded bread sliced in half with its center filled with a patty which is usually ground beef, then topped with vegetables such as lettuce, tomatoes and onions.",
      image: "hamburger.png",
    },
    () => {
      res.sendStatus(201);
    }
  );
};
```

`Recipe` refers to the mongoose Recipe model we imported. `Model.create()` is a mongoose method

## Mongoose Model.DeleteMany()

Review some of the [documentation](http://mongoosejs.com/docs/queries.html) for Mongoose and create a script to delete all recipes with [deleteMany](http://mongoosejs.com/docs/queries.html).

We called our endpoint 'kill-all.'

Add the corresponding function to the controllers file:

```js
exports.deleteAll = async (req, res) => {
  Recipe.deleteMany({}, (err) => {
    if (err) return console.log(err);
    return res.sendStatus(202);
  });
};
```

Run the function by visiting the `api/kill-all endpoint and then returning to the recipes endpoint to examine the results.

In this example we are deleting only those recipes whose title is Lasagna.

Change the filter: `{ title: 'Lasagna' }` to: `{}` to remove them all and run the functions again.

## Mongoose Model.create

We used `create()` in our import function in order to add multiple documents to our Recipes collection. Our POST handler uses the same method to add a single Recipe to the collection. Once added, the response is the full new Recipe's JSON object.

Edit `recipe-controllers.js`:

```js
exports.createOne = async (req, res) => {
  Recipe.create(req.body, (err, recipe) => {
    if (err) return console.log(err);
    return res.send(recipe);
  });
};
```

Add a form to index.html:

```html
<form id="addForm">
  <input type="text" placeholder="Recipe Title" name="title" value="Lasagna" />
  <input type="text" placeholder="Image" name="image" value="lasagna.png" />
  <textarea type="text" placeholder="Description" name="description">
Lasagna noodles piled high and layered full of three kinds of cheese to go along with the perfect blend of meaty and zesty, tomato pasta sauce all loaded with herbs.</textarea
  >
  <button type="submit">Submit</button>
</form>
```

Express has built in decoders that parse incoming requests with urlencoded or json payloads.

`npm i body-parser`

Add to server.js:

```js
// app.use(express.json({ extended: false }));
// app.use(express.urlencoded({ extended: false }));
import { json, urlencoded } from 'body-parser'
...
app.use(json());
app.use(urlencoded({ extended: true }));
```

The HTML form elements have an attribute named enctype, if not specified, its value defaults to "application/x-www-form-urlencoded".

Use promise chaining in scripts.js:

```js
// SEE ALTERNATIVE
function addRecipe(event) {
  event.preventDefault();

  const { title, image, description } = event.target;

  const recipe = {
    title: title.value,
    image: image.value,
    description: description.value,
  };

  fetch("api/recipes", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify(recipe),
  })
    .then((response) => response.json())
    .then(getRecipes);
}

const addForm = document.querySelector("#addForm");
addForm.addEventListener("submit", addRecipe);
```

Alternative:

```js
// Get the form
let form = document.querySelector("#post");

// Get all field data from the form
// returns a FormData object
let data = new FormData(form);

for (let [key, value] of data) {
  console.log(key);
  console.log(value);
}
```

Kill all exiting entries and test the form.

## Deleting a Recipe

Our next REST endpoint, _removeOne_, uses [model.deleteOne](https://mongoosejs.com/docs/api/model.html#model_Model.remove). Add this to `recipe.controllers.js`.

```js
exports.removeOne = async (req, res) => {
  let id = req.params.id;
  Recipe.deleteOne({ _id: id }, () => {
    return res.sendStatus(202);
  });
};
```

Check it out with curl (replacing the id at the end of the URL with a _known id_ from the GET (`api/recipes`) endpoint):

```sh
curl -i -X DELETE http://localhost:3000/api/recipes/<id>
```

## Deleting on the Front End

- Add a delete link to the generated output

```js
function renderRecipes(recipes) {
  recipes.forEach((recipe) => {
    recipeEl = document.createElement("div");
    recipeEl.innerHTML = `
      <img src="img/${recipe.image}" />
      <h3>${recipe.title}</h3>
      <p>${recipe.description}</p>
      <p>${recipe._id}</p>
      <a class="delete" data-id=${recipe._id} href="#">Delete</a>
    `;
    document.querySelector(".recipes").append(recipeEl);
  });
}
```

Note - the proper use of buttons vs links.

```css
.delete {
  background: unset;
  margin: unset;
  border: none;
  padding: 0;
  color: #007eb6;
  cursor: pointer;
}
```

Note the use of `data-id` above.

Use fetch passing it a second parameter - options:

```js
function deleteRecipe(event) {
  fetch(`api/recipes/${event.target.dataset.id}`, {
    method: "DELETE",
  }).then(location.reload());
}
```

Here are the client side scripts with a bit of restructuring and new functionality near the bottom to support deletion and importing:

```js
function getRecipes() {
  document.querySelector(".recipes").innerHTML = ``;
  fetch(`api/recipes`)
    .then((response) => response.json())
    .then((recipes) => renderRecipes(recipes));
}

function addRecipe(event) {
  event.preventDefault();
  const { title, image, description } = event.target;
  const recipe = {
    title: title.value,
    image: image.value,
    description: description.value,
  };
  fetch("api/recipes", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify(recipe),
  })
    .then((response) => response.json())
    .then(getRecipes);
}

function renderRecipes(recipes) {
  recipes.forEach((recipe) => {
    // destructure
    const { _id, title, image, description } = recipe;
    recipeEl = document.createElement("div");
    recipeEl.innerHTML = `
    <img src="img/${image}" />
    <h3>${title}</h3>
    <p>${description}</p>
    <button class="delete" data-id=${recipe._id} href="#">Delete</button>
  `;
    return document.querySelector(".recipes").append(recipeEl);
  });
}

function deleteRecipe(event) {
  fetch(`api/recipes/${event.target.dataset.id}`, {
    method: "DELETE",
  }).then(getRecipes());
}

// new
function seed() {
  fetch("api/import").then(getRecipes);
}

function handleClicks(event) {
  if (event.target.matches("[data-id]")) {
    deleteRecipe(event);
  } else if (event.target.matches("#seed")) {
    seed();
  }
}

document.addEventListener("click", handleClicks);

const addForm = document.querySelector("#addForm");
addForm.addEventListener("submit", addRecipe);

getRecipes();
```

## Find by ID

Let's create a detail page for each recipe using findById function.

Start by filling out the findByID function to use Mongoose's `Model.findOne` in `recipe.controllers`:

```js
exports.getOne = async (req, res) => {
  const id = req.params.id;
  Recipe.findOne({ _id: id }, (err, json) => {
    if (err) return console.log(err);
    return res.send(json);
  });
};
```

Then, add a link to the details page we are about to create.

In the `renderRecipes` function in scripts.js:

```html
<h3><a href="detail.html?recipe=${recipe._id}">${recipe.title}</a></h3>
```

Note that we are including the recipe id (`_id`) in the URL.

## Detail Page

- Save index.html as detail.html
- Remove the form
- change the html:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <link rel="stylesheet" href="css/styles.css" />
    <title>Recipes!</title>
  </head>
  <body>
    <div id="root">
      <h1>Recipes!</h1>

      <div class="recipe"></div>
    </div>
    <script src="js/details.js"></script>
  </body>
</html>
```

Create `details.js` and within a new function:

```js
function showDetail() {
  const urlParams = new URLSearchParams(window.location.search);
  const recipeId = urlParams.get("recipe");

  fetch(`api/recipes/${recipeId}`, {
    method: "GET",
  })
    .then((response) => response.json())
    .then((recipe) => renderRecipe(recipe));
}

function renderRecipe(recipe) {
  recipeEl = document.createElement("div");
  recipeEl.innerHTML = `
    <img src="img/${recipe.image}" />
    <h3>${recipe.title}</h3>
    <p>${recipe.description}</p>
    <a href="/">Back</a>
    `;
  document.querySelector(".recipe").append(recipeEl);
}

showDetail();
```

Note the use of [URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams).

Now we should be able to navigate to the detail page and see the recipe.

## Mongoose Model.findByIdAndUpdate

We will use a form in `detail.html` to update and edit the recipe.

Update `recipe.controllers` to use `findByIdAndUpdate`:

```js
exports.updateOne = async (req, res) => {
  console.log(req.body);
  const id = req.params.id;
  Recipe.findByIdAndUpdate(id, req.body, { new: true }, (err, response) => {
    if (err) return console.log(err);
    res.send(response);
  });
};
```

Edit the form in `detail.html`:

```html
<h3>Edit Recipe</h3>
<form id="editForm">
  <input type="text" placeholder="Recipe Title" name="title" />
  <input type="text" placeholder="Image" name="image" />
  <textarea type="text" placeholder="Description" name="description"></textarea>
  <button>Update</button>
</form>
```

Note the button action.

Populate the form fields using data from the recipe in `details.js`:

```js
function renderRecipe(recipe) {
  const { image, title, description } = recipe;
  recipeEl = document.createElement("div");
  recipeEl.innerHTML = `
    <img src="img/${image}" />
    <h3>${title}</h3>
    <p>${description}</p>
    <a href="/">Back</a>
    `;

  editForm.title.value = title;
  editForm.image.value = image;
  editForm.description.value = description;
  document.querySelector(".recipe").append(recipeEl);
}
```

We will test our updating capabilities by creating a function for the form's `updateRecipe` call using `fetch` and an options object.

```js
const updateRecipe = (event) => {
  event.preventDefault();
  const urlParams = new URLSearchParams(window.location.search);
  const recipeId = urlParams.get("recipe");
  const { title, image, description } = event.target;
  const updatedRecipe = {
    _id: recipeId,
    title: title.value,
    image: image.value,
    description: description.value,
  };
  fetch(`api/recipes/${recipeId}`, {
    method: "PUT",
    body: JSON.stringify(updatedRecipe),
    headers: {
      "Content-Type": "application/json",
    },
  }).then(showDetail);
};

const editForm = document.querySelector("#editForm");
editForm.addEventListener("submit", updateRecipe);
```

Editing the form should now change the entry.

## Adding File Upload

We will add file uploading to our API using the [File Upload](https://www.npmjs.com/package/express-fileupload) npm package for ExpressJS.

Install File Upload:

`npm i express-fileupload -S`

Require, register and create a route for it in `server.js`:

```js
...
const fileUpload = require('express-fileupload');
...
app.use(fileUpload());
...
app.post('/api/upload', recipeControllers.upload);
```

Here is a working function for the api endpoint. Add it to `recipe.controllers.js`:

```js
exports.upload = function (req, res) {
  console.log(req.files);
  if (Object.keys(req.files).length == 0) {
    return res.status(400).send("No files were uploaded.");
  }
  let file = req.files.file;
  file.mv(`./public/img/${req.body.filename}`, (err) => {
    if (err) {
      return res.status(500).send(err);
    }
    res.json({ file: `public/img/${req.body.filename}` });
    console.log(res.json);
  });
};
```

Looking at the [example project](https://github.com/richardgirges/express-fileupload/tree/master/example) we find a form to use as a starting point.

```html
<form action="/api/upload" method="post" enctype="multipart/form-data">
  <input type="file" name="file" accept="image/png" />
  <input type="text" placeholder="File name" name="filename" />
  <button type="submit">Upload</button>
</form>
```

Note the encType attribute on the form.

Upload an image (reuse one of the images in `/public/img/`) and give it a unique name and create a recipe that uses the new image.

Once successful, set the return value in the corresponding controller's function to `return res.sendStatus(200);`.

## Update the Recipe Model

Add a default `created` value of type Date to `recipe.model`:

```js
const RecipeSchema = new Schema({
  title: String,
  created: {
    type: Date,
    default: Date.now,
  },
  description: String,
  image: String,
});
```

`<p>Created ${recipe.created}<p>`

Test Mongoose by adding new properties to our recipes.

Edit the `import` function to include ingredients and preparation arrays:

```js
exports.import = function (req, res) {
  Recipe.create(
    {
      title: "Lasagna",
      description:
        "Lasagna noodles piled high and layered full of three kinds of cheese to go along with the perfect blend of meaty and zesty, tomato pasta sauce all loaded with herbs.",
      image: "lasagna.png",
      ingredients: ["salt", "honey", "sugar", "rice", "walnuts", "lime juice"],
      preparation: [
        { step: "Boil water" },
        { step: "Fry the eggs" },
        { step: "Serve hot" },
      ],
    },
    {
      title: "Pho-Chicken Noodle Soup",
      description:
        'Pho (pronounced "fuh") is the most popular food in Vietnam, often eaten for breakfast, lunch and dinner. It is made from a special broth that simmers for several hours infused with exotic spices and served over rice noodles with fresh herbs.',
      image: "pho.png",
      ingredients: ["salt", "honey", "sugar", "rice", "walnuts", "lime juice"],
      preparation: [
        { step: "Boil water" },
        { step: "Fry the eggs" },
        { step: "Serve hot" },
      ],
    },
    {
      title: "Guacamole",
      description:
        "Guacamole is definitely a staple of Mexican cuisine. Even though Guacamole is pretty simple, it can be tough to get the perfect flavor - with this authentic Mexican guacamole recipe, though, you will be an expert in no time.",
      image: "guacamole.png",
      ingredients: ["salt", "honey", "sugar", "rice", "walnuts", "lime juice"],
      preparation: [
        { step: "Boil water" },
        { step: "Fry the eggs" },
        { step: "Serve hot" },
      ],
    },
    {
      title: "Hamburger",
      description:
        "A Hamburger (often called a burger) is a type of sandwich in the form of  rounded bread sliced in half with its center filled with a patty which is usually ground beef, then topped with vegetables such as lettuce, tomatoes and onions.",
      image: "hamburger.png",
      ingredients: ["salt", "honey", "sugar", "rice", "walnuts", "lime juice"],
      preparation: [
        { step: "Boil water" },
        { step: "Fry the eggs" },
        { step: "Serve hot" },
      ],
    },
    function (err) {
      if (err) return console.log(err);
      return res.sendStatus(202);
    }
  );
};
```

If you delete with the `killall` endpoint and reload the recipes endpoint, it will not include the arrays.

Add new properties to our Recipe schema.

```js
const mongoose = require("mongoose");
const Schema = mongoose.Schema;

const RecipeSchema = new Schema({
  title: String,
  created: {
    type: Date,
    default: Date.now,
  },
  description: String,
  image: String,
  ingredients: Array,
  preparation: Array,
});

module.exports = mongoose.model("Recipe", RecipeSchema);
```

Kill and reimport the data using Postman. The data may be in a different order than in the schema.

There are eight data types supported by Mongoose:

1. String
2. Number
3. Date
4. Buffer
5. Boolean
6. Mixed
7. ObjectId
8. Array

Each data type allows you to specify:

- a default value
- a custom validation function
- indicate a field is required
- a get function that allows you to manipulate the data before it is returned as an object
- a set function that allows you to manipulate the data before it is saved to the database
- create indexes to allow data to be fetched faster

Certain data types allow you to customize how the data is stored and retrieved from the database. A String data type also allows you to specify the following additional options:

- convert it to lowercase
- convert it to uppercase
- trim data prior to saving
- a regular expression that can limit data allowed to be saved during the validation process
- an enum that can define a list of strings that are valid

```js
  const {
    created,
    image,
    title,
    description,
    ingredients,
    preparation,
  } = recipe;
  ...
  recipeEl.innerHTML = `
  <img src="img/${image}" />
  <h3>${title}</h3>
  <p>${description}</p>
  <h4>Ingredients</h4>
  <ul>
    ${ingredients[0]}
  </ul>
  <h4>Preparation</h4>
  <ul>
    ${preparation.map((prep) => `<li>${prep.step}</li>`).join("")}
  </ul>
  <p>Created ${created}<p>
  <a href="/">Back</a>
  `;
```

The Array data type allows you to store JavaScript-like arrays. With an Array data type, you can perform common JavaScript array operations on them, such as push, pop, shift, slice, etc.

## Deployment

We will deploy to [Heroku](https://devcenter.heroku.com/articles/git).

Before deployment we remove sensitive information and set environment variables for our project.

Create a `.env` file in the root:

`.env`:

```sh
NODE_ENV=development
DATABASE=mongodb+srv://daniel:dd2345@recipes-3k4ea.mongodb.net/test?retryWrites=true&w=majority
PORT=3000
```

Be sure to replace the DATABASE with your own url.

Install a helper [dotenv](https://www.npmjs.com/package/dotenv):

`$ npm install dotenv`

Require it in `server.js`:

```js
require("dotenv").config();
```

Note: you should use your own database. You should not push the .env file to Github by adding it to `.gitignore`.

Test it in `server.js`. Replace the existing dataBaseURL variable with:

```
const dataBaseURL = process.env.DATABASE;
```

Ensure that `server.js` specifies `process.env`. Replace the lines at the bottom with:

```js
const PORT = process.env.PORT;
app.listen(PORT, () => console.log(`Server running at port ${PORT}`));
```

The server should still be running successfully on port 3000.

Ensure that your package json includes `server.js` as the `main` file:

`"main": "server.js",`

and that you have a node start script defined:

`"start": "node server.js"`

Create a git repo and deploy to Github.

1. Create an account and login to Heroku
2. Create a project
3. Go to the deployment tab and specify with Github repo and branch you are deploying from and enable automatic deploys. Be sure to monitor the build.
4. Push the desired branch to Github
