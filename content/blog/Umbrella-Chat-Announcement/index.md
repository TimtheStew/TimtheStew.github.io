---
title: New Project! Umbrella Chat - an Encrypted Chat Service
date: "2019-03-20T17:39:03"
---

For about the past month, my friend Atta and I have been working on
a new project. We've wanted to build something together for some time
but kept throwing around ideas without actually starting. So finally 
we just picked one that interested us both, and went for it.

Umbrella Chat is going to be an end-to-end encrypted chat service built
as a 12-factor web-app. [Check out the source here!](https://github.com/TimtheStew/umbrella-chat)

## The Stack

On the back-end, we're using Apollo Server, for ease of setting up
a GraphQL API, and Sequelize as an ORM tool for our Postgres DB.

In the front, we're using webpack to bundle our assets, babel to maintain
compatibility, React to display a UI, and Redux to manage our state.

That all sounds great! Modern and efficient! But what does it mean?

Well strap in, and let me tell you...

### Back-End

[GraphQL](https://graphql.org/) is an alternative to traditional RESTful
API's, developed at facebook. The basic idea behind it is that instead of
having lots of endpoints for very specific queries, you have one
"smart" endpoint, that delivers just the data the client needs, and in whatever
configuration they need it. This cuts down on data processing on the front end, and 
means you don't have to write a new query every time your front-end needs a
different collection of data.

A GraphQL server is a layer that sits between your front end and your data store(s).
It works with three main components: Schemas, Queries, and Resolvers.

GraphQL's typed **schemas** and [Schema Definition Language](https://www.apollographql.com/docs/apollo-server/essentials/schema#sdl)
are at the core of how it functions. A schema is how you describe your data and 
it's types to graphql. SDL offers several scalar types (bools, ints, strings, etc.)
and some abstractions, like objects, to allow you to define your data in terms
of types and relationships. For instance a rudimentary Chat definition might 
look like:
```
type Chat{
    id: ID
    name: String
    users: [String]
    createdAt: String
    updatedAt: String
}
```
There are other things you'll probably want to include in schemas, such as extensions
to the Query and Mutation types, but I won't get into those here. 

**Queries** are how you ask for the data you want, and are store-agnostic! It doesn't 
matter where what you need comes from, GraphQL takes care of that for you. They can
also take arguments, which plays into that "only the data you need" idea.
So lets say I wanted a chat with a particular id, my query might look like this:
```
query{
    chat(id: "someUUID"){
        name
        users
        updatedAt
        createdAt
    }
}
```
Since both queries and schemas don't care about where things come from, you figure 
something has to right? Thats where **resolvers** come in! A resolver does what 
it says on the tin - it resolves abstract queries into concrete data by following
the procedures you lay out for it. More simply, it tells GraphQL where and how to look
for what it wants.

GraphQL is really just a specification, and can be implemented in any language. For this
project the implementation we're using is
[Apollo Server](https://www.apollographql.com/docs/apollo-server/) which allows for 
easy set-up and auto-documenting of the API your GraphQL server uses. It also 
offers compatibility with a wide range of sources
(such as a Postgres DB) and is popular, well-documented and well-maintained. 

For our database, we're using [PostgreSQL](https://www.postgresql.org/), which
is an open source object-relational database system. It has an emphasis on
extensibility and standards compliance, as well as a wide support base, given
it is the DB of choice for many individuals and organizations. 

For ease of querying our database, we're using an ORM, or Object Relational Mapping 
(library) called [Sequelize](http://docs.sequelizejs.com/). Sequelize is a
promise based Javascript ORM, that supports a handful of popular DBs such 
as MySQL, Microsoft SQL Server, **Postgres**, and SQLite. It's purpose is to hide the 
mechanics of SQL querying behind an object model. So instead of 
writing a statement like 
```
SELECT * FROM chats WHERE name = 'My Squad' 
```
you would do:
```
Chat.findAll({
  where: {
    name : 'My Squad'
  }
});
```
Now, with such a trivial example, it might seem we've actually made things 
more complicated, but as queries grow in demand or generality, they can become quite tricky to write
optimally, and that is where Sequelize really shines. It means that I don't 
have to be a SQL master to use fast queries, and the object model can greatly
simplify other more ungainly selections. However, since Sequelize isn't familiar 
with the ins and outs of my data, I can't expect it be as good as an actual SQL master 
at writing queries, but I can expect it to be better than me.

This object model is how our GraphQL resolvers will return the data they need to.

Using an ORM allows us to employ some OOP techniques to better manage and define our data,
even at the database level.

Also, because it abstracts the DB, we could switch to any other DB supported by 
Sequelize down the line without too much hassle. 

### Front-End







