---
template: main.html title: TypeORM integration
---

# TypeORM

TypeORM is an ORM that can run in NodeJS, and can be used with TypeScript.

TypeORM supports both Active Record and Data Mapper patterns, unlike all other JavaScript ORMs currently in existence,
which means you can write high quality, loosely coupled, scalable, maintainable applications the most productive way.

TypeORM provides support for many relational databases: PostgreSQL, Oracle, Microsoft SQL Server, SQLite, and even NoSQL
databases like MongoDB.

Resty framework fully supports integration with typeorm without any extra library or adapter. You can find full example
in [resty starters repository](https://github.com/typeix/resty-starters/tree/master/api-typeorm-sql).

## Installing

Let's start by creating new typeix project, by running `@typeix/cli` commands:

```bash
typeix new api-typeorm-sql
cd api-typeorm-sql
```

Install typeorm and postgres connector by running:

```bash
npm i  --save typeorm pg
```

You need to install docker for your local development environment and start postgres in docker container

```shell
docker run -p 5432:5432 --name typeorm-pg -e POSTGRES_DB=typeix -e POSTGRES_PASSWORD=admin -e POSTGRES_USER=postgres -d postgres
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
databases have their own specific [connection options](https://typeorm.io/#/connection-options).

Inside source let's create orm-config.json file with following configuration:

```json
{
  "type": "postgres",
  "host": "localhost",
  "port": 5432,
  "username": "postgres",
  "password": "admin",
  "database": "typeix",
  "synchronize": false,
  "migrationsRun": true,
  "logging": true,
  "cli": {
    "entitiesDir": "src/data-store/entity",
    "migrationsDir": "src/data-store/migration",
    "subscribersDir": "src/data-store/subscriber"
  },
  "entities": [
    "dist/data-store/entity/**/*.js"
  ],
  "migrations": [
    "dist/data-store/migration/**/*.js"
  ],
  "subscribers": [
    "dist/data-store/subscriber/**/*.js"
  ]
}
```

Inside datastore module let's create connection config file:

```ts
import {CreateProvider, Injectable} from "@typeix/resty";
import {Connection, createConnection, Logger} from "typeorm";
import {Repository} from "typeorm/repository/Repository";
import {EntityTarget} from "typeorm/common/EntityTarget";
import * as pgConfig from "~/orm-config.json";
import {ConnectionOptions} from "typeorm/connection/ConnectionOptions";
import {PgLoggerConfig} from "~/data-store/configs/pg.logger.config";

@Injectable()
export class PgConfig {

  @CreateProvider({
    provide: Connection,
    useFactory: async (logger: Logger) => {
      return await createConnection(<ConnectionOptions>{
        ...pgConfig,
        logging: process.env.NODE_ENV !== "prod",
        logger
      });
    },
    providers: [PgLoggerConfig]
  }) private connection: Connection;

  getConnection(): Connection {
    return this.connection;
  }

  getRepository<T>(entity: EntityTarget<T>): Repository<T> {
    return this.connection.getRepository(entity);
  }
}
```

## Logger

You can enable logging of all queries and errors by simply setting logging: true in your connection options. You can
enable different types of logging in connection options:

* query - logs all queries.
* error - logs all failed queries and errors.
* schema - logs the schema build process.
* warn - logs internal orm warnings.
* info - logs internal orm informative messages.
* log - logs internal orm log messages.

If you have performance issues, you can log queries that take too much time to execute by setting maxQueryExecutionTime
in connection options.

By Creating custom TypeORM logger we can integrate it with Typeix Logger:

```ts
import {Logger as TypeOrmLogger, QueryRunner} from "typeorm";
import {Inject, Injectable, Logger} from "@typeix/resty";
import * as chalk from "chalk";

const highlight = require('cli-highlight').highlight

@Injectable()
export class PgLoggerConfig implements TypeOrmLogger {
  @Inject() logger: Logger;

  protected getRunnerInfo(queryRunner?: QueryRunner) {
    return queryRunner && queryRunner.data["request"] ? "(" + queryRunner.data["request"].url + ") " : "";
  }

  log(level: "log" | "info" | "warn", message: any, queryRunner?: QueryRunner): any {
    const info = this.getRunnerInfo(queryRunner);
    if (level === "warn") {
      this.logger.warn(info + message);
    } else {
      this.logger.info(info + message);
    }
  }

  logMigration(message: string, queryRunner?: QueryRunner): any {
    const info = this.getRunnerInfo(queryRunner);
    this.logger.info(info + "Migration: " + chalk.white(highlight(message, {language: 'sql', ignoreIllegals: true})));
  }

  logQuery(query: string, parameters?: any[], queryRunner?: QueryRunner): any {
    const info = this.getRunnerInfo(queryRunner);
    this.logger.info(info + "Query: " + chalk.white(highlight(query, {language: 'sql', ignoreIllegals: true})));
  }

  logQueryError(error: string | Error, query: string, parameters?: any[], queryRunner?: QueryRunner): any {
    const info = this.getRunnerInfo(queryRunner);
    this.logger.error(info + "Query: " + chalk.white(highlight(query, {language: 'sql', ignoreIllegals: true})));
    this.logger.error(error);
  }

  logQuerySlow(time: number, query: string, parameters?: any[], queryRunner?: QueryRunner): any {
    const info = this.getRunnerInfo(queryRunner);
    this.logger.warn(info + "Slow Query: " + chalk.white(highlight(query, {language: 'sql', ignoreIllegals: true})));
  }

  logSchemaBuild(message: string, queryRunner?: QueryRunner): any {
    const info = this.getRunnerInfo(queryRunner);
    this.logger.warn(info + "Schema Build: " + chalk.white(highlight(message, {
      language: 'sql',
      ignoreIllegals: true
    })));
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
import {Column, Entity, PrimaryGeneratedColumn} from "typeorm";

@Entity()
export class User {

  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column()
  age: number;

}
```

## Services
Entity services are finely-grained services that look after a single data entity, normally exposing nothing more than simple CRUD methods.

In following example we will create generic entity service with some generic repository methods:
```ts
import {Inject, Injectable} from "@typeix/resty";
import {PgConfig} from "~/data-store/configs/pg.config";
import {Repository} from "typeorm/repository/Repository";
import {DeleteResult} from "typeorm/query-builder/result/DeleteResult";
import {FindConditions} from "typeorm/find-options/FindConditions";
import {FindOneOptions} from "typeorm/find-options/FindOneOptions";
import {ObjectID} from "typeorm/driver/mongodb/typings";

@Injectable()
export abstract class EntityService<T> {

  @Inject() protected connection: PgConfig;


  findAll(): Promise<Array<T>> {
    return this.getRepository().find();
  }

  findOne(id?: string | number | Date | ObjectID, options?: FindOneOptions<T>)
  findOne(options?: FindOneOptions<T>)
  findOne(conditions?: FindConditions<T>, options?: FindOneOptions<T>)
  findOne(criteria: any, options?: any): Promise<T> {
    return this.getRepository().findOne(criteria, options);
  }

  save(entity: T): Promise<T> {
    return this.getRepository().save(entity);
  }

  create(entity: T): T {
    return this.getRepository().create(entity);
  }

  remove(entity: T): Promise<T> {
    return this.getRepository().remove(entity);
  }

  update(criteria: number | object | Array<number>, entity: T): Promise<DeleteResult> {
    return this.getRepository().update(criteria, entity);
  }

  count(options?: FindOneOptions<T>)
  count(conditions?: FindConditions<T>)
  count(criteria: any): Promise<number> {
    return this.getRepository().count(criteria);
  }

  delete(criteria: number | object | Array<number>): Promise<DeleteResult> {
    return this.getRepository().delete(criteria);
  }

  abstract getRepository(): Repository<T>;
}
```

Finally let's implement UserService:
```ts
import {Injectable} from "@typeix/resty";
import {User} from "~/data-store/entity/user.entity";
import {Repository} from "typeorm/repository/Repository";
import {EntityService} from "~/data-store/services/enttiy.service";

@Injectable()
export class UserService extends EntityService<User> {
  getRepository(): Repository<User> {
    return this.connection.getRepository(User);
  }
}
```

NOTE:
> Name services should be done using verbs instead of nouns! <br />
> There is no “user domain” and there should also be no “UserService”.  <br />
> Instead, we can have “UserRegistrationService” or “UserAuthenticationService”. <br />

By naming our service by the primary business action or process we also make it clear what is the behavior we want it to implement.

Implementation in controller:
```ts
import {Controller, Inject, GET} from "@typeix/resty";
import {AppService} from "./app.service";
import {UserService} from "~/data-store/services/user.service";

@Controller({
  path: "/",
  providers: [],
  interceptors: []
})
export class AppController {

  @Inject() userService: UserService;

  @GET("users")
  findAllUsers() {
    return this.userService.findAll();
  }
}

```
## Transactions

## Subscribers

## Migrations
