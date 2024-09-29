---
icon: material/alert-circle
---

# Apollo Error Handling and Debugging

Error handling and debugging are crucial aspects of any GraphQL application. Apollo provides built-in mechanisms to capture, manage, and resolve errors both on the client and server sides. Apollo Client and Apollo Server offer various tools for detecting, logging, and handling GraphQL and network errors.

This section will cover how to handle errors gracefully in Apollo Client and Apollo Server, as well as best practices for debugging GraphQL queries and mutations.

## 1. Error Handling in Apollo Server

Apollo Server provides centralized error handling for GraphQL operations. Errors can be thrown in resolvers, and Apollo Server will format and return them to the client in the response.

### Throwing Errors in Resolvers

You can throw custom errors in your resolver logic if something goes wrong.

```javascript
const resolvers = {
  Query: {
    book: (parent, args, context) => {
      const book = getBookById(args.id);
      if (!book) {
        throw new Error('Book not found');
      }
      return book;
    },
  },
};
```

In this example, if the `getBookById` function doesn’t find the book, the resolver throws an error, which is then propagated to the client.

### Custom Error Formatting

You can customize the format of errors returned to the client by using the `formatError` option in Apollo Server.

```javascript
const server = new ApolloServer({
  typeDefs,
  resolvers,
  formatError: (err) => {
    // Customize error format
    return {
      message: err.message,
      code: err.extensions.code,
      path: err.path,
    };
  },
});
```

This allows you to control the structure of error messages sent back to the client, making them easier to debug and interpret.

## 2. Error Handling in Apollo Client

Apollo Client provides mechanisms for handling GraphQL errors, network errors, and cache errors. You can handle errors directly in your component code or use Apollo Link to manage errors globally.

### Handling Errors in Components

You can easily catch and display errors using the `useQuery` or `useMutation` hooks.

```javascript
const { loading, error, data } = useQuery(GET_BOOKS);

if (loading) return <p>Loading...</p>;
if (error) return <p>Error: {error.message}</p>;

return data.books.map(book => (
  <div key={book.id}>{book.title} by {book.author}</div>
));
```

In this example, the `error` object contains the error message returned from the server, which is displayed to the user.

### Global Error Handling with Apollo Link

Apollo Link allows you to handle errors globally across your application by using the `onError` function from `apollo-link-error`.

#### Example of Global Error Handling:

```javascript
import { ApolloClient, InMemoryCache, HttpLink, ApolloLink } from '@apollo/client';
import { onError } from '@apollo/client/link/error';

const errorLink = onError(({ graphQLErrors, networkError }) => {
  if (graphQLErrors) {
    graphQLErrors.forEach(({ message, locations, path }) =>
      console.log(`[GraphQL error]: Message: ${message}, Location: ${locations}, Path: ${path}`)
    );
  }

  if (networkError) {
    console.log(`[Network error]: ${networkError}`);
  }
});

const client = new ApolloClient({
  link: ApolloLink.from([errorLink, new HttpLink({ uri: '/graphql' })]),
  cache: new InMemoryCache(),
});
```

### Error Policies in Apollo Client

Apollo Client supports several error policies that determine how errors are handled during query or mutation execution.

- **none** (default): The query or mutation fails on any error.
- **all**: The query or mutation succeeds, and both data and errors are returned.
- **ignore**: Ignores errors and only returns successful data.

#### Example of Error Policy Usage:

```javascript
const { data, errors } = useQuery(GET_BOOKS, {
  errorPolicy: "all",  // Continue on error, return both data and errors
});
```

## 3. Apollo DevTools

Apollo provides a set of development tools that help you inspect, debug, and optimize your GraphQL queries and mutations. Apollo Client DevTools is a browser extension available for Chrome and Firefox that integrates directly with Apollo Client.

### Features of Apollo Client DevTools:

1. **GraphiQL Explorer**: Easily explore your GraphQL schema and test queries and mutations.
2. **Query Inspector**: View all active queries and mutations in your application, including their loading state and any errors that occur.
3. **Cache Inspector**: Inspect the Apollo cache, check which data is stored, and see how the cache changes after queries or mutations.

### Installation:

You can install Apollo Client DevTools as a browser extension from the Chrome or Firefox extension stores.

## 4. Debugging Apollo Server

Apollo Server provides detailed error messages and stack traces during development to help you debug issues in your GraphQL API. When deploying in production, you can suppress stack traces and customize error messages to avoid exposing sensitive information.

### Enabling Detailed Error Messages in Development:

During development, Apollo Server automatically provides detailed error messages and stack traces for easier debugging. However, you can also enable or disable these messages manually.

```javascript
const server = new ApolloServer({
  typeDefs,
  resolvers,
  debug: true,  // Enable detailed error messages
});
```

In production, it’s recommended to set `debug: false` to prevent exposing sensitive details about your server’s internals.

### Logging Errors in Apollo Server:

You can use middleware like `morgan` or `winston` to log errors in Apollo Server. This is especially useful for tracking errors in production environments.

```javascript
const server = new ApolloServer({
  typeDefs,
  resolvers,
  formatError: (err) => {
    console.error(err);  // Log the error to the console or an external service
    return err;
  },
});
```

## 5. Best Practices for Error Handling

1. **Catch and Display Errors in the UI**: Always display meaningful error messages to users in case something goes wrong, instead of leaving them in the dark.
2. **Use Global Error Handling**: Implement centralized error handling with Apollo Link to log or handle errors globally across your application.
3. **Leverage Apollo DevTools**: Use Apollo DevTools to inspect queries, cache, and any issues with your GraphQL API in real-time.
4. **Handle Network and GraphQL Errors Separately**: Network errors (e.g., connection issues) should be handled differently from GraphQL execution errors (e.g., failed resolvers).

## Conclusion

Apollo provides powerful tools for handling and debugging errors both on the client and server sides. By leveraging error policies, global error handling, and tools like Apollo DevTools, developers can efficiently track and resolve issues in their GraphQL applications. Ensuring that errors are handled gracefully improves both the user experience and the maintainability of the application.
