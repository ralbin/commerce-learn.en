---
title: Schema language with GraphQL
description: Learn about the schema involved with GraphQL. Read a description of the schema, along with some interesting patterns and ways to read the schema.
landing-page-description: This is an introduction to GraphQL. Understanding the schema and how to interpret some of the elements
short-description: This is an introduction to GraphQL. Understanding the schema and how to interpret some of the elements
kt: 11524
doc-type: tutorial
audience: all
last-substantial-update: 2022-12-13
feature: GraphQL
topic: Commerce, Architecture, Headless
role: Architect, Developer
level: Beginner, Intermediate
exl-id: 6b59db07-b99e-47ae-9ccb-d4904afc8251
---
# Schema language

The queries and mutations used rely on a specific data graph being implemented at the server, which the GraphQL runtime consumes and uses to resolve the query. The GraphQL specification defines an agnostic language for expressing the types and relationships of your data graph.

Here is an abbreviated type schema that supports the queries and mutations you've looked at so far:

```graphql
input FilterMatchTypeInput {
  match: String
}

type Money {
  value: Float
}

type Country {
  id: String
  full_name_english: String
}

interface ProductInterface {
  sku: String
  name: String
  related_products: [ProductInterface]
}

type CategoryFilterInput {
  name: FilterMatchTypeInput
}

type CategoryProducts {
  items: [ProductInterface]
}

type CategoryTree {
  name: String
  products(pageSize: Int, currentPage: Int): CategoryProducts
}

type CategoryResult {
  items: [CategoryTree]
}

type Products {
  items: [ProductInterface]
}

type Query {
  country (id: String): Country
  categories (filters: CategoryFilterInput): CategoryResult
  products (search: String): Products
}

input CartItemInput {
  sku: String!
  quantity: Float!
}

type CartPrices {
  grand_total: Money
}

type Cart {
  prices: CartPrices
  total_quantity: Float!
}

type AddProductsToCartOutput {
  cart: Cart!
}

type Mutation {
  addProductsToCart(cartId: String!, cartItems: [CartItemInput!]!): AddProductsToCartOutput
}
```

You can delve into [the GraphQL documentation](https://graphql.org/learn/schema/){target="_blank"} to learn about the details of the type system, including syntax for some concepts not represented here. The above example, however, is self-explanatory. (Also, note how similar the syntax is to query syntax.) Defining a GraphQL schema is simply a matter of expressing the available arguments and fields of a given type, along with the types of those fields. Each complex field type must itself have a definition, and so on, through the tree, until you get to simple scalar types like `String`.

The `input` declaration is in all respects like a `type` but defines a type that can be used as input for an argument. Also note the `interface` declaration. This serves a function more or less the same as interfaces in PHP. Other types inherit from this interface.

The syntax `[CartItemInput!]!` looks tricky but is fairly intuitive in the end. The `!` _inside_ the bracket declares that every value in the array must be non-null, while the one _outside_ declares that the array value itself must be non-null (for example, an empty array).

>[!NOTE]
>
>The logic for how data is fetched and formatted according to a schema, and how such logic is mapped to particular types, is up to the GraphQL runtime implementation. Implementations, however, should follow a conceptual flow that makes sense in light of an understanding around nested fields: A resolve operation associated with the root `Query` or `Mutation` type is performed, which examines each field specified in the request. For each field that resolves to a complex type, a similar resolve is done for that type, and so on, until everything has resolved into scalar values.

{{$include /help/_includes/graphql-rest-related-links.md}}
