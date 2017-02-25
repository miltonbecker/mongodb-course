# The MongoDB

Standard databases have **rows** which go into **tables** which go into **databases**.

Mongo has **documents** which go into **collections** which go into **databases**.

**Documents** are stored as **BSON** which is short for **Binary JSON**.

You can store any type of data that you can use in JavaScript: strings, numbers, objects, arrays...

## Some commands:

Show the databases: 

```
show dbs
```

Select or create the 'cities' database:

```
use cities
```

## Insert 

Insert a document into the cities collection:

```javascript
db.cities.insert({ "name": "Paris", "country": "France" })
```

## Find

Find documents which have country == France:

```javascript
db.cities.find({ "country": "France" })
```

Find documents which have country == France and rating == 10:

```javascript
db.cities.find({ "country": "France", "rating": 10 })
```

## Remove Document 

Remove documents which have country == France:

```javascript
db.cities.remove({ "country": "France" })
```

On the find and remove examples above, the object passed to the methods
is called a **query parameter**.

## Update 

There is also the **update parameter** which is passed to the update method.
The update parameter uses an **update operator** which, in the case below, is ``$set``.
**Operators** always start with a **$**.

```javascript 
db.cities.update({ "name": "Paris" }, { "$set": { "rating": 10 } })
``` 

In the example above, **only the first** matched document will be updated. 

If we did **not** use the **$set** operator, the whole matched document would be
redefined to only have the *rating* property, i.e., the document would be 
replaced with the update parameter. This is **useful when importing data**. 

```javascript 
db.cities.update({ "name": "Paris" }, { "rating": 10 })
``` 

To **update all** the matching documents, we need to use the options parameter.
In this case, we set the **multi** option. 

```javascript
db.cities.update(
    { "country": "France" }, 
    { "$set": { "rating": 10 } },
    { "multi": true }
)
```

The **$inc** operator increments the property by the amount you sent, like: 

```javascript
db.cities.update(
    { "country": "France" }, 
    { "$inc": { "rating": -1 } },
)
```
This is going to lower France's rating to 9.

The **upsert** option creates a document in case none was matched:

```javascript
db.cities.update(
    { "country": "Germany" }, 
    { "$set": { "rating": 10 } },
    { "upsert": true }
)
```
The document created would have 2 properties: country and rating. 

## Remove Field

To **remove a field/property** from all documents in the collection:

```javascript
db.cities.update(
    { },
    { "$unset": { "rating": "" } },
    { "multi": true }
)
```

## Rename

To **rename a field** in all documents:

```javascript
db.cities.update(
    { },
    { "$rename": { "name": "Name" } },
    { "multi": true }
)
```

## Array 

To **update an array value** without redefining the whole array,
we use the **positional operator ($)**:

```javascript
db.cities.update(
    { "regions" : "Downtown" },
    { "$set": { "regions.$": "Centre" } },
    { "multi": true }
)
```
You can use regions.0, regions.1, etc to change that specific array index's value. 

For arrays: **$pop (remove), $push (add), $addToSet (add if not already in)**.

## Comparison 

There are also **comparison operators**: **$gt, $gte, $lt, $lte, $ne**.

```javascript
db.cities.find({ "rating": { "$gte": 7 } })
```
Cities which the rating is **greater than or equal to 7**.

```javascript
db.cities.find({ "rating": { "$gt": 5, "$lt": 8 } })
```
Cities which the rating is **greater than 5 and lesser than 8**.

### Comparison and Arrays 

For arrays, we should use **$elemMatch** which means that at least one element 
in the array matches all the criteria. 
Like: do we have an element that is both > 5 and < 8?

If we don't use it, then all the elements in the array will be used to match 
the criteria separately. 
Like: do we have an element that is > 5? Ok, now do we also have an element 
that is < 8? 

```javascript
db.cities.find({ "trainLines": { "$elemMatch": { "$gt": 5, "$lt": 8 } } })
```

## Projection 

We can specify the fields **we want** to be returned.
This is called the **projection parameter**:

```javascript
db.cities.find(
    { "country": "France" },
    { "name": true, "rating": true }
)
```
So, only the *_id*, *name* and *rating* fields will the returned.
Note: The *_id* field is always going to be returned (unless explicitly set to false).

We can also specify the fields that **we do not want** to be returned:

```javascript
db.cities.find(
    { "country": "France" },
    { "country": false }
)
```
So, only the *country* field won't be returned. Everything else will.

**We cannot mix true and false values in the projection, except for 
the _id field, which we can force it not to be returned**:

```javascript
db.cities.find(
    { "country": "France" },
    { "name": true, "rating": true, "_id": false }
)
```

## Cursor methods

Count.

```javascript
db.cities.find({ "country": "France" }).count()
```

Sort. 1 for ASC and -1 for DESC

```javascript
db.cities.find({ "country": "France" }).sort({ "name": 1 });
```

Skip: skip the first *X* documents.
Limit: limit the result to *X* documents.

```javascript
db.cities.find({ "country": "France" }).skip(3).limit(10);
```
## Aggregation Pipeline

You can chain several queries in one command. 
To do it, you use the **aggregate** method.

Then you use a **stage operator**, like **$group**, **$match**, **$sort**, etc. 

## $group 

The **$group** operator's parameters are **one *_id*** and none or 
as **many accumulators** as needed, like explained below:

1. The **_id** followed by the field you want to aggregate by.
The **_id** is called the **group key** and is required.
  1. Note the **$** before the field name. This means the value will be replaced 
  by that field's value 
2. *(Optional)* An accumulator, like a count, sum, average, max number, etc. 

```javascript
db.cities.aggregate([ 
    { "$group": 
        { 
            "_id": "$country",
            "cities_count": { "$sum": 1 },
            "avg_cost_of_life": { "$avg": "$price" }
        }
    }
])
```

## Others 

```javascript
db.cities.aggregate([
    {$match: {rating: {$gte: 8}}}, //get only the cities with rating >= 8
    {$project: {regions: false, _id: false}}, //do not return the regions and _id fields 
    {$group: {_id: "$country"}}, //group by country 
    {$sort: {rating: -1}}, //sort by rating in descending order 
    {$limit: 10} //limit the result to 10 documents 
])
```
No need to wrap everything with quotes.

