# GraphQL

http://graphql.org

## Operation Types:

* [`query`](#query)
* [`mutation`](#mutation)
* [`subscription`](#subscription)

## Schemas

```bash
# this is the root object
{

}

# fields can be selected on this
{
  dog
}

# fields of that field can also be selected
{
  dog {
    name
  }
}
```

### Types

#### Query and Mutation

```bash
# query and mutation types specify fields that can be queried
type Query {
  dog(name: String): Dog
}

type Mutation {
  addFriend(name: String, friend: Dog): Dog
}
```

#### Scalar Types

```bash
String
Int
Float
Boolean
ID # a special type of String, unique for each value

# scalars can be added (although not all
# implementations support this)
scalar Date
```

#### Enums

```bash
enum Interval {
  YEAR
  DOGYEAR
  DAY
}
```

#### Null
```bash
# an exclamation point means a field cannot be null
# if the value is null, an execution error will occur
type Dog {
  name: String!
}

# error response
{
  errors: [
    {
      message: "...",
      locations: [
        { line: 1, column: 2 }
      ]
    }
  ]
}
```

#### Objects

```bash
# object types describe objects that can be fetched and their fields
type Dog {
  name: String!
  breed: String!
  # fields describe functions. The default is a function with zero
  # arguments, which doesn't need to be specified. Default values can be
  # provided to make arguments optional.
  age(unit: Interval = YEAR): Float!
}
```

#### Lists

```bash
type Dog {
  # arrays are specified with square brackets
  friends: [Dog]
}

# array can be null, but dogs cannot
friends: [Dog!]
# array cannot be null (will return empty array if no matches)
friends: [Dog]!
```

#### Interface

```bash
# interfaces define required fields
interface Animal {
  id: ID!
  name: String!
}

# types can implement an interface
type Dog implements Animal {
  id: ID!
  name: String!
  age: Float!
}

type Cat implements Animal {
  id: ID!
  name: String!
  live: Int!
}

# interfaces can be the type for a field
type Query {
  GetAnimal($name: String!): Animal
}

# however, you can only query on fields of the interface
query GetAnimal($name: String!) {
  # this will cause an error
  age
}
# inline fragments allow you to query type specific fields
```

#### Union

```bash
# a union groups types (not interfaces!), but without common fields
union Thing = Dog | Cat | Tree
# this can be useful for tasks that can return multiple types
```

#### Input Types

```bash
# input types look like regular types, but are specified
# with the input keyword
input DogInput {
  name: String!
  breed: String!
}
# input types are passed to mutations
```

### Resolvers

```bash
# resolvers are functions associated with a field
# if a resolver returns a scalar, then it stops. For other
# types, it goes deeper (to that type's fields)
{
  dog {
    name: String!, # string, stops
    friends: [Dog]! # object, so keeps going
  }
}
```

```js
// resolvers receive four arguments:
// 1. obj is the previous object (one level up, not old)
// 2. args are arguments provided in the field's query
// 3. context extra, useful data
// 4. info about query's execution state (use rarely)
{
  Query: {
    dog(obj, args, context, info) {
      return api.getDog(args.id).then(data => {
        return new Dog(data);
      });
    }
  },
  Dog: {
    name(obj, args.context) {
      return obj.name
    }
  }
}
```

## Introspection

Useful for exploring the GraphQL structure.

http://graphql.org/learn/introspection/


## Query

```bash
# the query operation type indicates that some gql is a query
query {
  ...
}
# this isn't required, but is useful for clarity
```

```bash
# a query get data
# the query specifies the shape of the data it wants
query {
  dog {
    breed
  }
}
# returns object with data property of same shape
{
  data: {
    dog: {
      breed: "corgi"
    }
  }
}

query {
  # you can also insert comments into a query
  dog
}
```

```bash
# subquery with nested properties
query {
  dog: {
    name,
    friends: {
      name
    }
  }
}
# returns
{
  data: {
    name: 'Ralph',
    friends: [
      { name: 'Spot' },
      { name: 'Fido' }
    ]
  }
}
```

### Arguments

```bash
query {
  dog(id: "k9") {
    name,
    breed
  }
}
# returns dog that matches the id
{
  data: {
    dog: {
      name: 'Sahara'
    }
  }
}
```

```bash
# sub-properties can also be queried
query {
  dog(id: "k9") {
    weight(unit: KG)
  }
}

{
  data: [
    dog: {
      weight: 23
    }
  ]
}
```

### Aliases

```bash
# properties can be aliased, which is helpful with duplicates
query {
  shortDog: dog(breed: "dachsund") {
    name
  },
  tallDog: dog(breed: "great dane") {
    name
  }
}

{
  data: {
    shortDog: {
      name: 'Goliath'
    },
    tallDog: {
      name: 'Tiny'
    }
  }
}
```

### Fragments

```bash
# fragments let you define partial queries, which can
# be included as parts of a larger query
query {
  shortDog: dog(breed: 'chihuahua') {
    ...quantitativeFields
  }
}

fragment quantitativeFields on Dog {
  height
  weight
  age
}
```

```bash
# inline fragments can be used to specify properties
# based on a type, which is useful for subtypes
query {
  animal {
    name
    ... on Dog {
      age
    }
    ... on Cat {
      lives
    }
  }
}

# these can be combined with named fragments,
# which already include an "on"
query {
  animal {
    name
    ...dogProperties
    ...catProperties
  }
}

fragment dogProperties on Dog {
  age
}
fragment catPropeties on Cat {
  lives
}
```

### Operation Name and Variables

```bash
# an operation name is like a function name
query DogData {
  dog {
    name
    breed
    age
  }
}

# operations must be named to pass dynamic variables
# variables start with $.
# The type of the variable must be provided.
# The exclamation point indicates this variable is required
query DogData($name: String!) {
  dog(name: $name) {
    name
    breed
    age
  }
}

# default variables can be provided
query DogData($name: String = "Terra") {
  dog(name: $name) {
    ...
  }
}

# directives can be used to control whether
# part of a query is included
query DogData($name: String!, $andFriends: Boolean!, $withAge: Boolean!) {
  dog(name: $name) {
    name
    breed
    # only @include friends when andFriends is true
    friends @include(if: $andFriends) {
      name
      breed
    }
    # you can also $skip fields
    age @skip(if: $withAge)
  }
}
```

### Meta

```bash
query {
  dog {
    # __typename returns an object's type
    # this is useful for queries that can return
    # multiple types
    __typename
  }
}
```

### Custom Directives

While GraphQL includes the `@skip` and `@include` directives, it is possible to write your own as well.

The exact implementation varies by GraphQL framework.

When defining a directive in a schema, it requires a name and what it can be added to (fields, types, etc.). The directive can also take arguments.

```bash
directive @yo(
 arg: String = "default value"
) on FIELD_DEFINTION

query {
  dog {
    name @yo
  }
}
```


## Mutations

```bash
# while queries read data,
# mutations are used for setting/modifying data
mutation AddFriend($name: String!, $friend: String!) {
  addFriend(name: $name, friend: $friend) {
    # you can return the results of the mutation
    name
    friends {
      name
    }
  }
}

# an operation can contains multiple mutations
# they will be run consecutively, not at the same time
```
