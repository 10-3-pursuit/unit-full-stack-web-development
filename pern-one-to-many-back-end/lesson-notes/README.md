## Bookmarks and Reviews

**One** bookmark can have **many** reviews.

How did we set up this relationship in SQL?

<details><summary>Answer</summary>

We added foreign keys (the primary key `id` of the `one` to the `many`).

</details>

## Create Full CRUD with a New Model Reviews

### Getting started

#### Option 1: Continue with your previous build

- Open your `class-db-bookmarks-backend` directory and make sure it is working.
- Maybe test routes in Postman to be sure.

#### Option 2: Clone down some starter code

- `fork` [class-db-bookmarks-backend
  ](https://github.com/10-3-pursuit/class-db-bookmarks-backend)
- `npm install`
- Update the `.env` to work on your machine based on the .env.example file
- `npm run db:init`
- `npm run db:seed`
- `npm run dev`

### Continue

Let's add a new table for reviews. We want to add a couple of constraints.

- We only want reviews to be between 0 and 5.
- We want to add the `REFERENCES` constraint, meaning that reviews cannot be created without a bookmark `PRIMARY KEY` in the `bookmarks` table.
- We also want to delete all the reviews for a bookmark if the bookmark is deleted.

### SQL Part

> **Note**: This is already included if you have cloned down the starter code. You only need to run the command.

Let's set up our tables and insert some seed data.

```SQL
- db/schema.sql
DROP TABLE IF EXISTS reviews;

CREATE TABLE reviews (
 id SERIAL PRIMARY KEY,
 reviewer TEXT,
 title TEXT,
 content TEXT,
 rating NUMERIC,
 CHECK (rating >= 0 AND rating <= 5),
 bookmark_id INTEGER REFERENCES bookmarks (id)
 ON DELETE CASCADE
);
```

Run the following command:

```bash
npm run db:init
```

- Where does this command come from?
- What does this command do?

Insert some reviews. Note, since we are dropping the bookmarks table when we insert the bookmarks again into a new table, we should get bookmarks with `id` of 1, 2, and 3, guaranteed.

> **Note**: This code is already included if you have cloned down the starter code. You only need to run the command.

Copy and paste these:

```SQL
-- db/seed.sql
INSERT INTO reviews (bookmark_id, reviewer, title, content, rating )
VALUES
('1', 'Evan', 'My Favorite', 'This website crushes it when it comes to awesome explanations', 3),
('2', 'Evan', 'My Favorite', 'This website crushes it when it comes to inspiring me', 3),
('3', 'Evan', 'My Least Favorite', 'This website crushes it when it comes to destroying my patience', 5),
('2', 'Juliana', 'I Love Going Here', 'I finally learned how to properly fold a fitted sheet', 5),
('2', 'David', 'Cool Site', 'But I got way into adding decorative pillows everywhere', 1),
('2', 'Mr. Mingo', 'So Helpful', 'I got some awesome recommendations for a ceiling fan and some spoons', 3),
('2', 'Alison', 'A lifesaver!','Helped me get my stove cleaner than I ever imagiend possible!', 4),
('3', 'Hannah', 'Insert Confetti Emoji Here', 'I survived 6 hours at the DMV!', 4),
('3', 'Gabi', 'My Friend Hannah', 'Gets a discount if I leave a positive review, so here it is', 5);
```

Run the following command:

```bash
npm run db:seed
```

- What does this command do?
- Why is the `bookmark_id` included in this seed data?

### Express/PG-promise

Let's create the RESTful routes for reviews and test them.

|     | Action  |     URL      | HTTP Verb |    CRUD    |               Description                |
| :-: | :-----: | :----------: | :-------: | :--------: | :--------------------------------------: |
|  1  | Create  |   /reviews   |   POST    | **C**reate |           Create a new review            |
|  2  |  Index  |   /reviews   |    GET    |  **R**ead  |   Get a list (or index) of all reviews   |
|  3  |  Show   | /reviews/:id |    GET    |  **R**ead  | Get an individual view (show one review) |
|  4  | Update  | /reviews/:id |    PUT    | **U**pdate |             Update a review              |
|  5  | Destroy | /reviews/:id |  DELETE   | **D**elete |             Delete a review              |

#### Continue with your app

> **Note**: This is already included if you have cloned down the starter code.

- `touch controllers/reviewsController.js` and copy the code from the [following code](https://raw.githubusercontent.com/pursuit-curriculum-resources/express-sql-seed-read-demo/starter-one-to-many/controllers/reviewsController.js).
- `touch queries/reviews.js` and copy the code from the [following code](https://raw.githubusercontent.com/pursuit-curriculum-resources/express-sql-seed-read-demo/starter-one-to-many/queries/reviews.js).

Don't forget to add this to your `app.js`.

```js
// app.js
// Reviews ROUTES
const reviewsController = require('./controllers/reviewsController.js')
app.use('/reviews', reviewsController)
```

### Test All Your routes

Here is a sample object:

```js
{
 "reviewer":"Lou",
 "title": "Fryin Better",
 "content": "With the great tips and tricks I found here",
 "bookmark_id": "1",
 "rating": "4"
}
```

> **Note**: For now, you will add the `bookmark_id` as part of the object because the routes are not nested yet. Once you have completed nesting the routes, the `bookmark_id` will come from the URL.

- Get index for reviews.
- Get show for a review.
- Post a new review to `/reviews`.
- Update that review.
- Delete that review.

Be sure to git, add, and commit if you continue from your code.

We now have two models with full CRUD. But how do we connect them?

## Nesting Reviews Resource

We need to do two things to use `/bookmarks/:id/reviews` as a base route.

- Pass an option to `express.Router()` to merge parameters from the bookmarks and reviews route
- Import the reviews controller in the bookmarks controller

> **Note**: If you have cloned down the starter code, this is where you will start adding new code.

```js
// controllers/reviewController.js
const reviews = express.Router({ mergeParams: true })
```

```js
// controllers/bookmarkController.js
const reviewsController = require('./reviewsController.js')
bookmarks.use('/:bookmark_id/reviews', reviewsController)
```

Now we can go to http://localhost:3003/bookmarks/1/reviews

We've made progress! However, instead of seeing the reviews for the bookmark with the `id` of 1, we see all the reviews.

Update the code by passing the parameter to the query and then updating the query to use the value.

```js
// controllers/reviewsController.js
// INDEX
reviews.get('/', async (req, res) => {
  const { bookmark_id } = req.params
  try {
    const allReviews = await getAllReviews(bookmark_id)
    res.json(allReviews)
  } catch (err) {
    res.json(err)
  }
})
```

```js
// queries/reviews
const getAllReviews = async (bookmark_id) => {
  try {
    const allReviews = await db.any(
      'SELECT * FROM reviews WHERE bookmark_id=$1',
      bookmark_id
    )
    return allReviews
  } catch (err) {
    return err
  }
}
```

It would be useful to see the bookmark with the review.

Import the query `getBookmark` to the reviews controller.

```js
const { getBookmark } = require('../queries/bookmarks.js')
```

Call it and make a new nested object to send back.

```js
// controllers/reviewsController.js
// INDEX
reviews.get('/', async (req, res) => {
  const { bookmark_id } = req.params
  const reviews = await getAllReviews(bookmark_id)
  const bookmark = await getBookmark(bookmark_id)
  if (bookmark.id) {
    res.status(200).json({ ...bookmark, reviews })
  } else {
    res.status(500).json({ error: 'Bookmark not found or server error' })
  }
})
```

## Cleaning up the Code

In this app, there is no need ever to see reviews without the bookmark. So you can remove those routes. Remember, this is only true for this application. Always consider the features you need and design your routes accordingly.

```js
// app.js
// REMOVE THIS CODE
const reviewsController = require('./controllers/reviewsController.js')
app.use('/reviews', reviewsController)
```

## Test the nested routes

### Show

Test a show route like http://localhost:3003/bookmarks/1/reviews/1

It should show you the review. But since this is a nested route, you would expect the bookmark details to be included rather than just the review. You can update the show route to include the bookmark data.

Use the `getBookmark()` function and create a new object with both the bookmark and review:

```js
// controllers/reviewsController.js
// SHOW
reviews.get('/:id', async (req, res) => {
  const { bookmark_id, id } = req.params
  const review = await getReview(id)
  const bookmark = await getBookmark(bookmark_id)
  if (review) {
    res.json({ ...bookmark, review })
  } else {
    res.status(404).json({ error: 'not found' })
  }
})
```

- Could you write this code as one database call?
- When is it essential to optimize code?

### Create

Create a new review with Postman to `/bookmarks/1/reviews`.

```json
{
  "reviewer": "Hadly",
  "title": "Life changing!",
  "content": "Wow! MDN is such a great resource. I can feel my coding skills get better every time I visit!",
  "rating": "5"
}
```

The `bookmark_id` is null. Update the `/reviews` POST route:

```js
// controllers/reviewsController.js
reviews.post('/', async (req, res) => {
  const { bookmark_id } = req.params
  const review = await newReview({ bookmark_id, ...req.body })
  res.status(200).json(review)
})
```

Update the new bookmark (include a review's id in the URL and a bookmark_id). It should be something like `/bookmarks/1/reviews/11`. Below is a sample object to test:

```json
{
  "reviewer": "Hadly",
  "title": "Upon further reflection",
  "content": "It's ok",
  "rating": 4
}
```

### Update

Make the updates to accept the correct arguments so that the ids come from the URL.

```js
// controllers/reviewsController.js
// UPDATE
reviews.put('/:id', async (req, res) => {
  const { id, bookmark_id } = req.params
  console.log(id, req.params.bookmark_id)
  const updatedReview = await updateReview({ bookmark_id, id, ...req.body })
  if (updatedReview.id) {
    res.status(200).json(updatedReview)
  } else {
    res.status(404).json('Review not found')
  }
})
```

Update the query to take one argument

```js
const updateReview = async (review) => {
  try {
    const updatedReview = await db.one(
      'UPDATE reviews SET reviewer=$1, title=$2, content=$3, rating=$4, bookmark_id=$5 where id=$6 RETURNING *',
      [
        review.reviewer,
        review.title,
        review.content,
        review.rating,
        review.bookmark_id,
        review.id,
      ]
    )
    return updatedReview
  } catch (error) {
    return error
  }
}
```

### Delete

Finally, test the delete route. It should work without needing to make more updates to the starter code. Why? Do you need the bookmark id to delete the bookmark?
