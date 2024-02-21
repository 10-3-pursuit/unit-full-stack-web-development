# Express CRUD/MVC

## Introduction

Let's take a moment to think about a website like Etsy. Etsy has thousands of shops and millions of products.

When a shop owner decides to sell a new product, they enter the data about the latest product: name, price, image(s), description, etc. And almost immediately, a new web page appears!

How is this happening? Etsy doesn't have a bunch of devs sitting around at the ready to hand roll a new web page. Instead, the developers build an HTML template and then, based on the data provided **embed** (insert) the data into the HTML

Let's look at an example - imagine this is some data provided by an Etsy shop owner through an online form:

![](./assets/form.png)

When the owner presses the `submit` button. The browser will send a `request` to a server. The server will save the data in a database. We can imagine the data looks something like this:

**DEMO, do not code**

```js
{
 name: 'Bismuth',
 price: 20,
 image: 'bismuth.png',
 description: 'Very cool mineral',
 shopName: 'Rock Shop'
}

```

The server will `respond` with confirmation that the `request` went through successfully or will provide an error message.

Then, when the owner wants to see the product. The owner will make a `request` to see the product in the browser. There will be a `response` that will involve some logic to **embed** the data inside HTML:

**DEMO, do not code**

```html
<h1>{rock.shopName}</h1>
<div class="card">
  <img src="{rock.img}" alt="{rock.name}" />
  <h3><span>{rock.name}</span> {rock.price}</h3>
  <p>{rock.description}</p>
</div>
```

And then, it gets rendered in the browser:

![](./assets/bismuth-html.png)

There are four main things to do with data: **C**reate, **R**ead, **U**pdate, and **D**elete. In the above case, a user created some data and then read the newly created data. We can imagine, at some point, that the user may want to update the price or delete the item altogether.

First, you'll build a back-end application with Express with full CRUD actions on a single data model. Like the third-party APIs you've been working with, you'll send and get JSON. Later, you'll build a front-end application to consume the API.

The focus will be fundamentals first, and as your skills build, you will learn to create more complex apps with more real-world features.

## Learning Objectives

By the end of this lesson, you should be able to:

- Describe the purpose of the MVC architecture.
- Create a controller and a static model in an Express server.
- Use Express Router to manage Express routes.
- Add a custom status code.

## Software Architecture: MVC

The apps you are building in class are tiny. Even with small apps, getting lost in code can be very easy. Where did you create a particular function? What file is displaying what you see in the browser? Organizing and maintaining code is quite challenging!

Our problems are a microcosm of an app that is years old that has had tens or possibly hundreds of developers work on it. Therefore, developers who are far more experienced have worked on creating conventions, frameworks, styles, and patterns that are easy to maintain, collaborate, and reuse.

Many patterns and best practices will make more sense as you gain experience.

**Bonus:** [Read more on software architecture](https://en.wikipedia.org/wiki/Software_architecture).

In class, you will loosely follow a prevalent software design pattern called [MVC](https://en.wikipedia.org/wiki/Model–view–controller): Model-View-Controller.

Following this pattern allows developers to separate the concerns of the data, the view, and the controller.

A common metaphor for this is a restaurant. Imagine a customer sitting down and interacting with the menu (views). A chef in the back takes orders and makes food (models). Finally, a server goes between the two (controller).

Everyone in the restaurant has a clear idea of what their responsibility is. Otherwise, the restaurant likely would fail. Imagine if the customers could go in the back and start cooking or the servers stopped ringing up orders!

Even though there is a typical pattern for a restaurant, there are many variations, like buffets, drive-throughs, and more. A pattern is a recommendation, not a rule.

In terms of code, you will have a folder called `models` and a folder called `controllers`. While it is possible to have a `views` folder (and many applications use this folder) and build HTML templates, your applications will serve JSON, and a separate front-end app will be created to consume and create views for the API you've built.

## Views

Views are the front-end component and what the client sees and interacts with. For this application, you will only be sending data and later, build a separate application to create the views.

## Models

Models will contain the code that interacts with the database. For now, it will store arrays of objects. Later you'll learn to connect a database and build out the code to interact with the database.

## Controllers

The controllers "control" how the views (client app) and models (data) interact. For example, if you have a user log in, you would only want them to be able to edit their account information and you would also not want them to see someone else's account information.

Some other examples are that you may want your user to be able to create, read, update and delete their posts on a social media post. Or maybe, you are building an interface for a weather station. In which case you don't want to allow editing or deleting of the data that is incoming.

Controllers handle the logic for the application and are the go-between from the client to the database.

> **NOTE:** With Express, you can name your files and folders whatever you would like. If you want to call your `controllers` folder `asdf` and your `models` folder `batman` - you can. However, no one will understand how your code is organized with names that don't make sense/are not semantic.

Because Express is unopinionated, you will likely see various names for different files. For example, when it comes to controllers, you may notice:

- birdController.js
- birdsController.js
- birds.js
- birds-controller.js
- birds-controllers.js
- birds.controller.js

When you join a company, they will have chosen a naming strategy, which you should follow. Within your work your naming should be consistent within each project.

## Controllers in terms of code

You may read or code along. To code along, you can continue the build from the last pre-reading.

Each resource (users, colors, lamps, tickets) within an application should have its own models file and its own controllers file.

A controllers file will contain the routes specific for that resource.

- Confirm you are in the same director as `package.json` before typing the following commands:
- `mkdir controllers`
- `touch controllers/colorsController.js`

First one must import Express into the file in order to be able to use the router functionality needed in the controllers file.

Then the `router` must be configured. The router is what will allow you to create groups of associated routes that are relative to the base route.

You can name the variable for `express.Router()` `router`, or you can use the name of the resource. Using the resource name can be helpful in reading and understanding the code.

```js
const express = require('express')
const colors = express.Router()
const colorsArray = require('../models/color.js')

colors.get('/', (req, res) => {
  res.json(colorsArray)
})

module.exports = colors
```

Notice that the relative path is `/`. Because this is in a controller, you'll take steps to access this route not at the base route of `http://localhost:3000` (typically the landing or home page goes there) but rather at `http://localhost:/3000/colors`. To set this up, follow the steps below.

Next, the colors resource code would be imported into the main file `app.js`.

```js
// app.js
// ROUTES
const colorsController = require('./controllers/colorsController.js')
```

And then configured. This `app.use()` function takes in a string which will be the URL path for the resource, and then the routes are the second argument.

```js
app.use('/colors', colorsController)
```

You can now remove the routes in `app.js` that use `/colors` as you're rebuilding them in the `colorsController.js` file.

To put it all together, all of the routes written in the `colorsController` file will be accessed through a base url of

http://localhost:3003/colors

## Custom status codes

What happens on a website when you try to access a view that doesn't exist? Typically, you get a 404 error message.

You can create your own catchall route with express. To do this, you will use a `wildcard` symbol `*`, then you would follow with creating the view or message you would want to send to the user.

Like dynamic routes, it is important that this route goes below the other routes.

```js
// 404 Page not found
app.get('*', (req, res) => {
  res.json({ error: 'Page not found' })
})
```

When you open up Chrome Dev Tools and go to the `Network` tab, you can see that this file loads with an http status of [304](https://http.cat/304) or [200](https://http.cat/200)

But because this is a 404 [page not found](https://http.cat/404), you should make sure you are sending through the correct status code.

```js
// 404 Page not found
app.get('*', (req, res) => {
  res.status(404).json({ error: 'Page not found' })
})
```
