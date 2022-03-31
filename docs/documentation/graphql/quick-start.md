---
template: main.html title: GraphQL Quick Start
---

# Getting started

[GraphQL](https://graphql.org/){:target="_blank"}  is a query language for APIs and a runtime for fulfilling those
queries with your existing data. GraphQL combined with [TypeScript](https://www.typescriptlang.org/){:target="_blank"}
helps you develop better type safety with your GraphQL queries, giving you end-to-end typing.

See [Comparison](https://dev-blog.apollodata.com/graphql-vs-rest-5d425123e34b){:target="_blank"} between GraphQL and
REST.

In this chapter, we assume a basic understanding of GraphQL, and focus on how to work with
the [Type-GraphQL](https://typegraphql.com/){:target="_blank"} module.

## Installation

Start by installing the required packages:

```bash
$ npm i -g @typeix/cli
$ typeix new grapql-project
$ cd grapql-project
$ npm i type-graphql graphql class-validator
$ typeix start --watch
```

[Type-GraphQL](https://typegraphql.com/){:target="_blank"} offers two ways of building GraphQL applications, the code
first and the schema first methods. You should choose the one that works best for you. Most of the chapters in this
GraphQL section are divided into two main parts:
one you should follow if you adopt code first, and the other to be used if you adopt schema first.

Resty fully supports integration with type-graphql without any extra library or adapter.
You can find full example
in [resty starters repository](https://github.com/typeix/resty-starters/tree/master/graphql-typeorm-sql){:target="_blank"}.

## Resty with Graphql
Goal is to generate following structure
```text
src
 L controllers
    L rest
        L permission.controller.ts
        L permission.controller.integration-spec.ts
        L user.controller.ts
        L user.controller.integration-spec.ts
    L graphql
        L user.resolver.ts
        L user.resolver.integration-spec.ts
        L permission.resolver.ts
        L permission.resolver.integration-spec.ts
    L graphql.config.ts
    L graphql.controller.ts
 L services
    L user.service.ts
    L permission.service.ts
 L app.module.ts
 L bootstrap.ts
```


TBD...
