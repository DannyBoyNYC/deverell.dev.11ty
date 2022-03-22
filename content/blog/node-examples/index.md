---
title: NodeJS Intro
date: "2021-07-07"
description: Intro Samples for Node JS
---

## Resources

An [Introduction](https://nodejs.dev/learn) to NodeJS.

## Try

Create a file and add:

```js
function addItems(num1, num2) {
  console.log(num1 + num2);
}

addItems(1, 2);
```

Run the file from the terminal:

```sh
node <filename>.js
```

Clearly, adding a line such as `const test = document.querySelector('???')` makes no sense in the context of a node app.

## Understand

Both the browser and Node.js use JavaScript as their programming language.

Building apps that run in the browser is a completely different thing than building a Node.js application.

Despite the fact that it's always JavaScript, there are some key differences that make the experience radically different.

From the perspective of a frontend developer who extensively uses JavaScript, Node.js apps bring with them a huge advantage: the comfort of programming everything - the frontend and the backend - in a single language.

You have a huge opportunity because we know how hard it is to fully, deeply learn a programming language, and by using the same language to perform all your work on the web - both on the client and on the server, you're in a unique position of advantage.

What changes is the ecosystem.

In the browser, most of the time what you are doing is interacting with the DOM, or other Web Platform APIs like Cookies. Those do not exist in Node.js, of course. You don't have the document, window and all the other objects that are provided by the browser.

And in the browser, we don't have all the nice APIs that Node.js provides through its modules, like the filesystem access functionality.

Another big difference is that in Node.js you control the environment. Unless you are building an open source application that anyone can deploy anywhere, you know which version of Node.js you will run the application on. Compared to the browser environment, where you don't get the luxury to choose what browser your visitors will use, this is very convenient.

This means that you can write all the modern ES6-7-8-9 JavaScript that your Node.js version supports.

Since JavaScript moves so fast, but browsers can be a bit slow to upgrade, sometimes on the web you are stuck with using older JavaScript / ECMAScript releases.

You can use Babel to transform your code to be ES5-compatible before shipping it to the browser, but in Node.js, you won't need that.

Another difference is that Node.js uses the CommonJS module system, while in the browser we are starting to see the ES Modules standard being implemented.

In practice, this means that for the time being you use require() in Node.js and import in the browser.

Here's the canonical example:

```js
const http = require("http");

const hostname = "127.0.0.1";
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader("Content-Type", "text/plain");
  res.end("Hello World\n");
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

## CommonJS module system

Node uses the CommonJS module system.

`const http = require('http');` is the syntax for importing in node applications. It is different from the ES6 module system we have been using in React, e.g.:

`import Header from './Header'`.

CommonJS uses a `require()` function to fetch dependencies and an `exports` or `module.exports` variable to export. CommonJS is an earlier system and was not intended for browsers where ES6 modules are used instead.

An example. `fetch` is a browser API. If you want to use fetch in node you would have to npm install it and then import or require it using:

`const fetch = require('node-fetch');`
