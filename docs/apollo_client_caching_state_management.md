---
icon: material/cached
---

# Apollo Client Caching and State Management

Apollo Client offers robust caching and state management features, making it a powerful tool for managing data in modern web and mobile applications. Its in-memory caching system helps minimize network requests and improve the performance of applications by storing query results and local state efficiently.

In this section, we'll explore the various aspects of Apollo Client's caching and state management, how to configure cache policies, and handle both local and remote data seamlessly.

## 1. InMemoryCache

At the core of Apollo Client's caching system is the `InMemoryCache`, which stores query results in memory and uses that data to satisfy future queries. It normalizes the cache by storing objects using unique IDs and only refetching data when necessary.

### Basic Cache Configuration

```javascript
import { ApolloClient, InMemoryCache } from '@apollo/client';

const client = new ApolloClient({
  uri: 'https://example.com/graphql',
  cache: new InMemoryCache(),
});
```

The `InMemoryCache` stores objects by their unique identifiers, which helps Apollo efficiently manage and retrieve cached data. This unique ID is generated using the `__typename` and `id` fields of the GraphQL objects.

### Custom Cache Keying with TypePolicies

By default, Apollo Client uses the `__typename` and `id` fields to uniquely identify cached objects. However, you can customize this behavior using **type policies**.

#### Example of Type Policies:

```javascript
const cache = new InMemoryCache({
  typePolicies: {
    Book: {
      keyFields: ["isbn"],  // Use ISBN as the unique identifier for Book objects
    },
  },
});
```

In this example, the `Book` type uses the `isbn` field as its unique key instead of the default `id`.

## 2. Caching Queries

Apollo Client's caching mechanism ensures that when a query is executed, the results are stored in the cache. If the same query is executed again, Apollo will return the data from the cache instead of making a new network request.

### Example of Query Caching:

```javascript
const GET_BOOKS = gql`
  query GetBooks {
    books {
      id
      title
      author
    }
  }
`;

function BookList() {
  const { loading, error, data } = useQuery(GET_BOOKS, {
    fetchPolicy: "cache-first",  // Use cached data if available
  });

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <ul>
      {data.books.map(book => (
        <li key={book.id}>
          {book.title} by {book.author}
        </li>
      ))}
    </ul>
  );
}
```

In this example, the query uses a `fetchPolicy` of `"cache-first"`, which means Apollo Client will return cached data if available, and only make a network request if the data is not in the cache.

## 3. Cache Fetch Policies

Apollo Client offers several cache fetch policies that determine how data is fetched and when the cache is used:

- **cache-first** (default): Tries to read data from the cache first, falls back to a network request if the data is not available.
- **cache-only**: Only reads data from the cache and never makes a network request.
- **network-only**: Always makes a network request, ignoring any cached data.
- **no-cache**: Does not store or reuse any query results.
- **cache-and-network**: Returns cached data (if available) while also sending a network request to refresh the data.

### Example of Using Different Fetch Policies:

```javascript
const { loading, data } = useQuery(GET_BOOKS, {
  fetchPolicy: "network-only",  // Always fetch data from the network
});
```

You can choose the appropriate fetch policy depending on your app’s needs.

## 4. Updating the Cache after a Mutation

Apollo Client provides ways to manually update the cache after performing a mutation to ensure that the UI remains consistent with the server state. This is important when adding or removing items, as the cache may need to be updated to reflect the changes.

### Example of Cache Update after Mutation:

```javascript
const ADD_BOOK = gql`
  mutation AddBook($title: String!, $author: String!) {
    addBook(title: $title, author: $author) {
      id
      title
      author
    }
  }
`;

const GET_BOOKS = gql`
  query GetBooks {
    books {
      id
      title
      author
    }
  }
`;

const [addBook] = useMutation(ADD_BOOK, {
  update(cache, { data: { addBook } }) {
    const { books } = cache.readQuery({ query: GET_BOOKS });
    cache.writeQuery({
      query: GET_BOOKS,
      data: { books: [...books, addBook] },
    });
  },
});
```

In this example, after a new book is added via the `addBook` mutation, the `GET_BOOKS` query is updated in the cache to include the new book.

## 5. Managing Local State with Apollo Client

In addition to managing remote data from a GraphQL API, Apollo Client can also manage local application state. This means you can store and query client-side state using the same GraphQL queries and mutations used for server data.

### Defining Local State

To manage local state, Apollo Client uses reactive variables (`makeVar`) or the Apollo cache to store and modify local data.

#### Example of Local State with `makeVar`:

```javascript
import { makeVar, gql, useQuery } from '@apollo/client';

// Define a local reactive variable
const isLoggedInVar = makeVar(false);

// Query local state
const IS_LOGGED_IN = gql`
  query IsUserLoggedIn {
    isLoggedIn @client
  }
`;

function LoginStatus() {
  const { data } = useQuery(IS_LOGGED_IN);
  return <p>{data.isLoggedIn ? "Logged in" : "Logged out"}</p>;
}

// Modify local state
function toggleLogin() {
  isLoggedInVar(!isLoggedInVar());
}
```

In this example, `makeVar` creates a local variable, `isLoggedInVar`, that can be used to store and modify the user’s login status. The local state is queried using `@client` in the query, and the UI is updated when the state changes.

## 6. Apollo Client Cache Eviction and Garbage Collection

Apollo Client automatically handles cache eviction and garbage collection to free up memory when cached data is no longer needed.

### Evicting Data from the Cache:

You can manually evict specific data from the cache using the `cache.evict` method.

```javascript
client.cache.evict({ id: 'Book:1' });
```

This will remove the `Book` object with ID `1` from the cache.

### Garbage Collection:

Garbage collection in Apollo Client automatically removes orphaned objects from the cache. After evicting data, you can trigger garbage collection manually to clean up the cache.

```javascript
client.cache.gc();
```

## 7. Optimistic UI Updates

Optimistic UI updates allow you to immediately reflect changes in the UI before the server responds to a mutation. This improves user experience by reducing the perceived delay between an action and its result.

### Example of Optimistic UI:

```javascript
const [addBook] = useMutation(ADD_BOOK, {
  optimisticResponse: {
    __typename: "Mutation",
    addBook: {
      __typename: "Book",
      id: -1,  // Temporary ID
      title: "Temporary Title",
      author: "Temporary Author"
    }
  },
  update(cache, { data: { addBook } }) {
    const { books } = cache.readQuery({ query: GET_BOOKS });
    cache.writeQuery({
      query: GET_BOOKS,
      data: { books: [...books, addBook] },
    });
  }
});
```

In this example, the UI immediately reflects the newly added book using a temporary optimistic response. Once the mutation completes and the server response is received, the cache is updated with the actual data.

## Conclusion

Apollo Client’s caching and state management system is highly flexible and powerful, enabling you to efficiently manage both remote and local data. With built-in features like `InMemoryCache`, reactive variables, optimistic UI updates, and cache eviction, Apollo Client helps developers build performant and maintainable applications with ease.
