# RFC: Field Coordinates

**Champion:** @magicmark

This RFC proposes formalizing "field coordinates" - a way to uniquely identify a
field defined in a GraphQL Schema.

This may be listed as an appendix item in the official specification to serve as
an official reference to third party library implementations.

## 🚫 What this RFC does _not_ propose

This RFC does not seek to change the GraphQL language in any way.

- There are **no proposed GraphQL runtime changes**
- There are **no proposed changes to how we parse documents**.

## 📜 Problem Statement

Third party GraphQL tooling and libraries may wish to refer to a field, or set of
fields in a schema. Use cases include documentation, metrics and logging
libraries.

![](https://i.fluffy.cc/5Cz9cpwLVsH1FsSF9VPVLwXvwrGpNh7q.png)

_(Example shown from GraphiQL's documentation search tab)_

There already exists a convention used by some third party libraries for writing
out fields in a unique way for such purposes. However, there is no formal
specification or name for this convention.

## ✨ Worked Example

For example, consider the following schema:

```graphql
type Person {
  name: String
}

type Business {
  name: String
  owner: Person
}

type Query {
  searchBusinesses(name: String): [Business]
}
```

We can write the following list of field coordinates:

- `Person.name` uniquely identifies the "name" field on the "Person" type
- `Business.name` uniquely identifies the "name" field on the "Business"
  type
- `Business.owner` uniquely identifies the "owner" field on the "Business" type
- `Query.searchBusinesses` uniquely identifies the "searchBusinesses" field on
  the "Query" type

This RFC standardizes how we write field coodinates as above.

## 🎨 Prior art

- The name "field coordinates" comes from [GraphQL Java](https://github.com/graphql-java/graphql-java)
  (4.3k stars), where this is already used as described - [Github comment](https://github.com/graphql/graphql-spec/issues/735#issuecomment-646979049) - [Implementation](https://github.com/graphql-java/graphql-java/blob/2acb557474ca73/src/main/java/graphql/schema/FieldCoordinates.java)

- GraphiQL displays field coordinates in its documentation search tab:

  ![](https://i.fluffy.cc/5Cz9cpwLVsH1FsSF9VPVLwXvwrGpNh7q.png)

- [GraphQL Inspector](https://github.com/kamilkisiela/graphql-inspector) (840 stars) shows type/field pairs in its output:

  ![](https://i.imgur.com/HAf18rz.png)

- [Apollo Studio](https://www.apollographql.com/docs/studio/) shows field
  coodinates when hovering over fields in a query:

  ![](https://i.fluffy.cc/g78sJCjCJ0MsbNPhvgPXP46Kh9knBCKF.png)

## 🗳️ Alternatives considered

### Naming

- "type/field pairs" was the original suggestion

  However, "field coordinates" was chosen as it is already understood and used by
  the popular [GraphQL Java](https://github.com/graphql-java/graphql-java)
  project.

- "Field path" / "GraphQL path"

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

  Note that here, the "path" is a serialized _response_ tree traversal, instead
  of describing the location of the field in the _schema_.

  Since "path" is already used in GraphQL nonclemanture to describe the location
  of a field in a response, we'll avoid overloading this term.

### Separator

This RFC proposes using "`.`" as the separator character. The following have
also been proposed:

- `Foo::bar`
- `Foo#bar`
- `Foo->bar`
- `Foo~bar`
- `Foo:bar`

"`.`" is already used in the existing implementations of field coordinates, hence
the suggested usage in this RFC. However, we may wish to consider one of the
alternatives above, should this conflict with existing or planned language
features.

## 🤔 Drawbacks / Open questions

- https://github.com/graphql/graphql-spec/issues/735 discusses potential
  conflicts with the upcoming namespaces proposal - would like to seek clarity on
  this

- **Is this extensible enough?** The above issue discusses adding arguments as
  part of this specifcation - we haven't touched on this here in order to keep
  this RFC small, but we may wish to consider this in the future (e.g.
  `Query.searchBusiness:name`).

- **How will this play with namespaces?** Not sure what this looks like yet!

- **Would we want to add a method to graphql-js?** A `fieldCoordinateToFieldNode`
  method (for example) may take in a field coordinate string and return a field
  AST node to serve as a helper / reference implementation of the algorithm to
  look up the field node.
