---
title: Severless
date: "2020-11-03"
description: "Notes on Serverless"
---

This is a review of [An introduction to serverless functions by Jason Lengstorf of Netlify](https://frontendmasters.com/courses/serverless-functions/)

## Setup

This course uses the Netlify CLI to run a dev environment with access to Netlify services.

The sample site itself employs a simple Eleventy site. (11ty uses BrowserSync to run locally.)

One interesting early item in the HTML is the use of the `<template>` tag.

```html
<template id="movie-template">
  <div class="movie">
    <img src="" />

    <h2></h2>
    <p class="tagline"></p>

    <h3>Viewer Ratings</h3>
    <ul class="scores"></ul>
  </div>
</template>
```

This inserts a DOM element but doesn't display it. Clone this node with:

```js
const template = document.querySelector("#movie-template")

const element = template.content.cloneNode(true)
```

And then use standard DOM manipulation:

`element.querySelector("h2").innerText = movie.title`

- Netlify toml file specifies the func directory specifies build command, publish directory and functions directory.
- `$ ntl dev` runs the dev server

```toml
[build]
  functions = 'functions'
  command = "npm run build"
  publish = "public"

[[redirects]]
  from = '/api/*'
  to = '/.netlify/functions/:splat'
  status = 200
```

Functions are written using CommonJS:

```js
exports.handler = async () => {
  return {
    statusCode: 200,
    body: "boop",
  }
}
```

- serverless ƒs call something called a `handler`
- `async` avoids the use of callbacks

## Returning JSON

Load a sample JSON file for data.

Need to stringify the data, data must be a string

```js
const movies = require(...)

exports.handler = async () => {
  return {
    statusCode: 200,
    body: JSON.stringify(movies),
  }
}
```

Call the serverless ƒ using fetch:

```js
async function initialize() {
  const movies = await fetch('/.netlify/functions/movies').then((response) =>
    response.json(),
  ) ...
```

## Sending and Receiving Data

Get the data from the query string parameters. In the serverless ƒ:

```js
// hello.js
exports.handler = async (event) => {
  console.log(event.queryStringParameters)
  const { text } = event.queryStringParameters
  return {
    statusCode: 200,
    body: `You said ${text}`,
  }
}
```

Now `localhost:62094/.netlify/functions/hello?test=test&num=1` will display a string in the browser and log to the console (an object - key/string value):

`{ text: 'Hello', munber: '3' }`



---

## Resources

My Site on Netlify

[Netlify App](https://app.netlify.com/sites/fem-dd-serverless/overview)

My Gihub Repo

[github.com/DannyBoyNYC/fem-serverless](https://github.com/DannyBoyNYC/fem-serverless)

Hasura Console

[legible-viper-42](https://legible-viper-42.hasura.app/console/)

Peek inside a JWT at

[JSON Web Tokens - jwt.io](https://jwt.io/)

Netlify Identity Widget Docs

[GitHub - netlify/netlify-identity-widget: A zero config, framework free Netlify Identity widget](https://github.com/netlify/netlify-identity-widget)

- [Netlify Functions](https://www.netlify.com/products/functions/?utm_source=fem-sls&utm_medium=functions-jl&utm_campaign=devex)
- [Netlify Identity](https://docs.netlify.com/visitor-access/identity/?utm_source=fem-sls&utm_medium=functions-jl&utm_campaign=devex)
- [Hasura](https://cloud.hasura.io/)
- [Heroku](https://www.heroku.com/)
