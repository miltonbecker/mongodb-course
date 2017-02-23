# The MongoDB

Standard databases have **rows** which go into **tables** which go into **databases**.

Mongo has **documents** which go into **collections** which go into **databases**.

**Documents** are **BSON** which are like **JSON**.

You can store any type of data that you can use in JavaScript: strings, numbers, objects, arrays...

## Some commands:

Show the databases:
`show dbs`

Select or create the 'cities' database:
`use cities`

Insert a document into the cities collection:
`db.cities.insert({ "name": "Paris", "country": "France" })`

Find documents which have country == France:
`db.cities.find({ "country": "France" })`

Remove documents which have country == France:
`db.cities.remove({ "country": "France" })`

On the find and remove examples above, the object passed to the methods
is called a **query parameter**.

There is also the **update parameter** which is passed to the update method.
The update parameter uses an **update operator** which, in the case below, is ``$set``.
**Operators** always start with a **$**.
`db.cities.update({ "name": "Paris" }, { "$set": { "rating": 10 } })`
