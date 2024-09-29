---
icon: material/shield-lock
---

# Apollo Security and Authentication

Securing your GraphQL APIs is critical, especially when dealing with sensitive data. Apollo provides several tools and techniques to help you secure your API, handle authentication, and implement authorization effectively. In this section, we’ll explore how to set up authentication, manage user sessions, and protect your API endpoints.

## 1. Authentication in Apollo Server

Authentication is the process of verifying the identity of a user. In Apollo Server, authentication can be handled through HTTP headers (such as tokens or cookies) that are passed with each request.

### Setting Up Authentication in Apollo Server

You can add authentication to Apollo Server by using the `context` function, which provides a way to pass authentication information to your resolvers. Typically, you’ll check the request headers for a token, decode it, and include the user information in the `context`.

#### Example of Adding Authentication with JWT:

1. **Install JWT Package**:
```bash
npm install jsonwebtoken
```

2. **Add JWT Token Validation to the Context**:

```javascript
const jwt = require('jsonwebtoken');

const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ req }) => {
    // Get the token from the request headers
    const token = req.headers.authorization || '';
    
    // Verify and decode the token
    let user = null;
    if (token) {
      try {
        user = jwt.verify(token, process.env.JWT_SECRET);
      } catch (e) {
        console.error('Invalid token', e);
      }
    }
    
    // Add the user to the context
    return { user };
  },
});
```

In this setup, the `context` function extracts the JWT from the `Authorization` header, verifies it using a secret key (`JWT_SECRET`), and adds the authenticated user to the context.

3. **Accessing User in Resolvers**:
Once the user is added to the `context`, you can access the user in any resolver.

```javascript
const resolvers = {
  Query: {
    books: (parent, args, context) => {
      if (!context.user) {
        throw new Error('You must be logged in to view books');
      }
      return getBooks();  // A function that fetches books
    },
  },
};
```

In this example, only authenticated users can access the list of books.

## 2. Authorization in Apollo Server

While authentication verifies who the user is, **authorization** ensures that the user has the necessary permissions to access a particular resource or perform a certain action.

You can implement role-based or permission-based authorization in Apollo Server by checking the user’s role or permissions in the resolver.

### Example of Role-Based Authorization:

```javascript
const resolvers = {
  Mutation: {
    addBook: (parent, { title, author }, context) => {
      if (!context.user) {
        throw new Error('You must be logged in');
      }
      if (context.user.role !== 'admin') {
        throw new Error('You do not have permission to add books');
      }
      return addBookToDatabase(title, author);
    },
  },
};
```

In this example, only users with the `admin` role are allowed to add books. Non-admin users will receive an error if they attempt to perform this action.

## 3. Securing GraphQL Endpoints

### Using HTTPS

Always secure your GraphQL endpoints with HTTPS to ensure that the communication between the client and server is encrypted. You can easily configure your Apollo Server to work with HTTPS by using an external proxy (such as Nginx) or by providing your own SSL certificates.

### Rate Limiting

To prevent abuse and protect your GraphQL API from denial-of-service (DoS) attacks, implement rate limiting. You can use middleware like `express-rate-limit` to limit the number of requests a user can make within a given timeframe.

#### Example of Rate Limiting in Apollo Server:

```bash
npm install express-rate-limit
```

```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,  // Limit each IP to 100 requests per windowMs
});

app.use(limiter);  // Apply rate limiting to your server
```

This middleware limits clients to 100 requests every 15 minutes, helping to protect your server from excessive requests.

### Depth Limiting

GraphQL allows clients to specify deeply nested queries, which can lead to performance issues. To prevent abuse, you can enforce query depth limits using a package like `graphql-depth-limit`.

#### Example of Depth Limiting:

```bash
npm install graphql-depth-limit
```

```javascript
const depthLimit = require('graphql-depth-limit');

const server = new ApolloServer({
  typeDefs,
  resolvers,
  validationRules: [depthLimit(5)],  // Limit query depth to 5 levels
});
```

This configuration limits the depth of GraphQL queries to 5 levels, helping to prevent resource exhaustion from overly complex queries.

## 4. Using Apollo Server with OAuth

OAuth is another popular authentication method, especially for third-party services like Google, Facebook, and GitHub. You can integrate OAuth with Apollo Server by handling the OAuth flow on the client and passing the token to Apollo Server for verification.

### Example of Using OAuth with Apollo Server:

1. **Client-Side OAuth**:
On the client side, you’ll implement the OAuth flow using a library like `react-oauth` or handle it manually. Once the user is authenticated, you’ll receive an OAuth token from the provider (e.g., Google).

2. **Verify OAuth Token on the Server**:
On the server side, verify the token using the appropriate provider’s API (e.g., Google API).

```javascript
const { OAuth2Client } = require('google-auth-library');
const client = new OAuth2Client(CLIENT_ID);

async function verifyToken(token) {
  const ticket = await client.verifyIdToken({
    idToken: token,
    audience: CLIENT_ID,
  });
  const payload = ticket.getPayload();
  return payload;  // Contains the user's information
}

const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: async ({ req }) => {
    const token = req.headers.authorization || '';
    const user = await verifyToken(token);
    return { user };
  },
});
```

In this setup, the server verifies the OAuth token and adds the user information to the context.

## 5. Securing Apollo Subscriptions

If you’re using subscriptions with Apollo Server (which typically use WebSockets), you’ll need to secure the WebSocket connection to prevent unauthorized users from subscribing to sensitive data.

You can add authentication to WebSockets by passing the token during the connection initialization phase.

### Example of Securing Subscriptions:

```javascript
const { ApolloServer, PubSub } = require('apollo-server');
const { createServer } = require('http');
const { SubscriptionServer } = require('subscriptions-transport-ws');
const { execute, subscribe } = require('graphql');

const pubsub = new PubSub();

const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ connection }) => {
    // Connection contains the auth token
    if (connection) {
      const token = connection.context.authorization || '';
      const user = verifyToken(token);  // Verify token logic here
      return { user };
    }
  },
});

const httpServer = createServer(server);
server.installSubscriptionHandlers(httpServer);

httpServer.listen(4000, () => {
  console.log(`Server is running on http://localhost:4000`);
});
```

In this example, the WebSocket connection is secured by checking the token passed during the connection initialization.

## 6. Best Practices for Securing Apollo GraphQL APIs

1. **Use HTTPS**: Ensure all communication is encrypted using HTTPS.
2. **Implement Authentication**: Use tokens (JWT, OAuth) to authenticate users before granting access to your API.
3. **Limit Query Depth**: Enforce query depth limits to prevent complex, expensive queries from affecting performance.
4. **Rate Limiting**: Implement rate limiting to prevent abuse and ensure fair usage of your API.
5. **Use Schema Validation**: Validate your schema with tools like Apollo’s `graphql-tools` to ensure that invalid or malicious queries are caught early.
6. **Log Suspicious Activity**: Keep track of suspicious or abnormal activity, such as repeated query failures or high request rates.

## Conclusion

Security is a crucial aspect of any GraphQL API, and Apollo Server provides the tools to help you implement authentication, authorization, and other security best practices. By leveraging HTTPS, JWTs, OAuth, rate limiting, and depth limiting, you can protect your GraphQL API from unauthorized access, abuse, and performance degradation.
