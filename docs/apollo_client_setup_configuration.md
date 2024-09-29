---
icon: material/hammer-wrench
---

# Apollo Client Setup and Configuration

Apollo Client is a comprehensive state management library that helps you fetch, cache, and manage local and remote data in modern applications. It’s designed to work seamlessly with GraphQL APIs, and integrates with popular frontend frameworks such as React, Angular, and Vue.

In this section, we’ll cover how to set up Apollo Client, configure it to work with your GraphQL API, and integrate it with frontend frameworks.

## 1. Installing Apollo Client

To start using Apollo Client, you'll need to install the core package, as well as the GraphQL package that Apollo Client depends on.

### Installation Command:
```bash
npm install @apollo/client graphql
```

## 2. Basic Apollo Client Setup

The Apollo Client instance is responsible for interacting with the GraphQL API. It also handles caching, error handling, and more. Let’s go through setting up a basic Apollo Client instance.

### Step 1: Create an Apollo Client Instance

```javascript
import { ApolloClient, InMemoryCache } from '@apollo/client';

const client = new ApolloClient({
  uri: 'http://localhost:4000/graphql',  // Replace with your GraphQL API endpoint
  cache: new InMemoryCache()              // Apollo’s default caching strategy
});
```

- `uri`: The URL where your GraphQL API is hosted.
- `cache`: Apollo Client’s `InMemoryCache` helps in optimizing query performance by caching query results.

### Step 2: Provide the Client to Your Application

To make Apollo Client available throughout your app, wrap your root component with the `ApolloProvider` component. This works similarly to React's `Context.Provider`.

```javascript
import { ApolloProvider } from '@apollo/client';
import ReactDOM from 'react-dom';
import App from './App';

ReactDOM.render(
  <ApolloProvider client={client}>
    <App />
  </ApolloProvider>,
  document.getElementById('root')
);
```

By using `ApolloProvider`, any component within your React app can execute GraphQL queries and mutations using Apollo Client.

## 3. Fetching Data Using `useQuery`

Apollo Client’s `useQuery` hook allows you to fetch data from your GraphQL API and use it in your component. Let’s look at a simple example.

### Example Query:

```graphql
query GetBooks {
  books {
    title
    author
  }
}
```

### React Component Using `useQuery`:

```javascript
import { useQuery, gql } from '@apollo/client';

const GET_BOOKS = gql`
  query GetBooks {
    books {
      title
      author
    }
  }
`;

function BookList() {
  const { loading, error, data } = useQuery(GET_BOOKS);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <ul>
      {data.books.map((book) => (
        <li key={book.title}>
          {book.title} by {book.author}
        </li>
      ))}
    </ul>
  );
}
```

### Explanation:
- `useQuery`: A React hook provided by Apollo Client that is used to execute queries and fetch data.
- `loading`: Boolean indicating if the request is still in progress.
- `error`: Captures any errors that occur during the query execution.
- `data`: The response data returned from the GraphQL server.

## 4. Mutating Data with `useMutation`

Mutations in Apollo Client are used to create, update, or delete data. The `useMutation` hook is used for this purpose.

### Example Mutation:

```graphql
mutation AddBook($title: String!, $author: String!) {
  addBook(title: $title, author: $author) {
    title
    author
  }
}
```

### Using `useMutation` in React:

```javascript
import { useMutation, gql } from '@apollo/client';

const ADD_BOOK = gql`
  mutation AddBook($title: String!, $author: String!) {
    addBook(title: $title, author: $author) {
      title
      author
    }
  }
`;

function AddBookForm() {
  let titleInput, authorInput;
  const [addBook, { data }] = useMutation(ADD_BOOK);

  return (
    <div>
      <form
        onSubmit={e => {
          e.preventDefault();
          addBook({ variables: { title: titleInput.value, author: authorInput.value } });
          titleInput.value = '';
          authorInput.value = '';
        }}
      >
        <input placeholder="Title" ref={node => { titleInput = node; }} />
        <input placeholder="Author" ref={node => { authorInput = node; }} />
        <button type="submit">Add Book</button>
      </form>
    </div>
  );
}
```

### Explanation:
- `useMutation`: A hook that triggers a mutation to modify the backend data.
- The `addBook` function is called when the form is submitted, sending the input values as arguments to the GraphQL mutation.

## 5. Apollo Client Caching

One of Apollo Client’s strongest features is its caching system. The `InMemoryCache` allows Apollo to cache query results so that they can be reused without needing to make another network request. It also allows for automatic cache updates when new data is added.

### Example of Manual Cache Updates:

If you add a new book to your system, you may want to update the cache to include the new book in your list.

```javascript
const [addBook] = useMutation(ADD_BOOK, {
  update(cache, { data: { addBook } }) {
    cache.modify({
      fields: {
        books(existingBooks = []) {
          return [...existingBooks, addBook];
        },
      },
    });
  },
});
```

This ensures that the cache is immediately updated with the new book so that the UI reflects the change without needing to refetch the data.

## 6. Apollo DevTools

Apollo provides a set of development tools that help you debug your GraphQL queries and mutations. You can install Apollo Client DevTools in your browser, which integrates directly with Apollo Client to inspect queries, mutations, and cache.

### Features of Apollo DevTools:
- **View Queries and Mutations**: See all the queries and mutations being sent from the client to the server.
- **Inspect Cache**: View the current state of Apollo Client’s cache and check how it is updated after mutations.
- **GraphiQL Integration**: Easily explore your API with an embedded GraphiQL query editor.

You can install the Apollo Client DevTools as a browser extension from the Chrome or Firefox extension stores.

## 7. Error Handling in Apollo Client

Apollo Client provides powerful error-handling features. You can handle errors directly within your components or globally across your app using Apollo Link.

### Error Handling in a Component:

```javascript
const { loading, error, data } = useQuery(GET_BOOKS);

if (loading) return <p>Loading...</p>;
if (error) return <p>Error: {error.message}</p>;

return (
  <ul>
    {data.books.map((book) => (
      <li key={book.title}>{book.title}</li>
    ))}
  </ul>
);
```

### Global Error Handling with Apollo Link:

Apollo Link provides more control over error handling. Using `onError`, you can define a centralized error-handling mechanism.

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
  link: ApolloLink.from([errorLink, new HttpLink({ uri: 'http://localhost:4000/graphql' })]),
  cache: new InMemoryCache(),
});
```

This error link will capture all errors globally across the app, allowing you to log or display them as necessary.

## Conclusion

Apollo Client is a powerful and flexible tool that integrates easily with modern frontend frameworks. With its efficient caching system, built-in error handling, and flexible state management, Apollo Client helps simplify the process of working with GraphQL APIs in your application. Whether you’re building simple apps or complex systems, Apollo Client provides the tools you need for efficient and maintainable data management.
