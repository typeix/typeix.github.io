---
template: main.html
title: GraphQL - types and fields
---
With Typescript we can easily map entities with GraphQL schema definitions.

##  Fields
Let's start by defining User entity:
````ts
class User {
    id: number;
    firstName: string;
    lastName: string;
    age: number;
}
````
In order for type-graphql to recognize types we have to use following decorators `@ObjectType` to make entity visible to 
type-graphql and `@Field` decorator to declare graphql properties.  

````ts

@ObjectType()
export class User {
    
    @Field(() => ID) 
    id: number;
    
    @Field() 
    firstName: string;
    
    @Field() 
    lastName: string;
    
    @Field(() => Int) 
    age: number;
}
````
Which will create following GraphQL schema in SDL:
````graphql
type User {
  id: ID!
  firstName: String!
  lastName: String!
  age: Int!
}
````
## Types
GraphQL supports scalar (ID, Int, Float, String, Boolean), objects and enums types.

For Arrays we  use the explicit [] array literal notation  `@Field(() => [Type])`.

For nested arrays, we just use the explicit [ ] notation to determine the depth of the array, e.g. 
`@Field(() => [[Type]])` would tell the compiler we expect an integer array of depth 2.

For Object we just reference object type `@Field(() => Type)`.

Nullables you can define in two ways, one is to override default behavior in schema build options `nullableByDefault: true`
but we recommend explicitly defining optional property.

````ts
@ObjectType()
export class User {

  @Field(() => ID)
  id: number;
  
  @Field()
  firstName: string;
  
  @Field()
  lastName: string;
  
  @Field(() => Int, { nullable: true })
  age?: number;
}
````
As you can see in example above we are using optional age property and we are explicitly defining options for that field.
Basic possible options are defined by FieldOptions interface.
````ts
interface FieldOptions {
  nullable?: boolean | "items" | "itemsAndList";
  defaultValue?: any;
  description?: string;
  name?: string; // Schema name
  complexity?: number;
}
````

Be aware that defining constructors is strictly forbidden on type graphql entites and we shouldn't use them there, as TypeGraphQL 
creates instances of object type classes under the hood by itself.
