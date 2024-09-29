---
icon: material/head-snowflake
---

# Core Concepts of Apollo GraphQL

Apollo GraphQL is built on top of key concepts that make it an efficient, scalable, and modern solution for building APIs. Here, we explore the foundational concepts of Apollo and GraphQL, illustrating how these pieces fit together to create a flexible data-fetching architecture.

## 1. Schema Definition Language (SDL)
GraphQL schemas are the backbone of any GraphQL service. The schema defines all the types, queries, mutations, and subscriptions that are allowed in your API. Apollo uses SDL (Schema Definition Language) to define the schema. Here’s a simple example of a schema:

```graphql
type Query {
  books: [Book]
  book(id: ID!): Book
}

type Book {
  id: ID!
  title: String
  author: String
}
```

In this schema, `Query` is the entry point to read data, and the `Book` type defines the structure of the book object. SDL ensures a strongly typed API, and it’s central to building and maintaining a GraphQL API with Apollo.

## 2. Resolvers
Resolvers are the functions that connect the schema to the actual data. Each field in the schema must have a corresponding resolver function that knows how to fetch or compute the value for that field.

### Example of a Resolver:
```javascript
const resolvers = {
  Query: {
    books: () => getBooks(),
    book: (_, { id }) => getBookById(id),
  },
};
```

In the above example, `books` and `book` are resolvers that fetch data for their respective fields. The `getBooks` and `getBookById` functions represent data-fetching logic that can pull data from databases, third-party APIs, or other sources.

### Resolver Arguments
- **parent:** The previous object, or `null` for root queries.
- **args:** An object containing the arguments passed into the query.
- **context:** A shared object across all resolvers, often used for authentication or data sources.
- **info:** Advanced information about the execution state of the query.

### Example with Arguments:
```graphql
query {
  book(id: "1") {
    title
    author
  }
}
```
This query fetches a single book by ID, and the resolver would handle the `id` argument.

## 3. Queries
In GraphQL, queries allow you to read data. Apollo uses the `Query` type to define the entry points for reading data in the schema. Queries are flexible, allowing clients to request only the data they need.

### Example Query:
```graphql
query {
  books {
    title
    author
  }
}
```

This query will fetch a list of books and return only their title and author fields.

### Nested Queries
GraphQL supports querying nested data, which is especially useful when fetching related entities.

```graphql
query {
  book(id: "1") {
    title
    author
    reviews {
      comment
      rating
    }
  }
}
```

Here, we not only fetch the book’s title and author but also its related `reviews`.

## 4. Mutations
While queries are for reading data, mutations are for modifying data. Apollo defines mutations as part of the schema using the `Mutation` type. Mutations can be used to create, update, or delete data.

### Example Mutation:
```graphql
mutation {
  addBook(title: "New Book", author: "Author Name") {
    id
    title
    author
  }
}
```

This mutation adds a new book to the system. The response contains the newly created book's data.

### Handling Input Types in Mutations:
GraphQL also supports custom input types for mutations, making it easier to pass complex data.

```graphql
input BookInput {
  title: String
  author: String
}

type Mutation {
  addBook(input: BookInput): Book
}
```

The client can then pass structured data as arguments when calling the mutation.

## 5. Subscriptions
Subscriptions in GraphQL enable real-time communication between clients and servers. With Apollo, subscriptions allow clients to receive updates whenever a particular piece of data changes.

### Example Subscription:
```graphql
subscription {
  bookAdded {
    title
    author
  }
}
```

Whenever a new book is added, clients subscribing to `bookAdded` will receive updates in real time. Subscriptions are often built on WebSockets to maintain a persistent connection.

## 6. Context in Apollo Server
Context is a shared object passed to every resolver in an operation. It is typically used to store authentication information, shared resources like database connections, or data sources.

### Example of Context Usage:
```javascript
const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ req }) => {
    const token = req.headers.authorization || '';
    const user = getUserFromToken(token);
    return { user };
  },
});
```

Here, the context contains the `user` object, which can then be accessed in any resolver to check authentication status.

## 7. Apollo Client and Queries
Apollo Client is used on the frontend to send GraphQL queries and mutations. The client is responsible for fetching data from the server and caching it for better performance.

### Fetching Data with Apollo Client:
```javascript
import { useQuery } from '@apollo/client';
import { gql } from 'graphql-tag';

const GET_BOOKS = gql`
  query {
    books {
      id
      title
      author
    }
  }
`;

function BookList() {
  const { loading, error, data } = useQuery(GET_BOOKS);
  
  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error :(</p>;

  return data.books.map(({ id, title, author }) => (
    <div key={id}>
      <p>{`${title} by ${author}`}</p>
    </div>
  ));
}
```

This example shows how the Apollo Client integrates with React to fetch and display data from a GraphQL server.

## 8. Caching in Apollo Client
Apollo Client has built-in support for caching GraphQL data, which reduces the need to make repeated network requests for the same data. The cache is normalized, meaning it stores individual objects by their unique identifiers and automatically updates the cache when mutations occur.

### Example of Cache Configuration:
```javascript
const client = new ApolloClient({
  uri: 'https://example.com/graphql',
  cache: new InMemoryCache(),
});
```

The `InMemoryCache` is the default caching mechanism in Apollo Client, but custom cache policies can also be defined for specific use cases.

## 9. Error Handling
Apollo Client provides mechanisms for handling errors in queries and mutations. When an error occurs, it can be caught and displayed appropriately in the UI.

### Example of Error Handling:
```javascript
const { loading, error, data } = useQuery(GET_BOOKS);

if (error) {
  console.error("Error loading books:", error);
  return <p>Failed to load books.</p>;
}
```

## Conclusion
Apollo GraphQL's core concepts offer a powerful way to define, fetch, and manage data in modern applications. By using a schema-driven approach, resolvers, and a flexible client-server communication model, Apollo provides a seamless development experience for both frontend and backend developers.
