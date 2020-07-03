# C. Tooling Conventions

This specification document formalizes naming and conventions that may be used by
GraphQL tooling and third party libraries.


## Field Coordinates

FieldCoordinate : TypeName `.` FieldName

A field coordinate may be used to uniquely identify a field in a GraphQL schema.
It is encoded as a string containing a dot separated parent type and field pair.

**Examples**

Consider the following schema:

```graphql example
type Person {
  name: String
  age: Int
}

type Business {
  name: String
  owner: Person
}
```

In order to refer to the `name` field on the `Business` type, we would write the
following field coordinate:

```example
Business.name
```

Simiarly, in order to refer to the `name` field on the `Person` type, we would
write the following field coordinate:

```example
Person.name
```

**Formal Specification**

* Split the field coordinate string by {`.`}
* Let {typeName} and {fieldName} be the zeroth and first results respectively
* {typeName} must be defined in the schema as a {Name} attribute of {ObjectTypeDefinition} 
* {fieldName} must be defined as a {Name} attribute in a {FieldDefinition} of the {FieldsDefinition} attribute of the {ObjectTypeDefinition} found above

## Converting a FieldNode to a Field Coordinate

Given a matching schema, we may wish to refer to a field used in a query document
using a field coordinate.

**Example**

Consider the following schema:

```graphql example
type Person {
  name: String
  age: Int
}

type Business {
  name: String
  owner: Person
}

type Query {
  searchBusinesses(name: String): [Business]
}
```

Also consider the following document:

```graphql example
query {
  searchBusinesses(name: "El Greco Deli") {
    name
    owner {
      name
    }
  }
}
```

We can calculate that the list of field coordinates referenced in this query is
as follows:

```example
Query.searchBusinesses
Business.name
Business.owner
Person.name
```

**Formal Specification**

TODO: Write up an algorithm

(code implementation: https://github.com/sharkcore/extract-field-coordinates/blob/master/src/extract-field-paths.js)