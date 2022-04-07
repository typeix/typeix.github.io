---
template: main.html
title: GraphQL - Resolvers
---
# Resolvers
Resolvers are like rest controllers but they provide mapping to graphql operations: query, mutation into data.
Big difference is that in REST you invoke for example 5 rest resource to do create,read,update,delete operations where in graphql
we can execute 5 different operations on same graphql request resource.

## Queries  
Queries are used to fetch data and are meant to be read only operations, inside query operation we can implement custom filters.
We need to use `@Resolver` decorator so that type-graphql can recognize that current class is resolver type.


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
Resolvers need's to be referenced to schema builder and to graphql custom controller so that typeix dependency injecton can 
create UserResolver object and deliver it to graphql request, as it's provided example in quick-start we use create provider 
on service to create schema on application startup time.
````ts
@CreateProvider({
  provide: "GraphqlConfigSchema",
  useFactory: async () => {
    return await buildSchema({
      resolvers: [UserResolver],
      container: ({context}) => context.container
    });
  },
  providers: []
})
````
and we provide provider info to controller so resty create those objects on request:
````ts
@Controller({
  path: "/graphql",
  providers: [UserResolver],
  interceptors: []
})
````
## Arguments
Usually, queries have some arguments - it might be the id of a resource, a search phrase or pagination settings. 
TypeGraphQL allows you to define arguments in two ways.

Using `@Arg()` decorator we can inject value, we need to provide argument name and options if nullable or defaultValue for example.
````ts
@Injectable()
@Resolver(User)
export class UserResolver {

  @Inject() private userService: UserService;
  @Inject() private logger: Logger;

  @Query(()=> [User])
  users(
    @Arg("firstName", { nullable: true }) firstName: string
  ): Array<User> {
    this.logger.debug(`Injected argument: ${firstName}`);
    return this.userService.findByName(firstName);
  }
}
````
Or we can simpy define argument type and inject argument type itself:
```ts
@ArgsType()
export class GetUserArgs {
  @Field({ nullable: true }) firstName?: string;
  @Field({ nullable: true }) lastName?: string;
  @Field({ nullable: true }) age?: number;
}
```
Now we can simply reference argument type in query using `@Args()` decorator and with destructuring syntax 
we gain access to single arguments as variables.
````ts
@Injectable()
@Resolver(User)
export class UserResolver {

  @Inject() private userService: UserService;
  @Inject() private logger: Logger;

  @Query(()=> [User])
  users(@Args() {firstName, lastName, age}: GetUserArgs): Array<User> {
    this.logger.debug(`Injected argument: ${firstName}`);
    return this.userService.findByName(firstName);
  }
}
````

## Interceptors
With power of typeix dependency injection we can implement interceptors which are working with any method as long as object is created
with Injector. Interceptors can be used to implement any custom logic and it can change result or behavior of method, for example
Logging, Authorization.

In following example we are going to implement logger Interceptor:
````ts
import {createMethodInterceptor, Inject, Injectable, Interceptor, Logger, Method} from "@typeix/resty";

@Injectable()
class LoggerInterceptor implements Interceptor {
  @Inject() private logger: Logger;

  async invoke(method: Method): Promise<any> {
    let result = await method.invoke();
    this.logger.debug([
      `Injected arguments: ${this.prettyPrint(method.methodArgs)}`,
      `Result ${this.prettyPrint(result)}`,
      `With decorator args: ${this.prettyPrint(method.decoratorArgs)}`
    ].join("\n"));
    return result;
  }

  private prettyPrint(value: any): string {
    return JSON.stringify(value, null, " ");
  }
}

export function LogOutput(value) {
  return createMethodInterceptor(LogOutput, LoggerInterceptor, {value});
}
````
And now we can use interceptor in our resolvers by using custom defined `@LogOutput()` decorator.
````ts
import {Inject, Injectable} from "@typeix/resty";
import {Arg, Query, Resolver} from "type-graphql";
import {UserService} from "~/services/user.service";
import {User} from "~/data/user.entity";
import {LogOutput} from "~/controllers/graphql/logger.interceptor";

@Injectable()
@Resolver(User)
export class UserResolver {

  @Inject() private userService: UserService;

  @Query(()=> [User])
  @LogOutput("Logging UserResolver Output")
  users(@Args() {firstName, lastName, age}: GetUserArgs): Array<User> {
    return this.userService.findByName(firstName);
  }
}
````
Example Log that is printed out is:
````text
[1649358777779] DEBUG (): Injected arguments: [
 null
]
Result [
 {
  "permissions": [],
  "id": 1,
  "firstName": "John",
  "lastName": "Doe",
  "age": 30
 },
 {
  "permissions": [],
  "id": 2,
  "firstName": "Tifany",
  "lastName": "Doe",
  "age": 31
 }
]
With decorator args: {
 "value": "Logging UserResolver Output"
}
````
## Field Resolvers
Using `@FieldResolver()` provides you a way to extend current entity field with custom logic, for example we have database link,
custom computed calculation like age from date of birth in User entity or to deliver permissions as lazy link

In following example we can see how permissions are delivered as link to users, for each user it will invoke permissions 
resolver to deliver permissions data to user.
````ts
import {Inject, Injectable, Logger} from "@typeix/resty";
import {Arg, FieldResolver, Query, Resolver, Root} from "type-graphql";
import {UserService} from "~/services/user.service";
import {User} from "~/data/user.entity";
import {LogOutput} from "~/controllers/graphql/logger.interceptor";
import {PermissionService} from "~/services/permission.service";
import {Permission} from "~/data/permission.entity";

@Injectable()
@Resolver(User)
export class UserResolver {

  @Inject() private userService: UserService;
  @Inject() private permissionService: PermissionService;
  @Inject() private logger: Logger;

  @Query(()=> [User])
  @LogOutput("Logging UserResolver.users")
  users(
    @Arg("firstName", { nullable: true }) firstName: string
  ): Array<User> {
    return this.userService.findByName(firstName);
  }

  @FieldResolver()
  @LogOutput("Logging UserResolver.users.permissions")
  permissions(@Root() user: User): Array<Permission> {
    return this.permissionService.find(user);
  }
}
````
GraphQL Query that will deliver permissions nested to user and permissions by as separate grapql query
````graphql
query GetUsersAndPermissions {
  users {
    id,
    firstName,
    lastName,
    age,
    permissions {
        id,
        action
    }
  }
  permissions {
    id,
    action
  }
}
````
And in User entity we need to create permissions link:
````ts
import {Field, ID, ObjectType} from "type-graphql";
import {Permission} from "~/data/permission.entity";

@ObjectType()
export class User {

  @Field(() => ID) id: number;

  @Field() firstName: string;

  @Field() lastName: string;

  @Field() age: number;

  @Field(() => [Permission]) permissions: Array<Permission> = [];
}
````
