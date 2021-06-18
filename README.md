# faunadb-graphql-api

FaunaDB is a serverless cloud database for modern applications. It is designed to focus on quick deployment, simplicity of daily operations, and developer’s ease of use. It offers a GraphQL API which can be joined with Hasura using Remote Joins.
In this example, we will look at how a schema in FaunaDB can be joined with existing data in Hasura. Assuming that there is a users table in Hasura with columns id and name, let's get started.

## Creating a GraphQL Schema in FaunaDB
Login to FaunaDB Cloud
- This is the dashboard to manage your database and security.
- Create a new database to store data. Let's name it hasura
- Create a GraphQL Schema. Let's create a simple schema to manage user's todos. Download the below schema

```graphql
type Todo {
   title: String!
   completed: Boolean!
   userId: Int!
}

type Query {
   allTodos: [Todo!]
   todosByUser(userId: Int): [Todo!]
}
```

Import your GraphQL Schema by clicking on GraphQL on the left sidebar navigation and choose the GraphQL schema file that you downloaded above. Once the import is successful, you should see the GraphQL Playground to start playing with the APIs.

## Insert sample data in FaunaDB
Let's insert sample data for the above schema. In the GraphQL Playground, execute this mutation to insert a todo.

```graphql
mutation CreateATodo {
   createTodo(data: {
   title: "Build an awesome app!"
   completed: false
   userId: 1
   }) {
       title
       completed
   }
}
```

This inserts a todo tagging to userId 1.

## Getting the Access Key

The GraphQL endpoint requires authentication with a specific FaunaDB database. We need to create a new key with role Server Read-Only.

![](https://hasura.io/blog/content/images/2019/08/create-faunadb-key.png)

The secret generated must be provided as an HTTP Basic Authorization header, encoded as a Base64 string. For example, if your FaunaDB secret is fnADMxRzydATDKibGAciQlNQWBs-HJdpJS1vJaIM, you can encode it like so:

```bash
echo -n "fnADMxRzydATDKibGAciQlNQWBs-HJdpJS1vJaIM:" | base64
Zm5BRE14Unp5ZEFUREtpYkdBY2lRbE5RV0JzLUhKZHBKUzF2SmFJTTo=
```

The trailing colon (:) is required.

Then your Authorization header would look like this:

```bash
Authorization: "Basic Zm5BRE14Unp5ZEFUREtpYkdBY2lRbE5RV0JzLUhKZHBKUzF2SmFJTTo="
```

## Adding FaunaDB as Remote Schema

To be able to query FaunaDB data via Hasura, it needs to be added as a Remote Schema using the Hasura Console.

The GraphQL API Endpoint is: https://graphql.fauna.com/graphql

Now copy the secret key (base64 encoded) and use it in Authorization headers like below:

```
Authorization: Basic <base64 encoded key>
```

In Hasura Console, head to Remote Schemas and enter GraphQL Server URL with the above endpoint. Under Additional Headers, enter the Authorization header with the secret as mentioned above.

![](https://hasura.io/blog/content/images/2019/08/add-remote-schema-faunadb.png)

Now, let's go to users table -> Relationships and add the remote relationship todos.

![](https://hasura.io/blog/content/images/2019/08/remote-relationship-faunadb.png)

Under configuration for todosByUser, we declare a filter to say:

```
userId: From column -> id
```

This ensures that while querying for todos in users, we get only the relevant todos written by the user who is queried for.

Now the GraphQL query to fetch this data in a single API call would look like the following:

```graphql
query {
  users {
    id
    name
    todos {
      data {
        title
        completed
      }
    }
  }
}
```

Notice that, the nested query todos come from FaunaDB and it will apply the filter of users.id = todos.data.userId, there by only giving todos written by the user.
