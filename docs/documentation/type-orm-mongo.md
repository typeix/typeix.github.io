---
template: main.html 
title: MongoDB with TypeORM integration
---

# Mongodb
MongoDB is a general purpose, document-based, distributed database built for modern application developers and for the cloud era.

TypeORM has basic MongoDB support, find out more by reading [official docs](https://typeorm.io/#/mongodb).

Resty framework fully supports integration with typeorm without any extra library or adapter. You can find full example
in [resty starters repository](https://github.com/typeix/resty-starters/tree/master/api-typeorm-mongo).

## Installing

Let's start by creating new typeix project, by running `@typeix/cli` commands:

```bash
typeix new api-typeorm-mongodb
cd api-typeorm-mongodb
```

Install typeorm and postgres connector by running:

```bash
npm i  --save typeorm mongodb
```

You need to install docker for your local development environment and start postgres in docker container

```shell
docker run -p 27017:27017 -d --name typeorm-mongo mongo
```

## Module

In following example let's encapsulate typeorm in datastore module by running:

```bash
typeix generate mdl DataStore
cd src/data-store
```

Goal is to generate following structure

```text
L src
  L orm-config.json
  L data-store
      L data-store.module.ts
      L config
         L pg.config.ts
         L pg.logger.config.ts
      L entity
         L user.entity.ts
      L services
         L entity.service.ts
         L user.service.ts
      L migration
         L {timestamp}-users.ts
```

## Connection

Connection options is a connection configuration you pass to createConnection or define in ormconfig file. Different
databases have their own specific [mongodb connection options](https://typeorm.io/#/connection-options/mongodb-connection-options).

Inside source let's create orm-config.json file with following configuration:

```json
{
  "type": "mongodb",
  "host": "localhost",
  "port": 27017,
  "username": null,
  "password": null,
  "database": "typeix",
  "useUnifiedTopology": true,
  "useNewUrlParser": true,
  "synchronize": false,
  "entities": [
    "dist/modules/data-store/entity/**/*.js"
  ]
}

```

Inside datastore module let's create connection config file:

```ts
import {CreateProvider, Injectable} from "@typeix/resty";
import {Connection, createConnection, ConnectionOptions, MongoEntityManager} from "typeorm";
import * as pgConfig from "~/orm-config.json";

@Injectable()
export class PgConfig {

  @CreateProvider({
    provide: Connection,
    useFactory: async () => {
      return await createConnection(<ConnectionOptions>{
        ...pgConfig,
        name: "default",
        logging: process.env.NODE_ENV !== "prod"
      });
    },
    providers: []
  }) private connection: Connection;
  
  getConnection(): Connection {
    return this.connection;
  }

  getEntityManager(): MongoEntityManager {
    return this.connection.mongoManager;
  }

  getMongoRepository<T>(entity: ObjectType<T>): T {
    return this.connection.getCustomRepository(entity);
  }
}
```

## Entity

Entity is a class that maps to a database table (or collection when using MongoDB).

Entity is your model decorated by an `@Entity` decorator, a database table will be created for such models. You can
load/insert/update/remove and perform other operations with them.

To add database columns, you simply need to decorate an entity's properties you want to make into a column with
a `@Column` decorator.

Let's create our first User Entity:
```ts
import {Column, Entity, ObjectIdColumn, ObjectID} from "typeorm";

@Entity()
export class User {

  @ObjectIdColumn()
  id: ObjectID;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column()
  age: number;

}
```

## MongoRepository
Repository is just like EntityManager but its operations are limited to a concrete entity.

In following example you can see implementation of UserRepository
```ts
import {Injectable} from "@typeix/resty";
import {User} from "~/data-store/entity/user.entity";
import {EntityRepository, MongoRepository} from "typeorm";

@Injectable()
@EntityRepository(User)
export class UserRepository extends MongoRepository<User> {

}
```

TypeORM and Resty needs to be aware about custom repositories, so we initialize them in modules:
```ts
import {Module} from "@typeix/resty";
import {PgConfig} from "~/data-store/configs/pg.config";
import {UserRepository} from "~/data-store/repository/user.repository";

function createRepositoryFactory<T>(Class: ObjectType<T>): IProvider {
  return  {
    provide: Class,
    useFactory: (config: MongoConfig) => config.getMongoRepository(Class),
    providers: [MongoConfig]
  };
}

@Module({
  providers: [
    MongoConfig,
    createRepositoryFactory(UserRepository)
  ],
  exports: [UserRepository]
})
export class MongoModule {

}
```



## Services
TBD...

NOTE:
> Name services should be done using verbs instead of nouns! <br />
> There is no “user domain” and there should also be no “UserService”.  <br />
> Instead, we can have “UserRegistrationService” or “UserAuthenticationService”. <br />

By naming our service by the primary business action or process we also make it clear what is the behavior we want it to implement.


