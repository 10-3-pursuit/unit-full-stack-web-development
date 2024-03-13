# PERN Stack: CRUD

## Getting Started

This build will review how to make:

- index route
- show route
- delete route

The new and edit views are already set up for you.

- Go to your bookmarks backend app and get it started. If you do not have a working copy you can `fork` and `clone` this one: [class-db-bookmarks-backend](https://github.com/10-3-pursuit/class-db-bookmarks-backend)
- Open a new tab in the terminal, do not shut down your bookmarks backend app.
- Navigate into your parent directory for the bookmarks app. Confirm you are not already inside a git repository.
- Fork the [class-db-bookmarks-frontend](https://github.com/10-3-pursuit/class-db-bookmarks-frontend).
- `git clone` the forked repository inside your parent folder.
- `npm install` to install dependencies already included in the `package.json`.
- `touch .env`

**.env**

```
VITE_BASE_URL=http://localhost:3003
```

- `git add -A`
- `git commit -m 'add front end starter code'`.
- `npm start`

## Index

<details><summary>Landing Page</summary>

![](../assets/landing-page.png)

</details>

Let's click on `Bookmarks` in the navigation to go to `/bookmarks`.

<details><summary>Index Page Empty</summary>

![](../assets/index-empty.png)

</details>

We can see that we have a view for our index page. But we need to connect this app to our backend. We can do so using the fetch API much like we did using third-party APIs.

## Setting Up the App to Make Requests

### Loading the Bookmarks Index on Page Load

#### A Small Note

Since the initial build, the team has reviewed the Bookmarks app. They had made several suggestions to improve the code. Here you'll note some code reorganization and updates to make the code easier to read and more maintainable.

Can you spot the differences? Can you talk about why you think the changes have improved the app?

### Building the Index Page Functionality

```js
// src/Components/Bookmarks.js

useEffect(() => {
  fetch(`${API}/bookmarks`)
}, [])
```

- Now we can make our API call, but `then`, what should happen?

Work from the outside in. Build the outer functions first.

```js
useEffect(() => {
  fetch(`${API}/bookmarks/${id}`)
    .then(() => {})
    .then(() => {})
    .catch(() => {})
}, [])
```

- What does the first function do?
- What does the first `.then()` function handle?
- What does the second `.then()` function handle?
- What does `.catch()` handle?
- Why are there two `.then()` functions? Why can't the code be combined into one?

Add the functionality in the right places:

```js
useEffect(() => {
  fetch(`${API}/bookmarks`)
    .then((response) => {
      return response.json()
    })
    .then((responseJSON) => {
      setBookmarks(responseJSON)
    })
    .catch((error) => console.error(error))
}, [])
```

## Show

Let's build out the show page.

![](assets/show-loaded.png)

First, let's start with the JSX, then we'll go back and add functionality. This is the recommended approach, based on the docs: [Thinking in React](https://react.dev/learn/thinking-in-react)

We will hard-code values to see our components and make sure things are set up and organized as we want. Then, we'll go back and add more functionality.

```js
// src/Components/BookmarkDetails.js
import { useState } from 'react'

function BookmarkDetails() {
  const [bookmark, setBookmark] = useState([])
  return (
    <article>
      <h3>{true ? <span>⭐️</span> : null} bookmark.name</h3>
    </article>
  )
}
```

Let's add the link for the actual bookmark page.

```js
function BookmarkDetails() {
  const [bookmark, setBookmark] = useState([])
  return (
    <article>
      <h3>{true ? <span>⭐️</span> : null} bookmark.name</h3>
      <h5>
        <span>
          <a href={bookmark.url}>bookmark.name</a>
        </span>
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; bookmark.url
      </h5>
      <h6>bookmark.category</h6>
      <p>bookmark.description</p>
    </article>
  )
}
```

- What is `&nbsp;`?

Let's add the buttons for edit, delete, and back.

```js
import { Link } from 'react-router-dom'

// Further down inside the component

return (
  <article>
    <h3>{true ? <span>⭐️</span> : null} bookmark.name</h3>
    <h5>
      <span>
        <a href={bookmark.url}>bookmark.name</a>
      </span>
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; bookmark.url
    </h5>
    <h6>bookmark.category</h6>
    <p>bookmark.description</p>
    <div className="showNavigation">
      <div>
        <Link to={`/bookmarks`}>
          <button>Back</button>
        </Link>
      </div>
      <div>
        <Link to={`/bookmarks/id/edit`}>
          <button>Edit</button>
        </Link>
      </div>
      <div>
        <button>Delete</button>
      </div>
    </div>
  </article>
)
```

## Adding Functionality

We want to load the bookmark details on page load.

- How can we load the bookmark details on page load?
- Are there any dependencies needed?
- Is the second argument of `useEffect()` (dependency array), necessary? What happens if you don't include the second argument?

```js
import { useState, useEffect } from "react";

// inside the BookmarkDetails function....

 useEffect(() => {
},[]);
 return <article>...</article>;
});

```

We will also require more information from the user to make the API call. We need to know the `id` of the bookmark for the show view. The `id` of the bookmark will come from the URL.

```js
import { useState, useEffect } from "react";
import { Link, useParams, useNavigate } from "react-router-dom";
const API = import.meta.env.VITE_BASE_URL;

// inside the BookmarkDetails function....
function BookmarkDetails() {
 const [bookmark, setBookmark] = useState([]);
 let { id } = useParams();

 useEffect(() => {
 fetch(`${API}/bookmarks/${id}`)
 .then((response) => response.json())
 .then((responseJSON) => setBookmark(responseJSON))
 .catch((error) => console.error(error));
 }, [id, API]);
```

Now that it works as expected let's go in and add the actual values where we've put our placeholders.

```js
return (
  <article>
    <h3>
      {bookmark.is_favorite ? <span>⭐️</span> : null} {bookmark.name}
    </h3>
    <h5>
      <span>
        <a href={bookmark.url}>{bookmark.name}</a>
      </span>{' '}
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
      {bookmark.url}
    </h5>
    <h6>{bookmark.category}</h6>
    <p>{bookmark.description}</p>
    <div className="showNavigation">
      <div>
        {' '}
        <Link to={`/bookmarks`}>
          <button>Back</button>
        </Link>
      </div>
      <div>
        <Link to={`/bookmarks/${id}/edit`}>
          <button>Edit</button>
        </Link>
      </div>
      <div>
        <button>Delete</button>
      </div>
    </div>
  </article>
)
```

## Delete Functionality

From this view, we want to be able to delete a bookmark. It's common to name a function that handles an event (like a click or hover `handle`).

- What are the pros of creating a generic `handleDelete` function?
- What are the pros of just having the onClick event call the delete functionality?

```js
// src/boookmarkDetails.jsx
const handleDelete = () => {
  console.log('button clicked')
}
```

Let's add a click event.

```js
<div>
  <button onClick={handleDelete}>Delete</button>
</div>
```

Click the button to check that your click handler works as expected.

We will use another fetch call to request the database to delete the bookmark.

When the deletion is completed on the backend, we can think about the user experience. A reasonable way to handle it is to take the user back to the Index page, so they can see that their bookmark has been successfully deleted.

We can do this by using `useNavigate` from React Router.

```js
// Top of file
import { Link, useParams, useNavigate } from 'react-router-dom'

let navigate = useNavigate()

// Inside BookmarkDetails function
const deleteBookmark = () => {
  fetch(`${API}/bookmarks/${id}`, {
    method: 'DELETE',
  })
    .then(() => navigate(`/bookmarks`))
    .catch((error) => console.error(error))
}
```

Finally, let's call this function:

```js
const handleDelete = () => {
  deleteBookmark()
}
```

## Reference Build

[The build](https://github.com/pursuit-curriculum-resources/starter-pern-crud/tree/solution).
