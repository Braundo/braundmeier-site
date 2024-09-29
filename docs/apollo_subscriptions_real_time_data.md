---
icon: material/timetable
---

# Apollo Subscriptions and Real-Time Data

Subscriptions in Apollo allow real-time communication between the client and server. This is especially useful for applications like chat apps, live dashboards, collaborative tools, and more. Subscriptions are typically implemented using WebSockets, which allow the server to push updates to the client when certain events occur.

In this section, we’ll cover how to set up and use Apollo Subscriptions to handle real-time data in your GraphQL applications.

## 1. What are Subscriptions?

In GraphQL, subscriptions allow clients to listen for specific events on the server and receive updates when those events occur. Unlike queries and mutations, which are one-time operations, subscriptions keep an open connection between the client and the server, pushing updates as they happen.

### Example of Subscription:

```graphql
subscription {
  bookAdded {
    title
    author
  }
}
```

This subscription listens for the event `bookAdded` and returns the title and author of any book that gets added to the system.

## 2. Setting Up Apollo Server for Subscriptions

Apollo Server uses WebSockets to support GraphQL subscriptions. To enable subscriptions, you’ll need to configure the server to use `apollo-server` and `subscriptions-transport-ws`.

### Step-by-Step Setup:

1. **Install Required Packages**:

```bash
npm install apollo-server subscriptions-transport-ws graphql
```

2. **Set Up Apollo Server with WebSocket Support**:

Here’s a basic setup of Apollo Server with WebSocket support for subscriptions:

```javascript
const { ApolloServer, gql, PubSub } = require('apollo-server');
const { createServer } = require('http');
const { SubscriptionServer } = require('subscriptions-transport-ws');
const { execute, subscribe } = require('graphql');
const { makeExecutableSchema } = require('@graphql-tools/schema');

const pubsub = new PubSub();

// Define the schema with subscription support
const typeDefs = gql`
  type Book {
    title: String
    author: String
  }

  type Query {
    books: [Book]
  }

  type Mutation {
    addBook(title: String, author: String): Book
  }

  type Subscription {
    bookAdded: Book
  }
`;

// Define resolvers for queries, mutations, and subscriptions
const resolvers = {
  Query: {
    books: () => [{ title: '1984', author: 'George Orwell' }],
  },
  Mutation: {
    addBook: (_, { title, author }) => {
      const newBook = { title, author };
      pubsub.publish('BOOK_ADDED', { bookAdded: newBook });
      return newBook;
    },
  },
  Subscription: {
    bookAdded: {
      subscribe: () => pubsub.asyncIterator(['BOOK_ADDED']),
    },
  },
};

// Create executable schema
const schema = makeExecutableSchema({ typeDefs, resolvers });

// Start Apollo Server with HTTP and WebSocket support
const server = new ApolloServer({ schema });

const httpServer = createServer(server);

server.listen().then(({ url }) => {
  console.log(`🚀 Server ready at ${url}`);
  new SubscriptionServer(
    {
      execute,
      subscribe,
      schema,
    },
    {
      server: httpServer,
      path: server.graphqlPath,
    }
  );
});
```

### Explanation:
- **PubSub**: This is an in-memory event bus provided by Apollo Server to publish and subscribe to events. It’s useful for local development but in production, you should use a more robust solution like Redis PubSub.
- **SubscriptionServer**: Handles WebSocket connections and listens for subscription events.

This server will allow clients to subscribe to the `bookAdded` event and receive real-time updates whenever a new book is added via the `addBook` mutation.

## 3. Implementing Subscriptions in Apollo Client

Apollo Client allows you to use subscriptions to receive real-time updates from the server. You’ll need to configure WebSocket support on the client as well.

### Step-by-Step Setup:

1. **Install WebSocket Client Packages**:

```bash
npm install @apollo/client subscriptions-transport-ws graphql
```

2. **Set Up Apollo Client with WebSocket Support**:

In this setup, we use the `WebSocketLink` to connect Apollo Client to the server’s WebSocket endpoint.

```javascript
import { ApolloClient, InMemoryCache, ApolloProvider, split } from '@apollo/client';
import { WebSocketLink } from 'apollo-link-ws';
import { HttpLink } from 'apollo-link-http';
import { getMainDefinition } from '@apollo/client/utilities';

// Set up HTTP link for queries and mutations
const httpLink = new HttpLink({
  uri: 'http://localhost:4000/graphql',
});

// Set up WebSocket link for subscriptions
const wsLink = new WebSocketLink({
  uri: `ws://localhost:4000/graphql`,
  options: {
    reconnect: true,
  },
});

// Split links: Use WebSocket link for subscriptions, and HTTP link for queries and mutations
const link = split(
  ({ query }) => {
    const definition = getMainDefinition(query);
    return (
      definition.kind === 'OperationDefinition' &&
      definition.operation === 'subscription'
    );
  },
  wsLink,
  httpLink
);

// Initialize Apollo Client with both links
const client = new ApolloClient({
  link,
  cache: new InMemoryCache(),
});
```

This configuration uses the `split` function to route subscription operations through the WebSocket link, while queries and mutations go through the HTTP link.

## 4. Using Subscriptions in React Components

You can use Apollo Client’s `useSubscription` hook to listen for real-time updates in your React components.

### Example React Component Using `useSubscription`:

```javascript
import { useSubscription, gql } from '@apollo/client';

const BOOK_ADDED_SUBSCRIPTION = gql`
  subscription OnBookAdded {
    bookAdded {
      title
      author
    }
  }
`;

function BookList() {
  const { data, loading } = useSubscription(BOOK_ADDED_SUBSCRIPTION);

  if (loading) return <p>Loading...</p>;

  return (
    <div>
      <h3>New Book Added</h3>
      <p>{data.bookAdded.title} by {data.bookAdded.author}</p>
    </div>
  );
}
```

This component listens for the `bookAdded` event and displays the new book’s information in real-time.

## 5. Real-World Use Cases for Subscriptions

### 1. **Chat Applications**: Subscriptions are ideal for chat applications where users need to receive new messages instantly.
### 2. **Live Dashboards**: You can build real-time dashboards where data updates live without requiring the user to refresh the page.
### 3. **Collaborative Tools**: For applications like Google Docs or Trello, where multiple users interact with shared data, subscriptions enable real-time collaboration.

## 6. Using Apollo Subscriptions with Redis PubSub

For production-grade applications, you may want to move away from the default in-memory `PubSub` and use a more scalable solution like Redis. Apollo supports Redis PubSub for distributed systems.

### Setup for Redis PubSub:

1. **Install Redis and Apollo Redis PubSub**:

```bash
npm install ioredis graphql-redis-subscriptions
```

2. **Configure Redis PubSub**:

```javascript
const Redis = require('ioredis');
const { RedisPubSub } = require('graphql-redis-subscriptions');

const options = {
  host: 'localhost',
  port: 6379,
  retryStrategy: times => Math.min(times * 50, 2000),
};

const pubsub = new RedisPubSub({
  publisher: new Redis(options),
  subscriber: new Redis(options),
});
```

With Redis, you can now scale your subscription service to multiple instances and maintain consistent event distribution across all your services.

## Conclusion

Apollo Subscriptions are a powerful feature for building real-time applications with GraphQL. Whether you’re building chat applications, live dashboards, or collaborative tools, Apollo Subscriptions enable you to deliver instant updates to your clients. By leveraging WebSockets, you can establish a persistent connection between the server and client, ensuring your application remains responsive and up-to-date with the latest data.
