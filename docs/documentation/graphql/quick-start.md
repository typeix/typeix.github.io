---
template: main.html 
title: GraphQL - Quick Start
---

# Quick Start

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
in [resty starters repository](https://github.com/typeix/resty-starters/tree/master/ts-graphql){:target="_blank"}.

## Resty with Graphql
Goal is to generate following structure:
```text
src
 L controllers
    L rest
        L test
            L user.controller.integration-spec.ts
            L permission.controller.integration-spec.ts
        L permission.controller.ts
        L user.controller.ts
    L graphql
        L test
            L permission.resolver.integration-spec.ts
            L user.resolver.integration-spec.ts
        L graphql.config.ts
        L graphql.controller.ts
        L user.resolver.ts
        L permission.resolver.ts
 L services
    L user.service.ts
    L permission.service.ts
 L app.module.ts
 L bootstrap.ts
```


## Schema config
In following example graphql.config.ts we need to use type-graphql to build schema, schema can be generated from code or from
schema .gql file using graphql syntax, in schema config we need to declare all resolvers.
````ts
import {CreateProvider, Injectable, RouterError} from "@typeix/resty";
import {buildSchema} from "type-graphql";
import {GraphQLSchema} from "graphql";
import {UserResolver} from "~/controllers/graphql/user.resolver";
import {PermissionResolver} from "~/controllers/graphql/permission.resolver";

@Injectable()
export class GraphQLConfig {

  @CreateProvider({
    provide: "GraphqlConfigSchema",
    useFactory: async () => {
      return await buildSchema({
        resolvers: [UserResolver, PermissionResolver],
        container: ({context}) => context.container
      });
    },
    providers: []
  })
  private schema: GraphQLSchema;

  public getSchema(): GraphQLSchema {
    return this.schema;
  }

  public parseBody(body: Buffer): any {
    try {
      return JSON.parse(body.toString());
    } catch (e) {
      throw new RouterError("GraphQL syntax error, cannot parse json request", 400);
    }
  }
}

````
## Controller
In order for GraphQL to process request it needs to have only one REST endpoint which accepts request body as json which contains
 query operation name and variables, query can contain graphql query and mutations. It's important to provide all resolvers to graphql
controller injection point, because that would create new instance of resolvers on each request.
````ts
import {addRequestInterceptor, BodyAsBufferInterceptor, Controller, Inject, Injector, POST, RouterError} from "@typeix/resty";
import {PermissionResolver} from "~/controllers/graphql/permission.resolver";
import {UserResolver} from "~/controllers/graphql/user.resolver";
import {GraphQLConfig} from "~/controllers/graphql/graphql.config";
import {graphql, parse, Source, specifiedRules, validate} from "graphql";

@Controller({
  path: "/graphql",
  providers: [PermissionResolver, UserResolver],
  interceptors: []
})
export class GraphQLController {

  @Inject() graphQlConfig: GraphQLConfig;
  @Inject() injector: Injector;

  @POST()
  @addRequestInterceptor(BodyAsBufferInterceptor)
  processGraphQL(@Inject() body: Buffer) {
    let schema = this.graphQlConfig.getSchema();
    let {query, operationName, variables} = this.graphQlConfig.parseBody(body);
    let source = new Source(query, operationName);
    let documentAST = parse(source);
    let validationErrors = validate(schema, documentAST, specifiedRules);
    if (validationErrors.length > 0) {
      throw new RouterError("GraphQL validation error.", 400, validationErrors);
    }
    return graphql(
      schema,
      source,
      null,
      {
        container: this.injector
      },
      variables,
      operationName
    );
  }
}
````

## Resolvers
Resolvers are like REST controllers, but it's based on graphql api and there you should define your queries and mutations.
You can specify type of resolver or you can live out type on resolver and bind it to query or mutation.
````ts
import {Inject, Injectable} from "@typeix/resty";
import {Query, Resolver} from "type-graphql";
import {UserService} from "~/services/user.service";
import {User} from "~/data/user.entity";

@Injectable()
@Resolver(User)
export class UserResolver {
    
  @Inject() private userService: UserService;

  @Query(()=> [User])
  users(): Array<User> {
    return this.userService.find();
  }
}
````
In following example we can see definition of user scalar type:
````ts
@ObjectType()
export class User {

  @Field(() => ID) id: number;

  @Field() firstName: string;

  @Field() lastName: string;

  @Field() age: number;

  static new(id: number, firstName: string, lastName: string, age: number): User {
    let obj = new User();
    obj.id = id;
    obj.firstName = firstName;
    obj.lastName = lastName;
    obj.age = age;
    return obj;
  }
}

````
User service data mock, normally we would call database to fetch data in service:
````ts
import {Injectable} from "@typeix/resty";
import {User} from "~/data/user.entity";

@Injectable()
export class UserService {
  find(): Array<User> {
    return [
      User.new(1, "John", "Doe", 30),
      User.new(2, "Tifany", "Doe", 31)
    ];
  }
}
````

## Testing
Integration testing with resty & graphql is very easy due to fake server implementation.
````ts
import {fakeHttpServer, FakeServerApi} from "@typeix/resty";
import {AppModule} from "~/app.module";


describe("UserResolver", () => {
  let fakeServer: FakeServerApi;

  beforeEach(async () => {
    fakeServer = await fakeHttpServer(AppModule);
  });
  describe("Integration", () => {
    test("GetUsers", async () => {
      let body = {
        "query": `
           query GetUsers {
            users {
              id,
              firstName,
              lastName,
              age
            }
          }
        `
      };
      let result = await fakeServer.POST("/graphql", {}, 
        Buffer.from(JSON.stringify(body))
      );
      expect(JSON.parse(result.getBody().toString())).toEqual({
        "data": {
          "users": [
            {
              "age": 30,
              "firstName": "John",
              "id": "1",
              "lastName": "Doe"
            },
            {
              "age": 31,
              "firstName": "Tifany",
              "id": "2",
              "lastName": "Doe"
            }
          ]
        }
      });
    });
  });
});
````

full working example can be found
in [resty starters repository](https://github.com/typeix/resty-starters/tree/master/ts-graphql){:target="_blank"}.

