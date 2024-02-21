# Introduction to Express Request

So far, you've been making simple `requests` and getting simple responses.

Let's look at a weather site:

[Dark Sky](https://darksky.net/forecast/40.7127,-74.0059/us12/en)

![](./assets/dark-sky.png)

What you will notice is that the URL is something like:

https://darksky.net/forecast/40.7127,-74.0059/us12/en?units=metric&date=05312021

Below marks the different parts of the URL:

```
 http://darksky.net/forcast/40.7127,-74.0059/us1/en?units=metric&date=05312021

 \___/ \_________/ \____________________________/\_________________________/
 protocol host/domain path query-string
```

How would you build this app? Are there developers building a route for every single latitude and longitude? That would seem like it would require a lot of developers to work around the clock, and the code would not be very DRY.

Instead, you can use the URL to pass in values and programmatically design web applications.

To the users, the URL looks the same. But for the server code, you would write the route as something like:

`http://darksky.net/forcast/:location/:timezone/:language`

In the actual request, you would be able to get an object that looks like this that comes from the request parameters:

```js
{
 location: "0.7127,-74.0059",
 timezone: "us1",
 language: "en",
}
```

And you would also be able to get a second object that comes from the query strings (key-value pairs that come after the question mark at the end of the URL), which would look like this:

```js
{
 units: "metric",
 date: "05312021",
}
```

Then you could use the values to get the data needed to complete the response.

## Learning Objectives

By the end of this lesson, you should be able to:

- Begin organizing your application into separate files with different roles for long-term maintainability
- Describe what a resource is and how it relates to internet applications.
- Make requests with query parameters and URL queries to a locally running server via the browser.
- Identify common errors that occur in building Express routes.
- Common error: Two responses
- Common error: Place routes in the correct order
- Access request parameters in an Express route.
- Access query parameters in an Express route.
- Access important data from the request object.

---

## JavaScript Environments

There are many places (environments) to run JavaScript, for example, in the browser or terminal. Each place is a different environment: Your computer is a different environment than your friend's computer. You both have different usernames, passwords, and other configurations. Therefore, adding a file that stores specific information about running the code in a particular environment is a crucial feature.

Make an environmental variable file and call it `.env`. Git should not track it, so it is added to the file in the project's `.gitignore`. This will allow each environment to keep its own file, and if there is sensitive information like passwords or API keys, they will also not be shared on GitHub. Remember, `.gitignore` is not a JavaScript file, so there is no need for quotes or semi-colons.

Example `.gitignore`:

```
node_modules
.env
```

To use environmental variables, you'll add a new npm package [dotenv](https://www.npmjs.com/package/dotenv)

- `touch .env`
- `npm install dotenv`

- Open `.env` add:

```
PORT=3333
```

> **Note**: The `.env` file is also a JavaScript file, so don't add semi-colons, quotes, or extra white space.

When you want to access the values, you will access them through the `process.env` object (after requiring and configuring `dotenv`, see below):

```js
// The whole process.env object
console.log(process.env)

// Accessing the value of PORT from the .env file
console.log(process.env.PORT)
```

> **Note**: A variable in all caps represents a constant in other coding languages. When you deploy your application, there is usually an expectation that the port is set to a variable name of `PORT`. Therefore, you should begin using this convention now to be able to deploy with ease later.

## Separating concerns

Separating concerns through moving functionality to certain files and folders in your server is common and best practice. Remember how you used `helper.js` to abstract helper functions. Then imported them into the files where they would be invoked?

To begin separating concerns you will set up the server in one file and the routes and other logic in other files. The following code examples will begin to demonstrate how to do this.

## Getting started

You make read or code along. To code along, you can get started with the steps below.

- make a new directory called `colors-backend` and `cd` into it
- `touch .gitignore .env`
- `npm init -y`
- `npm i express dotenv`
- `touch app.js server.js`

### Set Up app.js

```js
// DEPENDENCIES
const express = require('express')

// CONFIGURATION
const app = express()

// ROUTES
app.get('/', (req, res) => {
  res.send('Welcome to my Express app')
})

// EXPORT
module.exports = app
```

### Set up the server.js

```js
// DEPENDENCIES
const app = require('./app.js')

// CONFIGURATION
require('dotenv').config()
const PORT = process.env.PORT

// LISTEN
app.listen(PORT, () => {
  console.log(`Listening on port ${PORT}`)
})
```

Get the app running with `nodemon server.js` and go to http://localhost:3333

## Resources

A server's purpose is to maintain resources. For example, if you have a website with bicycles and unicycles for sale, each would be a separate resource.

The resources are a combination of URLs, data models, and data.

## Mocking data

Right now, you will be focused on learning Express. Later you'll learn about databases and how to integrate a database into an Express app.

In the meantime, you will mock a database by just using arrays of objects that will live in memory while the server is running.

The shape of the data (the key/value pairs and how they are organized(flat, nested)) is referred to as a data model.

Data modeling is a technique for defining business requirements for a database. Because of this, you will follow the naming convention and put your mock data in a folder called `models`. For now, you can make a simple array of colors.

Within this application, there will only be one data model or resource. However, there will be tens or hundreds in a professional application.

Remember, you need to leave your server running. To open a new Terminal tab, you can press <kbd>command</kbd> <kbd>t</kbd>

- Confirm you are in the same directory as `package.json` before typing the following commands:
- `mkdir models`
- `touch models/color.js`

```js
module.exports = [
  'MistyRose',
  'Coral',
  'Goldenrod',
  'MediumAquamarine',
  'DeepSkyBlue',
  'Violet',
]
```

Then the data must be exported with `module.exports` so it can import it elsewhere in your app.

> **Note**:if you forget to use `module.exports` or misspell it
> as `module.export`, you will get an empty object. Be sure to check this carefully, as it is a common bug.

## Integrating data into a route

When a request is made to a route that requires data, the data will be retrieved and then returned in some format. For now, it will be sent as simple strings. Later, you can send HTML or JSON.

## A route for a list of all colors

You can send this mock data to a client based on a request. Typically a list of a particular resource is referred to as an `index` or `index route`.

The URL, data model, and data make up a server resource. [For more in-depth information on best practices for a resource, there is a good summary here.](https://www.ibm.com/docs/en/sva/9.0.7?topic=control-resources).

To access the data, you can import the colors array into `app.js`.

```js
// app.js
// DEPENDENCIES
const express = require('express')
const colors = require('./models/color.js')

// CONFIGURATION
const app = express()

// ROUTES
app.get('/', (req, res) => {
  res.send('Welcome to My Express App')
})

// Index route
app.get('/colors', (req, res) => {
  res.json({ colors })
})

// EXPORT
module.exports = app
```

http://localhost:3333/colors

## A route for one color

There are six colors available. You could create a route for each color. Six is very doable. But what if this store is a huge success, and you have hundreds or thousands of colors?

Writing a route for each one would be:

- Tedious
- Hard to maintain

Instead, you want user input for the color the user wants to see. You'll now use the array position and have the user type in the URL. Eventually, you would create a complete web page with links and id numbers for your colors. These routes are _dynamic_ because they can change based on inputs. This contrasts with _static_ routes, which always stay the same (like a contact page or about page).

Create a request parameter by adding a `:` to distinguish it from a regular, static path.

Whatever value the user types will then be sent with the request and is accessible inside the `request.params` object.

```js
// ROUTES
app.get('/colors/:index', (req, res) => {
  console.log(req.params.index)
})
```

You can now use this value to access the item in the array's position 1.

http://localhost:3333/colors/1

```js
// ROUTES
app.get('/colors/:index', (req, res) => {
  res.json({ color: colors[req.params.index] })
})
```

You can make this code easier to read by using object destructuring.

```js
// ROUTES

// Show one color
app.get('/colors/:index', (req, res) => {
  const { index } = req.params
  res.json({ color: colors[index] })
})
```

## A common error

You can only have one response for every request: This is a rule of the http protocol. You'll get an error in the terminal if you try to send multiple responses.

```js
// A route with an error
app.get('/colors/oops/:index', (req, res) => {
  const { index } = req.params
  res.json({ color: colors[index] })
  // error cannot send more than one response!
  res.json({ message: `This is the index: ${index}` })
})
```

You can, however, have multiple statements if you use `if` statements or other program logic correctly:

```js
// Show one color, with error handling
app.get('/colors/:index', (req, res) => {
  const { index } = req.params
  if (colors[index]) {
    res.json({ color: colors[index] })
  } else {
    res.json({ message: `Cannot find any colors at this index: ${index}` })
  }
})
```

http://localhost:3333/colors/98

or

http://localhost:3333/colors/not_a_valid_index_position

## Place routes in the correct order

Express starts at the top of your `app.js` file and attempts to match the URL used by the browser with routes in the order in which they're defined.

URL params (e.g., `:foo`, `:example`, `like `) can be anything, a number or a string

Therefore, if you have these routes in this order in your `server.js`:

- `/colors/:id`
- `/colors/cool`

If you want to get to `/colors/cool` - you'll always hit the `/colors/:id` route because the URL parameter will accept _any_ string. It doesn't know that `colors` is something specific to look for.

- To fix this, put the more specific routes first
- `/colors/cool`
- `/colors/:id`
  Now, from top to bottom, the more specific route `/colors/cool` will be triggered when the URL has `/colors`; if it doesn't match `/colors/:id`, it will go to the following route.

Here is a code example, building on the app you have so far:

```javascript
// Show one color
app.get('/colors/:index', (req, res) => {
  const { index } = req.params
  if (colors[index]) {
    res.json({ color: colors[index] })
  } else {
    res.json({ message: `Cannot find any colors at this index: ${index}` })
  }
})

// This will never be reached
app.get('/colors/cool', (req, res) => {
  const coolColors = [colors[0], colors[1], colors[2]]

  res.json({ colors: coolColors })
})
```

When you have this situation, reorder the routes so that the more specific routes come before less particular routes (those with params in them).

## Multiple Parameters

We can add more parameters to the `req.params` object:

```js
app.get('/hello/:user/:food', (req, res) => {
  console.log(req.params)
  const { user, food } = req.params
  res.send(`My name is ${user} and my favorite food is ${food}`)
})
```

http://localhost:3333/hello/neo/pizza

Please try other users and foods.

## Query Strings

Another way to get values from the URL is with query strings.

Query strings go at the end of a path and start with a `?`. They are key-value pairs with the syntax of `value=100`.

You can add as many as you like by separating them with an ampersand `&`. You don't need to create a new route for them. The data can still be passed into the base route.

```js
app.get('/calculator/:operator', (req, res) => {
  console.log('This is req.params', req.params)
  console.log('This is req.query', req.query)
  const sum = req.query.num1 + req.query.num2
  res.send('sum is ' + sum)
})
```

http://localhost:3333/calculator/add?num1=5&num2=4

Uh oh! The sum is 54 instead of 9. Remember, incoming requests always come in as strings. Therefore they must be converted to numbers.

```js
app.get('/calculator/:operator', (req, res) => {
  const { num1, num2 } = req.query
  const sum = Number(num1) + Number(num2)
  res.send('sum is ' + sum)
})
```

Let's add a bit more logic.

```js
app.get('/calculator/:operator', (req, res) => {
  const { num1, num2 } = req.query
  let sum = 0
  if (req.params.operator === 'add') {
    sum = Number(num1) + Number(num2)
  }
  res.json({ sum })
})
```

**Bonus**: Can you figure out how to add the functionality of `subtract`, `multiple`, and `divide`?
