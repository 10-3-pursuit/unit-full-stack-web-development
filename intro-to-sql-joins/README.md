# Introduction to SQL Joins

## Lesson Objectives

By the end of this lesson, you should be able to:

- Explain the purpose of table normalization.
- Describe the business and technical implications of a one-to-many relationship.
- Explain the function of a foreign key.
- Implement a join on two tables through matching IDs
- Explain how to create and use a many-to-many relationship between two entities.
- Explain what a lookup table is and what function it serves

## Data modeling and normalization

It's unusual to have just one table in a business application. Typically, there are many tables to represent data in a meaningful way.

Let's imagine you are designing an application for a new boutique travel travel boutique, where guests will get to work with an agent to book experiences at unique hotels not part of any franchises. What kind of data would you need?

- Hotel
  - name - string
  - city - string
  - state - string
  - rating - integer
  - pets - boolean

You can image a table like so:

`Hotel Table`
| name | city | state | rating | pets |
| :--------------: | :--------: | :--------: | :----: | :--: |
| Hotel California | null | California | 4 | true |
| Great Northern | Twin Peaks | Washington | 3 | true |

But to book a room, you would need information on the rooms available in each hotel. **One** hotel **has many** hotel rooms.

Let's try to put the info in a one table:

`Hotel Table (with all the rooms in one table)`

|      name      |    city    |   state    | rating | pets |        room 1         |           room 2            |       room 3        |
| :------------: | :--------: | :--------: | :----: | :--: | :-------------------: | :-------------------------: | :-----------------: |
| Great Northern | Twin Peaks | Washington |   3    | true | queen vacant 1200 202 | penthouse 10,000 303 vacant | not vacant twin 101 |

You have a lot of data jammed into one column for each room. It's hard to read. It's hard to update. It is hard to make a query for which rooms are vacant.

It would be better if there were a separate table for each room.

- Room
  - type - string
  - price - integer
  - number - integer
  - vacant - boolean

`Rooms Table`
| name | price | room_num | vacant |
| :-------: | :---: | :------: | :----: |
| Queen | 1200 | 202 | true |
| Penthouse | 10000 | 303 | true |
| Twin | 600 | 101 | false |

Organizing data into multiple tables to improve updating, inserting, deleting, and reading data is called [database normalization](https://en.wikipedia.org/wiki/Database_normalization).

## Joins

### Relating tables and foreign keys

Now that hotels and their rooms are two separate tables, how can you relate the two tables to each other?

First, the hotels must have a unique identifying field. In this case, a simple integer will be used:

`Hotels Table Updated`
| id | name | city | state | rating | pets |
| :-: | :------------------: | :--------: | :------------------: | :----: | :--: |
| 1 | Hotel California | null | California | 4 | true |
| 2 | Great Northern | Twin Peaks | Washington | 3 | true |
| 3 | Grand Budapest Hotel | null | Republic of Zubrowka | 5 | null |

Next, the rooms will also have a unique id (in the id column).

The hotel rooms can belong to any hotel or no hotel. You can include the hotel id as a field on the hotel rooms.

`Rooms Table (updated)`
| id | name | price | room_num | vacant | hotel_id |
| :-: | :-------: | :---: | :------: | :----: | :------: |
| 10 | Queen | 1200 | 202 | true | 2 |
| 11 | Penthouse | 10000 | 303 | true | 1 |
| 12 | Twin | 600 | 101 | false | 2 |
| 13 | King | 6000 | 001 | false | 4 |

This would allow you to create a query to list all the rooms associated with The Great Northern.

```SQL
SELECT * FROM rooms WHERE hotel_id = 2;
```

Expected results:

| id  | name  | price | room_num | vacant | hotel_id |
| :-: | :---: | :---: | :------: | :----: | :------: |
| 10  | Queen | 1200  |   202    |  true  |    2     |
| 12  | Twin  |  600  |   101    | false  |    2     |

The id field of the hotel in the rooms table is called a `foreign key` because it is the unique identifier that belongs to the hotel in another table (a foreign table, if you will).

### Joining tables

Imagine you would like to know which hotels have vacancies. You would need to query the rooms table and find where vacant is true. Then, you would need to look up the hotels that match. You can make two separate queries and maybe take some notes or copy/paste to put the data together. But there is a better way. You can perform a query that joins information from both tables. To do this, you would use a `join` statement.

There are four main ways to join tables. Additionally, when you put two tables next to each other, one will be on the left and one on the right. Therefore there are references to `left` and `right` when describing joins.

In the following examples, hotels are on the left table, and rooms are on the right table:

- Listing only the hotels and rooms that have associations. (inner join).
- List all the hotel data and only the room data with an associated hotel. (left join).
- Listing only the hotel data that has at least one associated room. (right join).
- Listing all the hotels and room data. (full join).

Here is a simplified visual of the four types of joins.

![](./assets/sql-join-venn-diagram.png)

**Caveat:** Some people don't like using Venn Diagrams to describe joins as an oversimplified representation of what is happening. That's ok! Venn Diagrams are a great way to get started. When you are ready, you can read something more technical like [this](https://blog.jooq.org/2016/07/05/say-no-to-venn-diagrams-when-explaining-joins/).

#### Full Outer Join

Start with a full join. That will put all of the data of both tables together. You will join the two tables based on when the hotel `id` from the `hotels` table matches the `hotel_id` of the `rooms` table.

This will show you hotels with no room data (Grand Budapest Hotel) and three rooms with a hotel_id of 7, even though there is no hotel with an id of 7.

```SQL
SELECT * FROM hotels FULL OUTER JOIN rooms ON hotels.id = rooms.hotel_id;
```

You can see that you get a new (temporary) table showing all your hotel data on the left and your rooms data on the right. The rows are aligned based on the value of the hotel id from the hotels table and the `foreign key` from the rooms table.

| id  |         name         |    city    |        state         | rating | pets | \|  | id  |   name    | price | room_num | vacant | hotel_id |
| :-: | :------------------: | :--------: | :------------------: | :----: | :--: | :-: | :-: | :-------: | :---: | :------: | :----: | :------: |
|  1  |   Hotel California   |    null    |      California      |   4    | true | \|  | 10  |   Queen   | 1200  |   202    |  true  |    2     |
|  1  |   Hotel California   |    null    |      California      |   4    | true | \|  | 12  |   Twin    |  600  |   101    | false  |    2     |
|  2  |    Great Northern    | Twin Peaks |      Washington      |   3    | true | \|  | 11  | Penthouse | 10000 |   303    |  true  |    1     |
|  3  | Grand Budapest Hotel |    null    | Republic of Zubrowka |   5    | null | \|  |     |           |       |          |        |          |
|     |                      |            |                      |        |      | \|  | 13  |   King    | 6000  |   001    | false  |    4     |

Determining which table is on the left or right is based on the order you write your SQL statement.

```SQL
SELECT * FROM rooms FULL OUTER JOIN hotels ON hotels.id = rooms.hotel_id;
```

| id  | name |   price   | room_num | vacant | hotel_id | \|  | id  |         name         |    city    |        state         | rating | pets |
| :-: | :--: | :-------: | :------: | :----: | :------: | :-: | :-: | :------------------: | :--------: | :------------------: | :----: | :--: |
| 10  |  10  |   Queen   |   1200   |  202   |   true   | \|  |  1  |   Hotel California   |    null    |      California      |   4    | true |
| 11  |  11  | Penthouse |  10000   |  303   |   true   | \|  |  2  |    Great Northern    | Twin Peaks |      Washington      |  true  | true |
| 12  |  12  |   Twin    |   600    |  101   |  false   | \|  |  1  |   Hotel California   |    null    |      California      |   4    | true |
| 13  | King |   6000    |   001    | false  |   null   | \|  |  4  |                      |            |                      |        |      |
|     |      |           |          |        |          | \|  |  3  | Grand Budapest Hotel |    null    | Republic of Zubrowka |   5    | null |

Due to the simplicity of this example, there is no compelling reason to put your hotels or rooms on either side besides personal choice. As you learn to build more complex queries where you may be joining three or more tables, there would be a need for more careful thought about the order of how the tables are joined.

#### Inner Join

A lot of times, you want to avoid looking at data with null values. You'd use an inner join if you're going to ignore all the data with no matching values from both tables.

```SQL
SELECT * FROM hotels INNER JOIN rooms ON hotels.id = rooms.hotel_id;
```

| id  |       name       |    city    |   state    | rating | pets | \|  | id  |   name    | price | room_num | vacant | hotel_id |
| :-: | :--------------: | :--------: | :--------: | :----: | :--: | :-: | :-: | :-------: | :---: | :------: | :----: | :------: |
|  1  | Hotel California |    null    | California |   4    | true | \|  | 10  |   Queen   | 1200  |   202    |  true  |    2     |
|  1  | Hotel California |    null    | California |   4    | true | \|  | 12  |   Twin    |  600  |   101    | false  |    2     |
|  2  |  Great Northern  | Twin Peaks | Washington |   3    | true | \|  | 11  | Penthouse | 10000 |   303    |  true  |    1     |

You can still have null or empty values within your tables. What is excluded is `Grand Budapest Hotel` because there are no hotel rooms that go with it, and there are no rooms with a foreign key `hotel_id` of `4` because there is no hotel in your database with an `id` of `4`.

#### Left Join

If you still want to see the Grand Budapest Hotel, even though no rooms are associated with it. You can do a left join.

```SQL
SELECT * FROM hotels LEFT JOIN rooms ON hotels.id = rooms.hotel_id;
```

| id  |         name         |    city    |        state         | rating | pets | \|  | id  |   name    | price | room_num | vacant | hotel_id |
| :-: | :------------------: | :--------: | :------------------: | :----: | :--: | :-: | :-: | :-------: | :---: | :------: | :----: | :------: |
|  1  |   Hotel California   |    null    |      California      |   4    | true | \|  | 10  |   Queen   | 1200  |   202    |  true  |    2     |
|  1  |   Hotel California   |    null    |      California      |   4    | true | \|  | 12  |   Twin    |  600  |   101    | false  |    2     |
|  2  |    Great Northern    | Twin Peaks |      Washington      |   3    | true | \|  | 11  | Penthouse | 10000 |   303    |  true  |    1     |
|  3  | Grand Budapest Hotel |    null    | Republic of Zubrowka |   5    | null | \|  |     |           |       |          |        |          |

#### Right Join

The right join will exclude The Grand Budapest Hotel.

```SQL
SELECT * FROM hotels RIGHT JOIN rooms ON hotels.id = rooms.hotel_id;
```

| id  |       name       |    city    |   state    | rating | pets | \|  | id  |   name    | price | room_num | vacant | hotel_id |
| :-: | :--------------: | :--------: | :--------: | :----: | :--: | :-: | :-: | :-------: | :---: | :------: | :----: | :------: |
|  1  | Hotel California |    null    | California |   4    | true | \|  | 10  |   Queen   | 1200  |   202    |  true  |    2     |
|  1  | Hotel California |    null    | California |   4    | true | \|  | 12  |   Twin    |  600  |   101    | false  |    2     |
|  2  |  Great Northern  | Twin Peaks | Washington |   3    | true | \|  | 11  | Penthouse | 10000 |   303    |  true  |    1     |
|     |                  |            |            |        |      | \|  | 13  |   King    | 6000  |   001    | false  |    4     |

Now you see that the Grand Budapest Hotel is excluded, but the mystery room with no matching hotel is now showing up.

## Many-to-many joins

Next up, we can create tables with relationships of many-to-many. For example, a guest can have (book) many hotel rooms and a hotel room can have (be booked by) many guests.

Let's make some guests. We imagine we'd have information like a first name, last name, credit card number, email, phone, etc. But to keep things simple for this demonstration, our guests will only have one column for their name and whether or not they are traveling with a pet.

```SQL
CREATE TABLE guests (id SERIAL PRIMARY KEY, name TEXT, pet BOOLEAN);
```

| id  |     name     |  pet  |
| :-: | :----------: | :---: |
| 100 | Winter Jones | true  |
| 110 |  Forest Doe  | false |
| 120 | River Miller | true  |

How can we create a table that allows for this relationship? If we try to add to rooms, we end up with a similar predicament as when we first created our one-to-many relationship, and we would have the same troubles if we tried to tack on the data in some columns to the guests' table.

You'll make a `join` or `look up` table to solve this. This table will have no serial primary key. The room id and the guest id will distinguish the rows. We can also add some additional info, for example, the dates (we will use simple text for demonstration purposes).

```SQL
CREATE TABLE rooms_guests (room_id INT, guest_id INT, stay_dates TEXT);
```

| room_id | guest_id |  stay_dates   |
| :-----: | :------: | :-----------: |
|   11    |   100    |   June 6-12   |
|   12    |   120    |  August 6-12  |
|   10    |   110    | October 6-12  |
|   11    |   120    | November 6-12 |

While this table has data, it's not very useful for us to look at.

Let's do a query to show which rooms our guests have stayed in.

Select all columns (all tables) from rooms_guests, join the table guests on where guests.id matches rooms_guests.guest_id match.

Join those results to rooms_guests on where rooms.id rooms_guests.room_id match.

```SQL
SELECT
 *
FROM
 rooms_guests
JOIN
 guests
ON
 guests.id = rooms_guests.guest_id
JOIN
 rooms
ON
 rooms.id = rooms_guests.room_id;

```

| room_id | guest_id |  stay_dates   | \|  | id  |     name     |  pet  | \|  | id  |   name    | price | room_num | vacant | hotel_id |
| :-----: | :------: | :-----------: | :-: | :-: | :----------: | :---: | :-: | :-: | :-------: | :---: | :------: | ------ | -------- |
|   11    |   100    |   June 6-12   | \|  | 100 | Winter Jones | true  | \|  | 11  | Penthouse | 10000 |   303    | true   | 1        |
|   12    |   120    |  August 6-12  | \|  | 120 | River Miller | true  | \|  | 12  |   Twin    |  600  |   101    | false  | 2        |
|   10    |   110    | October 6-12  | \|  | 110 |  Forest Doe  | false | \|  | 10  |   Queen   | 1200  |   202    | true   | 2        |
|   11    |   120    | November 6-12 | \|  | 120 | River Miller | true  | \|  | 11  | Penthouse | 10000 |   303    | true   | 1        |

You can keep going and add information from the hotels table and select the columns and sort the data:

```SQL
SELECT
 guests.name, hotels.name, rooms.name, rooms_guests.stay_dates
FROM
 rooms_guests
JOIN
 guests
ON
 guests.id = rooms_guests.guest_id
JOIN
 rooms
ON
 rooms.id = rooms_guests.room_id
JOIN
 hotels
ON
 hotels.id = rooms.hotel_id
ORDER BY
 guests.name;
```

|     name     |        name        |   name    |  stay_dates   |
| :----------: | :----------------: | :-------: | :-----------: |
|  Forest Doe  | The Great Northern |   Queen   | October 6-12  |
| River Miller |  Hotel California  | Penthouse | November 6-12 |
| River Miller | The Great Northern |   Twin    |  August 6-12  |
| Winter Jones |  Hotel California  | Penthouse |   June 6-12   |

This type of query, while complex, relays important information in a concise and meaningful way.

Finally, when looking at the above table, there are three columns named `name`. You can create aliases for columns.

```SQL
SELECT
 guests.name AS guest_name, hotels.name AS hotel_name, rooms.name AS room_name, rooms_guests.stay_dates
FROM
 rooms_guests
JOIN
 guests
ON
 guests.id = rooms_guests.guest_id
JOIN
 rooms
ON
 rooms.id = rooms_guests.room_id
JOIN
 hotels
ON
 hotels.id = rooms.hotel_id
ORDER BY
 guests.name;
```

|  guest_name  |     hotel_name     | room_name |  stay_dates   |
| :----------: | :----------------: | :-------: | :-----------: |
|  Forest Doe  | The Great Northern |   Queen   | October 6-12  |
| River Miller |  Hotel California  | Penthouse | November 6-12 |
| River Miller | The Great Northern |   Twin    |  August 6-12  |
| Winter Jones |  Hotel California  | Penthouse |   June 6-12   |
