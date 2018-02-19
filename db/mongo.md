# Mongo

Mongo stores "documents", which are similar to JavaScript objects, but stored in binary (`BSON`).

A database has "collections", which are groupings of "documents". The documents in a collection do not have to have the same structure.

## Server

```bash
# start
sudo service mongod start

# stop
sudo service mongod stop

# restart
sudo service mongod restart

# open mongo shell (once db is running)
mongo
# can provide custom host/port
mongo --host localhost:12345
```

## Shell

```bash
# list current database
db

# list available databases
show dbs

# switch to a database
# you can switch to a non-existent, but it will not
# be created until you create a new collection in it
use <name>

# reference a collection through the db
db.someCollection.<someMethod>
```

## Collections

```bash
# collection.insertOne will create a new collection
# it will also create the database
db.petCollection.insertOne({ type: 'dog' })

# createIndex also will create a collection/database
db.petCollection.createIndex(...)

# db.createCollection can also be used
# this is useful for providing options for the collection
db.createCollection('veterinarians', {...})

# for example, a collection can use validation
# to enforce that its documents structure
# https://docs.mongodb.com/manual/core/schema-validation/
db.createCollection('pets', {
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['type'],
      properties: {
        type: {
          bsonType: 'string'
        }
      }
    }
  }
})

# collections can be referenced directly
# or using getCollection()
db.pets
db.getCollection('pets')
```

## Documents

```bash
{
  # a document's primary key
  # if this isn't provided, it will be
  # automatically generated
  _id: ObjectID(...)
}

# insertOne is used to insert a single document
# into a collection
db.pets.insertOne({ type: 'cat' })

# insertMany inserts an array of documents
db.pets.insertMany([
  { type: 'dog' },
  { type: 'cat' },
  { type: 'turtle' }
])

# find finds documents
db.pets.find({ type: 'dog' })

# an empty query will find all
db.pets.find({})

# $in matches if a property exists in the given array
db.pets.find({
  type: { $in: ['dog', 'cat'] }
})

# multiple query conditions is similar to
# a SQL AND
db.pets.find({
  type: 'dog',
  age: 3
})

# $or is similar to a SQL OR
db.pets.find({
  $or: [
    { type: 'dog' },
    { age: 5 }
  ]
})

# documents can be updated
# https://docs.mongodb.com/manual/reference/operator/update/

# updateOne will update the first match
db.pets.updateOne(
  { type: 'dog' },
  { $set: { name: 'Hank' }}
)

# updateMany will update all matches
db.pets.updateMany(
  { type: 'dog' },
  { $set: { noise: 'bark' }}
)

# replaceOne will replace all fields
# but the _id
db.pets.replaceOne(
  { name: 'Hank' },
  { type: 'dog', name: 'Henry' }
)

# documents can be removed using
# deleteOne and deleteMany
db.pets.deleteOne({ name: 'Henry' })
db.pets.deleteMany({ type: 'iguana' })
```

## Indexes

Indexes allow for more efficient querying.

```bash
# an index is created using a field name as a key
# and the index type (sort order) as the value
# where an index of 1 is ascending and
# an index of -1 is descending
db.pets.createIndex({
  age: 1
})

# an index can be created on multiple fields
# the index will be created in the order the
# fields are provided
db.pets.createIndex({
  type: 1,
  age: -1
})
```