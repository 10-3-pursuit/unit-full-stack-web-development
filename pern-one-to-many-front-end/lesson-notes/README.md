# PERN Stack: One to Many - Front-end

## Important

Make sure all your routes work as expected. It does not matter how amazing your front-end code is; it cannot repair a broken back-end.

> **Note**: If needed, you can also clone down the completed back-end [here](https://github.com/10-3-pursuit/class-db-bookmarks-backend).

## Getting Started

### Option 1

Use the `class-db-bookmarks-frontend` repo you have already been working in.

### Option 2

- `git fork` the [class-db-bookmarks-frontend](https://github.com/10-3-pursuit/class-db-bookmarks-frontend) repo.
- `clone the repo into the parent folder that also contains `class-db-bookmarks-backend` repo.
- `cd` into the frontend repo.
- `npm install`
- `touch .env`
- Add `.env` variables (base URL for the back-end see `.env.template`)

## Adding the front-end

We will be adding views of reviews only to the `BookmarkDetails` view.

- Keep your back-end running
- Make sure you are at the root of your react-app (same level as its `package.json`)
- `touch src/Components/Review.jsx`
- `touch src/Components/Reviews.jsx`

## Reviews All

Create an index view of all reviews belonging to one bookmark.

```js
// src/Reviews.jsx
import { useState, useEffect } from 'react'
import { useParams } from 'react-router-dom'
import Review from './Review'

const API = import.meta.env.VITE_BASE_URL

function Reviews() {
  const [reviews, setReviews] = useState([])
  let { id } = useParams()

  useEffect(() => {
    fetch(`${API}/bookmarks/${id}/reviews`)
      .then((response) => response.json())
      .then((data) => setReviews(data.allReviews))
  }, [id])

  return (
    <section className="Reviews">
      {reviews.map((review) => (
        <Review key={review.id} review={review} />
      ))}
    </section>
  )
}

export default Reviews
```

Create a review component:

```js
// src/Review.js
function Review({ review }) {
  return (
    <div className="Review">
      <h4>
        {review.title} <span>{review.rating}</span>
      </h4>
      <h5>{review.reviewer}</h5>
      <p>{review.content}</p>
    </div>
  )
}

export default Review
```

Import `Reviews` into BookmarkDetails.

```js
// src/BookmarkDetails.jsx
import Reviews from './Reviews'
```

After the buttons and closing `div`s (right above the closing `article` tag):

```js
<Reviews />
```

<details><summary>Reviews Added to the bookmarks show route</summary>

![](../assets/reviews-read.png)

</details>

## Review Form

We will handle the form differently than we did with the bookmarks form. We will keep the new form on the same page as the bookmarks/reviews.

Additionally, we will reuse this form for new and editing bookmarks.

In the interest of time during the lecture, let's copy-paste this code and discuss what is going on.

`ReviewForm.jsx`

```js
import { useState, useEffect } from 'react'
import { useParams } from 'react-router-dom'

// purposely using the word props now so i can distinguish between handleEdit and handleAdd
function ReviewForm({
  reviewDetails,
  handleEdit,
  handleAdd,
  toggleView,
  children,
}) {
  const { id } = useParams()

  const [newOrUpdatedReview, setNewOrUpdatedReview] = useState({
    reviewer: '',
    title: '',
    content: '',
    rating: '',
    bookmark_id: id,
  })

  const handleTextChange = (event) => {
    setNewOrUpdatedReview({
      ...newOrUpdatedReview,
      [event.target.id]: event.target.value,
    })
  }

  useEffect(() => {
    if (reviewDetails) {
      setNewOrUpdatedReview(reviewDetails)
    }
  }, [id, reviewDetails])

  const handleSubmit = (event) => {
    event.preventDefault()
    //if there are reviewDetails, it means that we are editing, otherwise we are creating a new review.
    // here we are now using the actual function names instead of handleSubmit for both functions
    if (reviewDetails) {
      handleEdit(newOrUpdatedReview, id)
    } else {
      console.log(newOrUpdatedReview)
      handleAdd(newOrUpdatedReview)
    }

    //after i submit, toggle this view back to displaying the review
    if (reviewDetails) {
      toggleView()
    }
    setNewOrUpdatedReview({
      reviewer: '',
      title: '',
      content: '',
      rating: '',
      bookmark_id: id,
    })
  }
  return (
    <div className="Edit">
      {children}
      <form onSubmit={handleSubmit}>
        <label htmlFor="reviewer">Name:</label>
        <input
          id="reviewer"
          value={newOrUpdatedReview.reviewer}
          type="text"
          onChange={handleTextChange}
          placeholder="Your name"
          required
        />
        <label htmlFor="title">Title:</label>
        <input
          id="title"
          type="text"
          required
          value={newOrUpdatedReview.title}
          onChange={handleTextChange}
        />
        <label htmlFor="rating">Rating:</label>
        <input
          id="rating"
          type="number"
          name="rating"
          min="0"
          max="5"
          step="1"
          value={newOrUpdatedReview.rating}
          onChange={handleTextChange}
        />
        <label htmlFor="content">Review:</label>
        <textarea
          id="content"
          type="text"
          name="content"
          value={newOrUpdatedReview.content}
          placeholder="What do you think..."
          onChange={handleTextChange}
        />

        <br />

        <input type="submit" />
      </form>
    </div>
  )
}

export default ReviewForm
```

If we are editing a form, want to be able to pre-fill the data if it is the edit form. We can `useEffect()` to check if data is being passed down and update the form inputs.

If it is the new form, we want to display an extra `h3` to inform the users this is the form to create a new review. We will do this by using `children` to display this additional `h3`.

## Add new review functionality

Here are the new pieces of code. Place them in their appropriate locations(along the top, inside the function, inside the return).

```js
// Top of file
// src/Reviews.jsx
import ReviewForm from './ReviewForm'
```

```js
// add this inside the Reviews() function
const handleAdd = (newReview) => {
  fetch(`${API}/bookmarks/${id}/reviews`, {
    method: 'POST',
    body: JSON.stringify(newReview),
    headers: {
      'Content-Type': 'application/json',
    },
  })
    .then((response) => response.json())
    .then((data) => setReviews([data, ...reviews]))
    .catch((error) => console.error('catch', error))
}
```

```js
// replace the return statement with this code
return (
  <section className="Reviews">
    <h2>Reviews</h2>
    <ReviewForm handleAdd={handleAdd}>
      <h3>Add a New Review</h3>
    </ReviewForm>
    {reviews.map((review) => (
      <Review key={review.id} review={review} />
    ))}
  </section>
)
```

## Add Delete Functionality

```js
// src/Reviews.jsx
const handleDelete = (id) => {
  fetch(`${API}/bookmarks/${id}/reviews/${id}`, {
    method: 'DELETE',
  })
    .then(
      (response) => {
        const copyReviewArray = [...reviews]
        const indexDeletedReview = copyReviewArray.findIndex((review) => {
          return review.id === id
        })
        copyReviewArray.splice(indexDeletedReview, 1)
        setReviews(copyReviewArray)
      },
      (error) => console.error(error)
    )
    .catch((error) => console.warn('catch', error))
}
```

Add the delete method to the Review component.

```js
<Review key={review.id} review={review} handleDelete={handleDelete} />
```

Add `handleDelete` as a parameter and then add an `onClick` event on delete button in the `Review` component.

```js
// src/Review.js
function Review({ review, handleDelete }) {
  return (
    <div className="Review">
      <h4>
        {review.title} <span>{review.rating}</span>
      </h4>
      <h5>{review.reviewer}</h5>
      <p>{review.content}</p>
      <button onClick={() => handleDelete(review.id)}>delete</button>
    </div>
  )
}
```

## Add Edit Functionality

Add the API call to Reviews.jsx

```js
// src/Reviews.jsx
const handleEdit = (updatedReview) => {
  fetch(`${API}/bookmarks/${id}/reviews/${updatedReview.id}`, {
    method: 'PUT',
    body: JSON.stringify(updatedReview),
    headers: {
      'Content-Type': 'application/json',
    },
  })
    .then((response) => response.json())
    .then((responseJSON) => {
      const copyReviewArray = [...reviews]
      const indexUpdatedReview = copyReviewArray.findIndex((review) => {
        return review.id === updatedReview.id
      })
      copyReviewArray[indexUpdatedReview] = responseJSON
      setReviews(copyReviewArray)
    })
    .catch((error) => console.error(error))
}
```

Pass it to the `Review` component

```js
<Review
  key={review.id}
  review={review}
  handleEdit={handleEdit}
  handleDelete={handleDelete}
/>
```

The way we will approach this is to add a button to edit. This will toggle the view from a `review` to a `reviewForm` by using conditional rendering with a ternary operator. It will end up looking like this example:

```js
// DEMO DO NOT CODE
{
  viewEditForm ? <ShowIfValueIsTrue /> : <ShowIfValueIsFalse />
}
```

<br />

```js
// Components/Review.jsx
import { useState } from 'react'
import ReviewForm from './ReviewForm'
function Review({ review, handleDelete, handleEdit }) {
  const [viewEditForm, setEditForm] = useState(false)
  const toggleView = () => {
    setEditForm(!viewEditForm)
  }
  return (
    <div className="Review">
      {viewEditForm ? (
        <ReviewForm
          reviewDetails={review}
          toggleView={toggleView}
          handleEdit={handleEdit}
        />
      ) : (
        <div>
          <h4>
            {review.title} <span>{review.rating}</span>
          </h4>
          <h5>{review.reviewer}</h5>
          <p>{review.content}</p>
        </div>
      )}
      <div className="review-actions">
        <button onClick={toggleView}>
          {viewEditForm ? 'Cancel' : 'Edit this review'}
        </button>
        <button onClick={() => handleDelete(review.id)}>delete</button>
      </div>
    </div>
  )
}

export default Review
```

Now you should be able to click the `edit this review` button and toggle between the two components.

And that should be it! A one-to-many relationship is one way to do full CRUD on a second model.

### Bonus

Those ratings numbers are boring. Can you figure out a way to display the correct number of star emojis ⭐️ to represent ratings?

## New Bookmarks Bonus Challenges/Lab time

Are you looking for a challenge?

BONUS FORM change the category to options of already created categories, then allow for a new category option where a user will enter a new category

Add functionality that the categories are always lowercase, including new entries.

Alternatively, if you have finished the Tuner lab requirements, you can implement a `playlist` model where a `playlist` will have many songs.

**HINT**: If you delete a playlist, do you want the songs to be deleted too? This will likely be a different use case than what we did with bookmarks.
