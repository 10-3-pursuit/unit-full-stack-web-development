# Express RESTful Routes: Show and Create, and Middleware

## Introduction

So far, you've learned about the MVC pattern for organizing and maintaining yur code. You'll keep building your app with a loose adaptation of this pattern.

You may encounter other code libraries or frameworks like Ruby on Rails(RoR). Ruby on Rails follows a principle of convention over configuration - meaning if you follow the rules of RoR, things will work _automagically_. Express is about thinking about your app needs and organizing it sensibly on your own. There is no one true way to organize an Express app, and you have to configure each step. The configuration gives you more control over how to build your app.

To fully design your back-end, it's important to learn critical concepts and successful patterns to help you build robust back-ends.

## Learning objectives

By the end of this lesson, you should be able to:

- Identify the five common RESTful routes that are associated with each resource.
- Identify the four common actions that make up the CRUD acronym.
- Identify the naming conventions for the five common RESTful routes.
- Build a show route
- Build a create route
- Describe how sending a response changes the code flow of servers.
- Describe the function and purpose of middleware in Express.
- Describe how the `next()` function changes the code flow of servers.
- Differentiate between application and route-level middleware.

---

## Resource Design

Imagine you are building a website that lets people select an artist to custom-make something like a table, sculpture, or embroidered family portrait.

What kind of data do you need? What would be the data model?

How would you organize the routes? Would you do it by artist? By item type? By craft type?

How would you name your routes so that a user could select an item or artist, then select the customization, then pay and then be able to see updates?

How could artists manage their services? They would need to be able to:

- Create an account
- See their account information
- Update their account to show what products they are available to make
- Delete some products they no longer make

This can be pretty challenging to design on your own. Therefore there are some standard patterns you can follow to ensure you are creating:

- Stable routes (don't need to change as the app grows more complex)
- Organized routes (easy to find and maintain)
- Simple patterns(unnecessary complications are removed)
- Easily reorganized code (design changes to the app require minor refactoring over a complete rewrite)
- Understandable by creators and users.

## C.R.U.D. and methods

So far, in your server builds, you have only been making requests to read data. Recall that there are four main things to do with data: **C**reate, **R**ead, **U**pdate, and **D**elete. Each of the four actions has an associated HTTP verb (also can be referred to as a method):

- `GET` - request to read data
- `POST` - request to create data
- `PUT` - request to update data
- `DELETE` - request to destroy data

How can you create and organize routes that follow the principles to keep the routes stable, organized, simple, easy to reorganize, and understandable by creators and users?

## RESTful Routes

The organization of server resources to abide by the above principles was so challenging that a computer scientist named Roy Fielding ended up doing his PhD dissertation on [Architectural Styles and the Design of Network-based Software Architectures](https://en.wikipedia.org/wiki/Representational_state_transfer) to create a system that can be broadly used. He took feedback from over 500 developers to hone a design to a core set of principles called REST.

REST stands for **Re**presentational **S**tate **T**ransfer - the technical meaning can take a while to study and learn and gets much deeper than you need to worry about today. However, you can effectively utilize the pattern of routes to build basic apps that use these best practices right away.

To build a back-end Express app that adheres to RESTful routes, there are five basic actions with specific URLs and HTTP verbs that should be built with the following pattern.

|  #  | Action  |     URL     | HTTP Verb |    CRUD    |               Description               |
| :-: | :-----: | :---------: | :-------: | :--------: | :-------------------------------------: |
|  1  | Create  |   /colors   |   POST    | **C**reate |           Create a new color            |
|  2  |  Index  |   /colors   |    GET    |  **R**ead  |   Get a list (or index) of all colors   |
|  3  |  Show   | /colors/:id |    GET    |  **R**ead  | Get an individual view (show one color) |
|  4  | Update  | /colors/:id |    PUT    | **U**pdate |             Update a color              |
|  5  | Destroy | /colors/:id |  DELETE   | **D**elete |             Delete a color              |

> **Note**: This pattern of routes is similar across many technical stacks. If you ever need a refresher, you can easily google it and find a resource [like this one](https://guides.rubyonrails.org/routing.html#crud-verbs-and-actions).

## Show

You've already learned about the index route, which shows a list of items. The show route, in contrast, will show one item. Typically, an index route has limited information about the items on display (for example, it would display a name, a price, and an image), and a show route would show many more details like what sizes and colors are available, reviews, and more.

In this example, the show route uses the index position of the array. When you connect a database, this value will be replaced with a unique id.

| Action |     URL     | HTTP Verb |   CRUD   |               Description               |
| :----: | :---------: | :-------: | :------: | :-------------------------------------: |
|  Show  | /colors/:id |    GET    | **R**ead | Get an individual view (show one color) |

```js
// SHOW
colors.get('/:arrayIndex', (req, res) => {
  const { arrayIndex } = req.params
  res.json(colorsArray[arrayIndex])
})
```

You can improve the user experience by adding some error handling.

```js
// SHOW
colors.get('/:arrayIndex', (req, res) => {
  const { arrayIndex } = req.params
  if (colorsArray[arrayIndex]) {
    res.json(colorsArray[arrayIndex])
  } else {
    res.status(404).json({ error: 'Not Found' })
  }
})
```

What is the URL to access this route?

http://localhost:3333/colors/0

or

http://localhost:3333/colors/1

etc.

## Create

| Action |   URL   | HTTP Verb |    CRUD    |    Description     |
| :----: | :-----: | :-------: | :--------: | :----------------: |
| Create | /colors |   POST    | **C**reate | Create a new color |

Creating new data will use a new HTTP verb: POST. However, the path name in the URL will stay the same as for the index route. Different HTTP verbs create different routes. You can manipulate the data using different routes, here you will push the new data into the array.

You can choose to either show the entire updated array or show only the new item. The response can change depending on the goal of your application.

```js
// CREATE
colors.post('/', (req, res) => {
  colorsArray.push(req.body)
  res.json(colorsArray[colorsArray.length - 1])
})
```

[You can read more about the application method POST in the documentation](https://expressjs.com/en/4x/api.html#app.post.method).

## Making POST requests and cURL

Now that you've created a POST route, how can you access it? The browser URL only lets you make GET requests.

You have a few options:

- Build an `index.html` form, load it into your browser, and test it.
- Download and use a program like Postman.
- Use a command-line program like cURL.

Building an HTML form is time-consuming, and you may not need it later, depending on the application you are building. For example, if a user `creates a new order` (buying something online) and clicks buttons like `add to cart` and `buy now`, their name, address, and payment information are likely captured at a different time - no forms are needed.

Postman (or similar programs) is a GUI program to assist with making HTTP requests can be an excellent tool. But it can have a bit of a learning curve and take some time to set up. Programs like Postman are more useful for continual application development than quickly testing something.

[cURL](https://curl.se) stands for `client URL` and has existed since 1998. It is a powerful library that is not only used on the command line but also can be integrated into cars, TVs, and other devices known as the Internet of Things.

Because it works on the command line, the UI is the same as any other command-line application. Once you know how to use the command line, learning a new program is much simpler and faster than learning a new GUI and it makes it an excellent option for quick demos and testing.

First, make sure your server is running and open a new terminal tab (<kbd>command</kbd> + <kbd>t</kbd>) and start by making a simple request. This will be a GET request to the index view:

- `curl http://localhost:3333/colors`

![curl GET](./assets/curl-get.png)

Next, you can make a POST request. You'll need to add some flags and arguments to do so.

- `-X` (HTTP verb) so you can choose POST (without this, the default is GET)
- `-d` (data) the request body data you want to send

Putting it all together, the command looks like this:

- `curl -X POST -d 'blanchedalmond' http://localhost:3333/colors`

> **Note**: You are likely not getting anything back, despite writing a code that should return the new item you sent. Try to debug it.

Ask: was anything pushed onto `colorsArray`?

- `curl http://localhost:3333/colors`

![pushed null to array](./assets/pushed-null-to-array.png)

Yes, but it was a null value.

Ask: what is the value of `req.body`? ( Remember, console logs will appear in the terminal tab of your server, and responses will be in the tab where you put your cURL command).

To test this, use the up arrow, and rerun the cURL command to POST.

```js
// CREATE
colors.post('/', (req, res) => {
  // What's the value of req.body?
  console.log('This is req.body', req.body)
  colorsArray.push(req.body)
  res.json(colorsArray[colorsArray.length - 1])
})
```

![req.body is undefined](./assets/req.body-is-undefined.png)

When you check [the documentation](https://expressjs.com/en/4x/api.html#req.body), you'll learn that, by default, `req.body` is undefined and populated only when you use body-parsing middleware.

## Body Parser

In the above example, you want to push the request body value, but what datatype would the request body have? Remember, HTTP is always sending strings. However, what is contained in those strings can vary.

For example, you can send HTML, JSON, no format, or form data. By default, HTML form data is [URL encoded](https://www.w3schools.com/tags/ref_urlencode.ASP#:~:text=URL%20encoding%20converts%20characters%20into,into%20a%20valid%20ASCII%20format.).

So how can you determine what format the data you are trying to send is and convert it into a JavaScript object that can be pushed into the array?

Firstly, you should include the data type you are sending when you request. So you'll need to update your cURL command by adding a flag `-H` for headers where you can specify the data type. By default, some header properties are always sent through. Adding the -H command will allow you to overwrite or add new values.

- `curl -H "Content-Type: application/json" -X POST -d 'blanchedalmond' http://localhost:3333/colors`

Next, update the data to be in proper JSON format. It must be an object, and each key and value must be wrapped in double quotes. Notice that the entire JSON object must be wrapped in single quotes.

- `curl -H "Content-Type: application/json" -X POST -d '{"name": "papayawhip"}' http://localhost:3333/colors`

Next, you must convert that incoming data into a usable JSON object. Express has built-in functionality to do this. In the next step, you'll set it up.

</details>

### Built-in middleware for body-parser

Express has built-in body-parsing functionality. To parse incoming JSON, you'll use
[express.json()](https://expressjs.com/en/4x/api.html#express.json).

Where should you put this functionality? Ideally, this function would happen after the request but before the response. Express allows you to create functionality in the middle of requests and responses, and these functions are called `middleware`.

To mount middleware functions in your Express app, you will use the Express function [`app.use()`](https://expressjs.com/en/guide/using-middleware.html).

Since you always want to parse any incoming JSON data, you should put this function inside `app.js` after configuration but **BEFORE** the routes. Remember, the flow of the server when adding and designing your middleware and routes.

```js
//app.js
// CONFIGURATION
const app = express()

// MIDDLEWARE
app.use(express.json()) // parse incoming middleware

// ROUTES
// Home Page
app.get('/', (req, res) => {
  res.send('Welcome to Colors App')
})
```

You should now be able to add a new color object using the following cURL command successfully.

- `curl -H "Content-Type: application/json" -X POST -d '{"name":"blanchedalmond"}' http://localhost:3333/colors`

You now have a new small issue: this new data not a simple string, unlike your model. Take a moment to update your model in `/models/color.js` to be an array of objects:

```js
// /models/color.js
module.exports = [
  { name: 'MistyRose' },
  { name: 'Coral' },
  { name: 'Goldenrod' },
  { name: 'MediumAquamarine' },
  { name: 'DeepSkyBlue' },
  { name: 'Violet' },
]
```

Now your old and new data will all be in the same format.

> **Note**: Test what happens when you move the middleware below the routes. What happens? Why does this happen? Testing this to learn the behavior is essential because it is a common bug. Understanding what is happening and why will improve your debugging skills.

```js
//app.js
// CONFIGURATION
const app = express()

// ROUTES
// Home Page
app.get('/', (req, res) => {
  res.send('Welcome to Colors App')
})

// body-parsing functionality is below the routes in this example
// MIDDLEWARE
app.use(express.json()) // parse incoming middleware
```
