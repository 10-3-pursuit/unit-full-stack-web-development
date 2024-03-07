# Express and PostgreSQL: Update & Delete

This is a continuation of the build that started with seed and read, and then show and create.

## Learning Objectives

- Create a model file connected to a database that can update a specific row in a table and remove a particular row from a table.
- Handle asynchronous errors within a controller file and respond from the server appropriately.
- Use asynchronous code within a controller file to respond to client requests with persisted data.

## Delete

Create an async arrow function and be sure to include it in `module.exports`

```js
// queries/colors.js
const deleteColor = async (id) => {
  try {
  } catch (error) {
    return error
  }
}

module.exports = {
  getAllColors,
  getColor,
  createColor,
  deleteColor,
}
```

```js
const deleteColor = async (id) => {
  try {
    const deletedColor = await db.one(
      'DELETE FROM colors WHERE id = $1 RETURNING *',
      id
    )
    return deletedColor
  } catch (error) {
    return error
  }
}
```

Import the function:

```js
// controllers/colorController.js
const {
  getAllColors,
  getColor,
  createColor,
  deleteColor,
} = require('../queries/color')
```

Create the delete route and test it with Postman. Remember to have selected:

- Route ` DELETE` `/colors/:id`

Test that the route works:

```js
// controllers/colorController.js
// DELETE
colors.delete('/:id', async (req, res) => {
  const { id } = req.params
  res.status(200).json({ id })
})
```

Now add the database call:

```js
// DELETE
colors.delete('/:id', async (req, res) => {
  const { id } = req.params
  const deletedColor = await deleteColor(id)
  res.status(200).json(deletedColor)
})
```

Add some error handling.

```js
colors.delete('/:id', async (req, res) => {
  const { id } = req.params
  const deletedColor = await deleteColor(id)
  if (deletedColor.id) {
    res.status(200).json(deletedColor)
  } else {
    res.status(404).json('Color not found')
  }
})
```

Test it with Postman again.

Remember to have:

- route DELETE /colors/:id

## Update

Create an async arrow function and be sure to include it in `module.exports`

```js
// queries/colors.js
const updateColor = async (id, color) => {
  try {
  } catch (error) {
    return error
  }
}

module.exports = {
  getAllColors,
  getColor,
  createColor,
  deleteColor,
  updateColor,
}
```

Add the query:

```js
const updateColor = async (id, color) => {
  try {
    const updatedColor = await db.one(
      'UPDATE colors SET name=$1, is_favorite=$2 where id=$3 RETURNING *',
      [color.name, color.is_favorite, id]
    )
    return updatedColor
  } catch (error) {
    return error
  }
}
```

Import the function

```js
// controllers/colorController.js
const {
  getAllColors,
  getColor,
  createColor,
  deleteColor,
  updateColor,
} = require('../queries/color')
```

First confirm the route is reachable.

```js
// UPDATE
colors.put('/:id', async (req, res) => {
  const { id } = req.params
  res.status(200).json({ id })
})
```

Now add the database call.

```js
// UPDATE
colors.put('/:id', async (req, res) => {
  const { id } = req.params
  const updatedColor = await updateColor(id, req.body)
  res.status(200).json(updatedColor)
})
```

Test it and remember to have the following:

- route PUT /colors/:id

```js
{
 "name":"floralwhite",
 "is_favorite": "true"
}
```

Remember, the same rules apply that applied to create. There must be a `name`, and `is_favorite` must be a boolean (or undefined - why is undefined an ok option?).

You can now easily update the route to do those checks because you separated this functionality from the create route.

```js
colors.put("/:id", checkName, checkBoolean, async (req, res) => {
```

**Thought Question**: Do you need to put these validations in Index, Show or Delete? Why or why not?

## Save it

- `git add -A`
- `git commit -m 'update and delete complete'`.

## Reference

[Here is the reference build](https://github.com/10-3-pursuit/class-db-bookmarks-backend)
