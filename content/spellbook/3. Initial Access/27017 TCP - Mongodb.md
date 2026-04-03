## Connect

```sh
wget https://downloads.mongodb.com/compass/mongodb-mongosh_2.5.10_amd64.deb
sudo dpkg -i ./mongodb-mongosh_2.5.10_amd64.deb
```

```sh
mongosh mongodb://127.0.0.1:27017
```
## Syntax

```js
show databases
// Mongo create a db when you insert data into a non-existent table
use academy
// Show tables
show collections
// Insert 1 row data into apples table
db.apples.insertOne({type: "Granny Smith", price: 0.65})
// Insert many rows data into apples table
db.apples.insertMany([{type: "Golden Delicious", price: 0.79}, {type: "Pink Lady", price: 0.90}])
// Get all data in apples table
db.apples.find({})
// Get specific data in apples table
db.apples.find({type: "Granny Smith"})
// Update one, set "price" value
db.apples.updateOne({type: "Granny Smith"}, {$set: {price: 1.99}})
// Update many, increasing "price" value by 1
db.apples.updateMany({}, {$inc: {quantity: 1, "price": 1}})
// Completely overwrites instead of update
db.apples.updateMany({}, {$inc: {quantity: 1, "price": 1}})
// Remove row with "price" less than 0.8
db.apples.remove({price: {$lt: 0.8}})
```

**Operators table:**

|Type|Operator|Description|Example|
|---|---|---|---|
|Comparison|`$eq`|Matches values which are `equal to` a specified value|`type: {$eq: "Pink Lady"}`|
|Comparison|`$gt`|Matches values which are `greater than` a specified value|`price: {$gt: 0.30}`|
|Comparison|`$gte`|Matches values which are `greater than or equal to` a specified value|`price: {$gte: 0.50}`|
|Comparison|`$in`|Matches values which exist `in the specified array`|`type: {$in: ["Granny Smith", "Pink Lady"]}`|
|Comparison|`$lt`|Matches values which are `less than` a specified value|`price: {$lt: 0.60}`|
|Comparison|`$lte`|Matches values which are `less than or equal to` a specified value|`price: {$lte: 0.75}`|
|Comparison|`$nin`|Matches values which are `not in the specified array`|`type: {$nin: ["Golden Delicious", "Granny Smith"]}`|
|Logical|`$and`|Matches documents which `meet the conditions of both` specified queries|`$and: [{type: 'Granny Smith'}, {price: 0.65}]`|
|Logical|`$not`|Matches documents which `do not meet the conditions` of a specified query|`type: {$not: {$eq: "Granny Smith"}}`|
|Logical|`$nor`|Matches documents which `do not meet the conditions` of any of the specified queries|`$nor: [{type: 'Granny Smith'}, {price: 0.79}]`|
|Logical|`$or`|Matches documents which `meet the conditions of one` of the specified queries|`$or: [{type: 'Granny Smith'}, {price: 0.79}]`|
|Evaluation|`$mod`|Matches values which divided by a `specific divisor` have the `specified remainder`|`price: {$mod: [4, 0]}`|
|Evaluation|`$regex`|Matches values which `match a specified RegEx`|`type: {$regex: /^G.*/}`|
|Evaluation|`$where`|Matches documents which [satisfy a JavaScript expression](https://www.mongodb.com/docs/manual/reference/operator/query/where/)|`$where: 'this.type.length === 9'`|

```js
db.apples.find({$where: `this.type.startsWith('G') && this.price < 0.70`});
// Sort on descending price and limit first 2
db.apples.find({}).sort({price: -1}).limit(2)
```
