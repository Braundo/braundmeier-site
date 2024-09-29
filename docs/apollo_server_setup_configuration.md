---
icon: material/server
---

# Apollo Server Setup and Configuration

Apollo Server is a highly flexible and customizable GraphQL server that can be easily integrated into any Node.js-based project. This section covers step-by-step instructions for setting up and configuring Apollo Server, including handling queries, mutations, authentication, and data sources.

## 1. Installing Apollo Server

To get started with Apollo Server, you'll need to install two main packages: `apollo-server` and `graphql`.

### Installation Command:
```bash
npm install apollo-server graphql
```

This will install the Apollo Server package, which provides the tools necessary to create a GraphQL server, and the `graphql` package, which is the core GraphQL engine.

## 2. Basic Apollo Server Setup

Here’s a basic example of how to set up Apollo Server with a simple GraphQL schema.

### Example Schema:
```graphql
type Query {
  books: [Book]
}

type Book {
  title: String
  author: String
}
```

### Resolvers:
```javascript
const resolvers = {
  Query: {
    books: () => [
      { title: '1984', author: 'George Orwell' },
      { title: 'Brave New World', author: 'Aldous Huxley' },
    ],
  },
};
```

### Starting the Server:
```javascript
const { ApolloServer } = require('apollo-server');

const typeDefs = gql`
  type Query {
    books: [Book]
  }

  type Book {
    title: String
    author: String
  }
`;

const resolvers = {
  Query: {
    books: () => [
      { title: '1984', author: 'George Orwell' },
      { title: 'Brave New World', author: 'Aldous Huxley' },
    ],
  },
};

const server = new ApolloServer({ typeDefs, resolvers });

server.listen().then(({ url }) => {
  console.log(`🚀 Server ready at ${url}`);
});
```

This will start a server on `http://localhost:4000/graphql`. The server is ready to receive and execute GraphQL queries.

## 3. Handling Queries and Mutations

### Queries:
In GraphQL, queries are used to fetch data. The `Query` type defines the structure of all possible queries. For example, fetching all books:

```graphql
query {
  books {
    title
    author
  }
}
```

The corresponding resolver for this query will return a static list of books in our simple setup.

### Mutations:
Mutations allow you to modify data on the server. Mutations can create, update, or delete resources.

### Example Mutation:
```graphql
type Mutation {
  addBook(title: String, author: String): Book
}
```

### Resolver for the Mutation:
```javascript
const resolvers = {
  Mutation: {
    addBook: (_, { title, author }) => {
      const newBook = { title, author };
      books.push(newBook);  // Assuming `books` is an array storing the books.
      return newBook;
    },
  },
};
```

Now, you can execute a mutation like this:
```graphql
mutation {
  addBook(title: "The Great Gatsby", author: "F. Scott Fitzgerald") {
    title
    author
  }
}
```

## 4. Adding Authentication to Apollo Server

Authentication is a critical part of most applications. You can add authentication to your Apollo Server by passing the request object into the `context` function.

### Example of Adding Authentication:
```javascript
const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ req }) => {
    const token = req.headers.authorization || '';
    const user = getUserFromToken(token);  // Function that decodes the token and returns the user.
    return { user };
  },
});
```

In your resolvers, you can access the authenticated user:
```javascript
const resolvers = {
  Query: {
    books: (parent, args, context) => {
      if (!context.user) throw new Error('You must be logged in');
      return books;
    },
  },
};
```

## 5. Apollo Data Sources

Apollo Server provides a way to integrate different data sources (databases, REST APIs, etc.) into your GraphQL API. Apollo Data Sources are classes that encapsulate fetching logic for these data sources.

### Example of a REST Data Source:
```javascript
const { RESTDataSource } = require('apollo-datasource-rest');

class BooksAPI extends RESTDataSource {
  constructor() {
    super();
    this.baseURL = 'https://example.com/api/';
  }

  async getBook(bookId) {
    return this.get(`books/${bookId}`);
  }
}

const server = new ApolloServer({
  typeDefs,
  resolvers,
  dataSources: () => ({
    booksAPI: new BooksAPI(),
  }),
});
```

You can now access `booksAPI` in your resolvers:
```javascript
const resolvers = {
  Query: {
    book: (_, { id }, { dataSources }) => dataSources.booksAPI.getBook(id),
  },
};
```

## 6. Subscriptions with Apollo Server

Apollo Server supports subscriptions, which allow real-time updates over WebSockets. Subscriptions are used for real-time applications like chat apps, live dashboards, and more.

### Example Subscription:
```graphql
type Subscription {
  bookAdded: Book
}
```

### Setting up the Subscription Resolver:
```javascript
const { PubSub } = require('apollo-server');
const pubsub = new PubSub();

const resolvers = {
  Subscription: {
    bookAdded: {
      subscribe: () => pubsub.asyncIterator(['BOOK_ADDED']),
    },
  },
  Mutation: {
    addBook: (parent, { title, author }) => {
      const newBook = { title, author };
      books.push(newBook);
      pubsub.publish('BOOK_ADDED', { bookAdded: newBook });
      return newBook;
    },
  },
};
```

In this example, every time a new book is added, the `bookAdded` subscription will notify all clients listening to that event.

## 7. Apollo Server with Express

Apollo Server can be used with any HTTP framework, such as Express, to provide more flexibility in your server setup.

### Example Setup with Express:
```javascript
const express = require('express');
const { ApolloServer } = require('apollo-server-express');

const server = new ApolloServer({ typeDefs, resolvers });

const app = express();
server.applyMiddleware({ app });

app.listen({ port: 4000 }, () =>
  console.log(`🚀 Server ready at http://localhost:4000${server.graphqlPath}`)
);
```

This setup allows you to integrate GraphQL with existing Express middleware, route handling, or even serve static assets alongside your GraphQL API.

## Conclusion

Apollo Server is a powerful and flexible GraphQL server that integrates seamlessly with various data sources, authentication mechanisms, and even WebSocket subscriptions for real-time data. This setup guide provides a foundation for building scalable GraphQL APIs with Apollo Server.
