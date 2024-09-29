---
icon: material/speedometer
---

# Apollo Performance Optimization

Optimizing the performance of your Apollo GraphQL server and client is essential for building efficient, scalable applications. GraphQL’s flexibility, while beneficial, can lead to performance challenges such as over-fetching or under-fetching of data, inefficient resolvers, and high network overhead. Apollo provides several tools and techniques to improve the performance of your GraphQL APIs.

This section covers best practices and techniques for optimizing the performance of Apollo Server and Apollo Client.

## 1. Query Optimization in Apollo Client

### 1.1. Avoid Over-fetching Data

One of the key advantages of GraphQL is the ability to specify exactly which data fields are needed in a query. By only requesting the necessary data, you can reduce the payload size and avoid over-fetching, improving both network performance and client efficiency.

#### Example of Efficient Query:

```graphql
query GetBookTitles {
  books {
    id
    title  # Only fetching necessary fields (title and id)
  }
}
```

In this example, only the `id` and `title` fields of the `books` are fetched, instead of retrieving the entire object.

### 1.2. Use Apollo Client Caching

Apollo Client provides an `InMemoryCache` that reduces the need for repeated network requests by caching query results. Using caching effectively can significantly improve performance by reducing data fetching over the network.

### Cache-and-Network Fetch Policy:

To ensure you have the freshest data while leveraging the cache, use the `cache-and-network` fetch policy.

```javascript
const { data, loading } = useQuery(GET_BOOKS, {
  fetchPolicy: "cache-and-network",
});
```

With this fetch policy, Apollo Client returns cached data while sending a network request to fetch updated data.

### 1.3. Use Query Deduplication

Apollo Client automatically deduplicates queries, meaning if the same query is sent multiple times, Apollo will only make one network request. Ensure that query deduplication is enabled (it is by default).

## 2. Batch Requests and Persisted Queries

### 2.1. Batch HTTP Requests

Apollo Client allows you to batch multiple GraphQL queries into a single HTTP request. This can reduce the overhead of making multiple requests, especially in situations where the client sends several queries at once.

```javascript
import { ApolloClient, InMemoryCache, ApolloLink, HttpLink } from '@apollo/client';
import { BatchHttpLink } from "@apollo/client/link/batch-http";

const client = new ApolloClient({
  link: ApolloLink.from([
    new BatchHttpLink({ uri: 'http://localhost:4000/graphql', batchMax: 10, batchInterval: 20 }),
  ]),
  cache: new InMemoryCache(),
});
```

In this example, multiple queries are batched into a single HTTP request, improving network efficiency.

### 2.2. Use Persisted Queries

Persisted queries are a performance optimization technique where query strings are stored on the server, and the client sends only a unique query hash rather than the full query string. This reduces the size of requests and improves performance, especially for mobile clients with limited bandwidth.

#### Enabling Persisted Queries:

```bash
npm install @apollo/client @apollo/link-persisted-queries
```

```javascript
import { createPersistedQueryLink } from "@apollo/client/link/persisted-queries";
import { ApolloClient, InMemoryCache, HttpLink } from "@apollo/client";

const client = new ApolloClient({
  link: createPersistedQueryLink().concat(new HttpLink({ uri: 'http://localhost:4000/graphql' })),
  cache: new InMemoryCache(),
});
```

## 3. Optimizing Apollo Server Performance

### 3.1. Use DataLoader for Batch and Cache

In GraphQL, it’s common to encounter the **N+1 problem**, where fetching a list of related objects results in multiple database queries. DataLoader is a utility that helps batch and cache requests to avoid redundant queries.

#### Example of DataLoader:

```javascript
const DataLoader = require('dataloader');

const bookLoader = new DataLoader(keys => fetchBooksByIds(keys));

const resolvers = {
  Query: {
    books: (parent, args, context) => bookLoader.loadMany(args.ids),
  },
};
```

DataLoader batches and caches database requests to minimize redundant fetches, improving the overall performance of your resolvers.

### 3.2. Use Apollo Federation for Scalability

Apollo Federation allows you to split your GraphQL services into smaller, manageable subgraphs that can be deployed independently. This modular approach can help improve the performance of your GraphQL services by reducing the load on individual services.

### 3.3. Use GraphQL Directives for Performance Control

Custom directives can be used to optimize query performance. For example, you could create a directive to limit the number of results returned by a query or implement rate limiting on specific fields.

#### Example of a Custom Directive for Limiting Results:

```javascript
class LimitResultsDirective extends SchemaDirectiveVisitor {
  visitFieldDefinition(field) {
    const { resolve = defaultFieldResolver } = field;
    field.resolve = async function (...args) {
      const result = await resolve.apply(this, args);
      return result.slice(0, args[1].limit);
    };
  }
}
```

### 3.4. Apollo Cache Control

Apollo Server provides a cache control extension that allows you to define caching hints for individual fields in your schema, helping reduce the load on your resolvers.

#### Example of Cache Control Configuration:

```javascript
const server = new ApolloServer({
  typeDefs,
  resolvers,
  cacheControl: {
    defaultMaxAge: 5,  // Cache responses for 5 seconds by default
  },
});
```

Fields in the schema can also be individually configured to have specific cache hints:

```javascript
const resolvers = {
  Query: {
    book: (parent, args, context) => {
      context.cacheControl.setCacheHint({ maxAge: 60 });  // Cache this field for 60 seconds
      return getBook(args.id);
    },
  },
};
```

### 3.5. Limit Query Depth

To prevent abuse and protect your server from overly complex queries, you can limit the query depth. This ensures that users can’t send deeply nested queries that may degrade performance.

```bash
npm install graphql-depth-limit
```

```javascript
const depthLimit = require('graphql-depth-limit');

const server = new ApolloServer({
  typeDefs,
  resolvers,
  validationRules: [depthLimit(5)],  // Limit query depth to 5
});
```

## 4. Monitoring and Performance Metrics

### 4.1. Apollo Studio for Performance Monitoring

Apollo Studio provides powerful tools for monitoring the performance of your GraphQL API. It allows you to track query performance, error rates, resolver timing, and more.

#### Enabling Apollo Studio:

```javascript
const { ApolloServerPluginLandingPageGraphQLPlayground } = require('apollo-server-core');

const server = new ApolloServer({
  typeDefs,
  resolvers,
  plugins: [
    ApolloServerPluginLandingPageGraphQLPlayground({
      graphRef: "your-graph-id@current",
      key: "your-apollo-key",
    }),
  ],
});
```

Apollo Studio helps you track real-time metrics, allowing you to identify slow queries, bottlenecks, and areas for improvement.

### 4.2. Use Apollo Server Tracing

Apollo Server has built-in support for tracing, which provides detailed information about query execution times and resolver performance.

To enable tracing in Apollo Server:

```javascript
const server = new ApolloServer({
  typeDefs,
  resolvers,
  tracing: true,  // Enable tracing for performance insights
});
```

Tracing provides a breakdown of how long each resolver takes to execute, helping you pinpoint slow resolvers or inefficient queries.

## Conclusion

Optimizing the performance of your Apollo GraphQL API is essential for ensuring a smooth user experience and scalability. By leveraging caching, batching, persisted queries, DataLoader, and Apollo Federation, you can minimize network overhead and improve resolver efficiency. Additionally, tools like Apollo Studio and server tracing provide valuable insights into the performance of your API, allowing you to continuously monitor and improve your GraphQL operations.
