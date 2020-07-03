# RFC: Field Coordinate Serialization

_This is a sister RFC to [RFC: Field Coordinates](./FieldCoordinates.md). This
document assumes knowledge of the Field Coordinates RFC._

This RFC outlines the seralization of field nodes in a GraphQL query to ["field
coordinates"](./FieldCoordinates.md).

## 📜 Problem Statement

Third party GraphQL tooling and libraries may wish to refer to a field, or set of
fields refered to in a document (e.g. query, mutation or subscription). Use cases include documentation, metrics and logging libraries.

![](https://i.fluffy.cc/g78sJCjCJ0MsbNPhvgPXP46Kh9knBCKF.png)

_(Field coordinates shown in a query - [Apollo Studio](https://www.apollographql.com/docs/studio/)_)

There already exists a convention used by some third party libraries for
calculating field coordinates in a query. However, there is no formal
specification or name for this convention.

This RFC proposes standardization for computing field coordinates.

## 🐕 Worked Example

Consier the following schema:

```graphql
type Person {
  name: String
}

type Business {
  name: String
  owner: Person
}

type Query {
  searchBusiness(name: String): [Business]
}
```

And the following query:

```graphql
query {
  searchBusinesses(name: "El Greco Deli") {
    name
    owner {
      name
    }
  }
}
```

A GraphQL server may wish to log which fields are accessed in a query, such that
we can compute a list of "most popular fields" in the schema. We may then want to
use this list to optimize the most frequently accessed fields in our schea.

Field coordinates allow us to store this list of "most popular fields".

We want a way to calculate field coordinates from a given field node (and by
extension, calculate a list of all field coordinates contained in a document).

From the query above, we would calculate the following list of field coordinates:

- `Query.searchBusinesses`
- `Business.name`
- `Business.owner`
- `Person.name`

## Examples in industry

- [Apollo Studio](https://www.apollographql.com/docs/studio/) shows field
  coodinates when hovering over fields in a query:

  ![](https://i.fluffy.cc/g78sJCjCJ0MsbNPhvgPXP46Kh9knBCKF.png)

- [extract-field-coordinates](https://github.com/sharkcore/extract-field-coordinates)
  extracts a list of field coordinates from a schema and query document.

## Drawbacks

- Requires the corresponding schema as an input to be able to calculate the field
  coordinate

- **Edge Cases:** There are some edge cases where we need to decide (or let the
  user decide) behaviour:

- What happens when the schema is outdated or a type referenced in the query is
  not present in the schema? Do we throw an error as the document/schema is
  invalid, or do we try a "best effort" approach to preserve as much information
  as possible?

- TODO: add more edge case behaviour from the [sample implementation](https://github.com/sharkcore/extract-field-coordinates)

## Alternatives Considered

### Field Paths

[`path` exists as an attribute on `GraphQLResolveInfo`](https://github.com/graphql/graphql-js/blob/8f3d09b54260565/src/type/definition.js#L951).

Given the following query:

```graphql
query {
  searchBusinesses(name: "El Greco Deli") {
    name
    owner {
      name
    }
  }
}
```

The field coordinate `Person.name` may also be written as the following field
path:

```json
["query", "searchBusinesses", 1, "owner", "name"]
```

- Like field coordinates, this uniquely identifies the "name" field and
  disambiguates it from the "name" field on the `Business` type.
- Note that we also didn't need a schema to calculate the field path - this can
  be with just the query document.

However - we are unable to tell from a field path alone what the parent _type_
was.

Consider the following query:

```graphql
query {
  getUsers(name: "mark") {
    name
  }
}
```

The same field coordinate `Person.name` in the context of this query would be
written as:

```json
["query", "getUsers", 1, "name"]
```

Field Paths alone are insufficient to do normalization of the field nodes. We
need the parent type information from the schema to work out that both of these
field paths reference the same field in the schema.
