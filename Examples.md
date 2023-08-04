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

### Versioning 
REST API: Versioning in REST APIs is typically done by including version numbers in the URL, such as `/v1/endpoint`

GraphQL: GraphQL avoids the need for versioning because clients can explicitly specify the data they need, and the schema is self-descriptive. This means that changes to the schema can be made without breaking existing clients.

