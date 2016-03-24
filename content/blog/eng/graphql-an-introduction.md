---
authors: ["vito"]
comments: true
date: "2016-01-18"
draft: false
image: ""
menu: ""
share: true
categories: [GraphQL, REST, HTTP, Node.js]
title: "GraphQL: an introduction"
type: "post"

---


The lesson we learned with REST
---------------------------------------

For many developers, nowadays, building an **API** for their applications essentially means mapping the _resources_ of the domain to _URIs_, with the [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) principles in mind.
Usually creating a ***RESTful*** system is not difficult, and the simplicity of the idea makes the task easier.
Let's see some of the consequences and benefits of choosing REST:

* Every HTTP verb has its own meaning, allowing the developer to understand immediately what kind of operation (typically among the CRUD ones) is going to be performed on the resource identified by the URI.
* Often the same URI can be used with multiple verbs to accomplish different tasks (e.g. "example.com/tag/123" refers to a particular tag that you can retrieve, update or delete).
* Controllers in many server-side frameworks can be created with a RESTful approach, each one of them representing one or more resources.

However, unfortunately, it's often easy to come across some problems:

* Complex URIs can be difficult to write or understand and sometimes they are subject to arbitrary interpretations.
* In order to filter collections you may need to use one or more ids embedded in the URL, but also _query parameters_, e.g. when you want to sort the results of a GET by one particular field or retrieve resources by some field other than the id.
* There is no standard or easy way for the client to specify the list of fields that it needs, and the server will usually return all the data related to the requested resource.
* More importantly, there is no standard way for the client to decide which resources related to the one requested should be returned, resulting in a lot of noise in the response content (unless the server application offers multiple endpoints to access the same resource but with different subsets of data... but this could lead to API design problems).

You can't really blame REST for these and other cons. Its power comes from simplicity and developers are thankful for this breath of fresh air. You have just one transport layer, i.e. HTTP, with its features and rules.

Now let's see how we can take a step forward and add some salt to our APIs, to make them even more powerful and expressive.

A new way to design API: GraphQL
-----------------------------------------

**GraphQL** maintains all the advantages of REST (relies on HTTP, very small overhead on requests, stateless and cacheable) and adds some very useful enhancements.
One of the main reasons why **Facebook** created it back in 2012 was the need to find a way to access data on their server that allowed any type of client and device (especially mobile site and app) to decide exactly what fields they need, reducing the payload and then minimizing the traffic.

Instead of explaining all the features, which can be read in the [official guide](http://graphql.org/docs/getting-started/), in this post we'll focus on what makes GraphQL interesting.
In the next section we'll use types and queries presented in our example project, that you can clone from [this repository](https://github.com/poetcyborg/graphql-node-starter-kit).

### Types and fields driven queries

**Example #1**

```
query {
	games {
        id
	    title
	}
}

```
The previous snippet describes a query issued by the client when it just needs to retrieve _id_ and _title_ of a ***game***.
Since no argument has been passed to filter the data of the specified type (_game_), all the games will be retrieved.
The output for this query is:

```
{
  "data": {
    "games": [{
        "id": 1,
        "title": "Frogger"
      }, {
        "id": 2,
        "title": "Galaxian"
      }, {
        "id": 3,
        "title": "Tiger Road"
      }, {
        "id": 4,
        "title": "Mendel Palace"
      }
    ]
  }
}
```

With REST, the same task could be accomplished with the following url:

```
GET /game?fields=id,title
```

Unless the server applications is taking care of field filtering based on query params, all the information about every game will be returned.

**Example #2**

```
query {
	games(id:2) {
        id
        title
        year
	}
}
```

**Example #3**

```
query {
	games(title:"Frogger") {
		id
        title
        year
	}
}
```

Examples #2 and #3 show a way to filter games by their _id_ and _title_ respectively.
As you can see, the id has no semantic meaning like in REST, so you can describe the filter criterion in the same manner (_fieldname: "value"_).

The REST equivalent (selected fields are omitted) for Example #2:

```
GET /game/2
```

and #3:

```
GET /game?title=Frogger
```

### Relations between entities: graph representation

**Example #4**

```
query {
    games(id:2) {
        title
        developer {
            name
            nation
        }
    }
}
```

The **types** called _Game_ and _Company_ are related to each other; in our example, for the sake of simplicity, a game can only have one company as developer and one as publisher, while any company can have multiple developed or published games.
In Example #4, you can notice the complex field _developer_; we want to know the name and the nation of the company that developed the game with id = 2.

The same two types can be reused to express the opposite of the relation described in the previous example:

**Example #5:**

```
query {
    companies(name:"Game Freak") {
        name
        gamesAsDeveloper {
            title
        }
    }
}
```

Let's see briefly how you can describe these relations in the **Javascript** implementation.

In the '_Game_' type definition:

```
developer: {
	type: Company,
    description: 'The developer of the game',
}
```

And in the '_Company_' type:

```
gamesAsDeveloper: {
    type: new GraphQLList(Game),
    description: 'The games created as a developer',
}
```

The field _developer_ is defined as type _Company_, while the field _gamesAsDeveloper_ is a list of _Game_.
Please refer to the [demo project](https://github.com/poetcyborg/graphql-node-starter-kit) for the full implementation of this example.

### Minimizing the number of requests to the server

This time we need the list of the _Japanese companies_ and some information about the games they worked on, but only for the _NES_ (Nintendo Entertainment System) platform.

**Example #6:**

```
query {
    companies(nation:"Japan") {
	    name
        gamesAsDeveloper(platform:"NES") {
		    title
	        year
        }
        gamesAsPublisher(platform:"NES") {
            title
            year
        }
    }
}
```

This query's output:

```
{
  "data": {
    "companies": [{
        "name": "Konami",
        "gamesAsDeveloper": [],
        "gamesAsPublisher": []
      }, {
        "name": "Namco",
        "gamesAsDeveloper": [],
        "gamesAsPublisher": [{
            "title": "Mendel Palace",
            "year": "1989"
          }]
      }, {
        "name": "Game Freak",
        "gamesAsDeveloper": [{
            "title": "Mendel Palace",
            "year": "1989"
          }],
        "gamesAsPublisher": []
      }, {
        "name": "Capcom",
        "gamesAsDeveloper": [{
            "title": "Tiger Road",
            "year": "1987"
          }],
        "gamesAsPublisher": []
      }
    ]
  }
}
```

In the REST world, the same response could be obtained in several ways:

* by sending a request to an endpoint that was built exactly for the purpose of filtering companies and games by some fields, e.g.:

```
GET /companyWithGames?companyNation=Japan&gamePlatform=NES&fields=...
```

* by sending several requests to more generic endpoints, e.g.:

```
GET /company?nation=Japan&fields=...
GET /game?developerName=[developer 1 name]&gamePlatform=NES&fields=...
GET /game?developerName=[developer 2 name]&gamePlatform=NES&fields=...
GET ...
```

There's a couple of notable things here:

* the expressivity of the query language becomes more evident when things start becoming more interesting; filtering subsets of data feels natural and easy as you can just specify the criteria for the filters directly on the subtree root (companies -> nation, gamesAsDeveloper -> platform, gamesAsPublisher -> platform);
* on server side, the developer can write the code to retrieve the data for every subset of data independently, considering that, when a request is sent, every subtree receives the data returned by the upper levels (in this case, filtering _games_ with the given _platform_ is performed only on companies with the selected _nation_).

### API Documentation

When you design a REST WebService, for every endpoint you're mostly going to take care of its URI and the structure of the request and the response body.
Unfortunately, there is no standard way to document this information and make it available to the client developers.

With GraphQL, using ***introspection*** you can ask the server information about the available types and their fields.

**Example #7**:

```
query {
    __schema {
        types {
            kind
            name
            description
        }
    }
}
```

A query with the query root ***__schema*** returns the list of types with their descriptions and fields:

```
{
  "data": {
    "__schema": {
      "types": [
		...
        {
          "kind": "OBJECT",
          "name": "Company",
          "description": "A company object",
          "fields": [...]
        }, {
          "kind": "OBJECT",
          "name": "Game",
          "description": "A game object"
          "fields": [...]
        },
        ...
}
```

Another way to get the information about the types is using the query root ***__type***, passing a specific type name:

**Example #8**

```
query {
    __type(name:"Game") {
        fields {
            name
            description
            type {
                name
            }
        }
    }
}
```

Output:

```
{
    "data": {
        "__type": {
            "fields": [{
                "name": "id",
                "description": "The id of a game",
                "type": {
                    "name": "Int"
                }
            }, {
	            "name": "title",
                "description": "The title of a game",
                "type": {
                    "name": "String"
                }
            },
            ...
}
```

Mutations
------------

We've already talked about queries, but a few words need to be spent about the other important operation that you can perform on a GraphQL server: **mutations**.

**Example #9**

```
mutation {
	updateGameRating(id:2, rating:7) {
		id
		title
		rating
	}
}
```

The syntax is no different from the previous ones, except for the ***mutation*** root.
It is the equivalent of a POST or a PATCH in REST, and lets us change the value of some fields in a resource.
In this example we are assigning a value (7) to the _rating_ field of the _Game_ types; this mutation could be defined in the following way:

```
{
	name: 'UpdateGameRating',
	type: Game,
	args: {
	    rating: { type: new GraphQLNonNull(GraphQLInt) },
    }
}
```

As you can see, the specified type for the mutation is _Game_; this means not only that the update should be performed on games, but also that the _Game_ field included in the request (id, title and rating) should be returned, just like a normal query.


Implementations
---------------------

In the present article we sometimes referred to the official **Javascript** library for GraphQL, but as you can see in [this project](https://github.com/chentsulin/awesome-graphql) there are many different implementations for the most popular languages.


The future of GraphQL
--------------------------

It's difficult to say what will become of GraphQL in the next year.
Sure there's a lot of hype around it, and for some very solid reasons.
It's not going to replace REST in developers' hearts anytime soon, but if they're brave enough, they can add a **GraphQL layer** over an existing REST API in their application.
In fact, in my opinion, another good point of interest regarding GraphQL is the ability to aggregate multiple endpoints that reference various resources. Its nature is very domain-centric and it's more  focused on ***what*** data needs to be precisely retrieved, rather than on ***how to*** get it.

Some questions arise when you think about GraphQL in a more vast spectrum:

* how to **cache** resources on server-side, even if they were originally fetched using different queries? (Facebook is trying to give some answers with [Relay](https://facebook.github.io/relay/docs/thinking-in-relay.html));
* how to **limit the access** to the resources (or the introspection on them) only to some users? GraphQL is authorization-agnostic, but as you can read in [this Github issue](https://github.com/graphql/graphql-js/issues/113) its a common problem that needs to be at least faced.

The entire Facebook ecosystem of open source projects is worth of attention (don't forget about React, Flux and the other Flux-like architectures) and, even if it's very young, it has become very popular and used by many little and big companies around the world.

My personal hope is that every lesson that can be learned while working with these projects, especially GraphQL, might lead to a better World Wide Web and a better developing experience, maybe with more standard and free implementations than the Facebook (or Google, or Microsoft, or Apple) ones.
