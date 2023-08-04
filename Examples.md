# Examples

## GraphQL v REST API

### Data fetching and Response Structure 
### Single Endpoint versus Multiple Endpoints
 REST API: GET request to `/users/1` returns the entire user resource, including fields that might not be needed by the client.

GraphQL: GraphQL query `{ user(id: "1") { name, email } }` allows the client to request only the name and email fields for the user with ID 1.

```
User:
id
name
email
```

```
Post:
id
title
content
userId (foreign key linking to the User who created the post)
```

**REST API Example:**
REST API Endpoints:

Get a user by ID:
`GET /users/:id`

Get posts by a user's ID:
`GET /users/:id/posts`

Response Structure for Getting a User:
```
{
  "id": 1,
  "name": "John Doe",
  "email": "john@example.com"
}
```

Response Structure for Getting User's Posts:

```
[
  {
    "id": 1,
    "title": "First Post",
    "content": "This is my first blog post.",
    "userId": 1
  },
  {
    "id": 2,
    "title": "Second Post",
    "content": "This is my second blog post.",
    "userId": 1
  }
]
```

In the REST API approach, you need to make two separate requests to get the user information and their posts.

** GraphQL API Example:**
GraphQL Schema:

```
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
}
```
```
type Post {
  id: ID!
  title: String!
  content: String!
  userId: ID!
}
```

```
type Query {
  user(id: ID!): User
}
```
GraphQL Query for Getting a User and their Posts:

graphql
```
query GetUserAndPosts($userId: ID!) {
  user(id: $userId) {
    id
    name
    email
    posts {
      id
      title
      content
    }
  }
}
```
GraphQL Response:


```
{
  "data": {
    "user": {
      "id": "1",
      "name": "John Doe",
      "email": "john@example.com",
      "posts": [
        {
          "id": "1",
          "title": "First Post",
          "content": "This is my first blog post."
        },
        {
          "id": "2",
          "title": "Second Post",
          "content": "This is my second blog post."
        }
      ]
    }
  }
}
```

In the GraphQL API approach, you can fetch both user information and their posts in a single request, and the response includes exactly the data that the client requested.

As you can see, the GraphQL API allows the client to specify the specific fields it needs, avoiding over-fetching or under-fetching of data. It's important to note that the response structure in GraphQL directly matches the structure of the query, providing a more predictable and efficient data-fetching experience compared to REST API's fixed response structures.

REST APIs usually have multiple endpoints, with each endpoint corresponding to a specific resource or action. This can lead to a more complex API structure and sometimes requires multiple requests to fetch related data.

 GraphQL APIs have a single endpoint, and all data requests are made to this endpoint using a single POST request. Clients can request all the required data in a single query, and the server resolves the request based on the query's structure.

### Schema and Documentation 

REST API: Documentation for REST APIs is usually provided externally, and the API endpoints need to be well-documented for clients to understand how to interact with them.

GraphQL: GraphQL APIs are self-documenting due to their strongly-typed schema. The GraphQL schema serves as a contract between the client and server, providing clear information on available types, fields, and operations. Tools like GraphiQL and GraphQL Playground offer interactive documentation.

Let's look at `https://rickandmortyapi.com/graphql` together. 

If you want to explore with `graphiql` later 
`brew install --cask graphiql`

example query 
```
query character($id: ID!){
    character(id: $id){
        id
        name
        status
        species
        type
        gender
        origin{
            id
            name
            type
            dimension
            created
        }
        location{
            id
            name
            type
            dimension
            created
        }
        image
        episode{
            id
            name
            air_date
            episode
            created
        }
        created
    }
}
```
variables
`{"id":ID!}`

### Versioning 
REST API: Versioning in REST APIs is typically done by including version numbers in the URL, such as `/v1/endpoint`

GraphQL: GraphQL avoids the need for versioning because clients can explicitly specify the data they need, and the schema is self-descriptive. This means that changes to the schema can be made without breaking existing clients.

## Apollo Client 

**Caching Example**
Apollo Client comes with built-in caching that can help avoid redundant network requests and improve application performance. Here's how to use caching with Apollo Client:

```
import { gql, useQuery } from '@apollo/client';

const GET_USERS = gql`
  query GetUsers {
    users {
      id
      name
      email
    }
  }
`;

function UsersList() {
  const { loading, error, data } = useQuery(GET_USERS);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <ul>
      {data.users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

```

In this example, `useQuery` automatically caches the result of the `GET_USERS` query. If you navigate to another component that also fetches the same data, Apollo Client will use the cached result instead of making another network request.

### State Management 
Apollo Client can also be used for state management within your React application. You can use local state, which is stored in the Apollo cache but not sent to the server, to manage UI-related states.

```
import { gql, useQuery, useMutation } from '@apollo/client';

const GET_USER_NAME = gql`
  query GetUserName {
    userName @client
  }
`;

const SET_USER_NAME = gql`
  mutation SetUserName($name: String!) {
    setUserName(name: $name) @client
  }
`;

function UserNameEditor() {
  const { data } = useQuery(GET_USER_NAME);
  const [setUserName] = useMutation(SET_USER_NAME);

  return (
    <div>
      <input
        type="text"
        value={data.userName}
        onChange={(e) => setUserName({ variables: { name: e.target.value } })}
      />
    </div>
  );
}
```

In this example, we define local state using `@client` directive in the GraphQL schema. The `GET_USER_NAME` query fetches the local `userName` state, and the `SET_USER_NAME` mutation updates the local state.

### Error Handling Example:
Apollo Client provides error handling capabilities to handle errors returned from the GraphQL server.

```
import { gql, useQuery } from '@apollo/client';

const GET_USER = gql`
  query GetUser($id: ID!) {
    user(id: $id) {
      id
      name
      email
    }
  }
`;

function UserProfile({ userId }) {
  const { loading, error, data } = useQuery(GET_USER, {
    variables: { id: userId },
    onError: (error) => {
      console.error('GraphQL Error:', error.message);
      // You can handle errors here, e.g., show an error message to the user
    },
  });

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <div>
      <p>Name: {data.user.name}</p>
      <p>Email: {data.user.email}</p>
    </div>
  );
}
```
In this example, we use the `onError` option to handle GraphQL errors. If an error occurs during the execution of the query, the `onError` function will be called with the error object, allowing you to log the error or display an error message to the user.