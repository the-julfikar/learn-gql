# Apollo Server for GraphQL

**Apollo Server** is a popular, open-source JavaScript library that simplifies building a **GraphQL** API in a Node.js environment. It provides tools to define, implement, and run a GraphQL server that can handle queries, mutations, and subscriptions. Using Apollo Server, you can easily set up a GraphQL API to manage and retrieve data, whether from databases, REST APIs, or third-party services.

## Installation

```bash
npm init
npm pkg set type="module"
npm install @apollo/server graphql
```

## Architecture
**index.js**

```javascript
import { ApolloServer } from '@apollo/server';
import { startStandaloneServer } from '@apollo/server/standalone';

// server setup***
const server = new ApolloServer({
    // typeDefs
    // resolvers
});

const { url } = await startStandaloneServer(server, {
    listen: { port: 4000 }
});

console.log('Server ready at port - ', 4000);
```

**schema.js**. Here ***type Query*** is entry point in the **Graph** and this is not **optional**.

```javascript
export const typeDefs = `#graphql
    type Game {
        id: ID!
        title: String!
        platform: [String!]!
    }
    type Review {
        id: ID!
        rating: Int!
        content: String!
    }
    type Author {
        id: ID!
        name: String!
        verified: Boolean!
    }
    type Query {
        reviews: [Review]
        games: [Game]
        authors: [Author]
    }
`;

// int, float, boolean, string, ID
```

**index.js**.

```javascript
const resolvers = {
    Query: {
        games() {
            return db.games
        },
        authors() {
            return db.authors
        },
        reviews() {
            return db.reviews
        }
    }
};
```

## Query Variable
**schema.js**

```javascript
......
   type Query {
        reviews: [Review]
        review(id: ID!): Review
        games: [Game]
        authors: [Author]
    }
......
```

**index.js**

```javascript
const resolvers = {
    Query: {
        review(_, args) {
            return db.reviews.find((review) => review.id === args.id)
        }
    }
};

// here _ indicates parent
```
**Apollo client**
```javascript
query Review($reviewId: ID!) {
  review(id: $reviewId) {
    rating,
    content
  }
}
```
*Varibale:*
```javascript
{
  "reviewId": 100
}
```

## Relational Data
**schema.js**

```javascript
type Game {
        id: ID!
        title: String!
        platform: [String!]!
        reviews: [Review!]
    }
    type Review {
        id: ID!
        rating: Int!
        content: String!
        game: Game!
        author: Author!
    }
    type Author {
        id: ID!
        name: String!
        verified: Boolean!
        reviews: [Review!]
    }
```

## Parents Chaining
**index.js**
```javascript
query ReviewQ($reviewId: ID!) {
  review(id: $reviewId) {
    rating,
    content,
    game { //*parent
      title,
      platform,
      reviews { //*chaining
        rating
      }
    },
    author {
      name,
      verified
    }
  }
}
```

## Mutation
**schema.js**
```javascript
  type Mutation {
        addGame(game: AddGameInput!): Game
        deleteGame(id: ID!): [Game]
        updateGame(id: ID!, edits: EditGameInput!): Game
    }
    input AddGameInput {
        title: String!
        platform: [String!]!
    }
    input EditGameInput {
        title: String!
        platform: [String!]
    }
```

**index.js**
```javascript
    Mutation: {
        deleteGame(_, args) {
            db.games = db.games.filter((g) => g.id !== args.id)

            return db.games
        },
        addGame(_, args) {
            let game = {
                ...args.game,
                id: Math.floor(Math.random()*10000).toString()
            }
            db.games.push(game)
            return game
        },
        updateGame(_, args) {
            db.games = db.games.map((g) => {
                if(g.id === args.id) {
                    return {...g, ...args.edits}
                }
                return g
            })

            return db.games.find((g) => g.id === args.id)
        }
    }
```

**Apollo Server**
```javascript
mutation DelGame($deleteGameId: ID!) {
  deleteGame(id: $deleteGameId) {
    id,
    title,
    platform
  }
}
```
## Apollo Server Docs

[Tech-doc](https://www.apollographql.com/docs/apollo-server/getting-started#step-1-create-a-new-project)