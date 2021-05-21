---
template: main.html 
title: SQL with TypeORM integration
---

# TypeORM

Object–relational mapping (ORM, O/RM, and O/R mapping tool) in computer science 
is a programming technique for converting data between incompatible type systems 
using object-oriented programming languages.

TypeORM is an ORM that can run in NodeJS, and can be used with TypeScript. TypeORM provides support for 
many relational databases: PostgreSQL, Oracle, Microsoft SQL Server, SQLite, and even NoSQL
databases like MongoDB.

TypeORM supports both Active Record and Data Mapper patterns, unlike all other JavaScript ORMs currently in existence,
which means you can write high quality, loosely coupled, scalable, maintainable applications the most productive way.

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
      L pg.module.ts
      L config
         L pg.config.ts
         L pg.logger.config.ts
      L entity
         L user.entity.ts
      L repository
         L user.repository.ts
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
import {Connection, createConnection, Logger, ObjectType, EntityManager, ConnectionOptions} from "typeorm";
import {PgLoggerConfig} from "~/data-store/configs/pg.logger.config";
import * as pgConfig from "~/orm-config.json";

@Injectable()
export class PgConfig {

  @CreateProvider({
    provide: Connection,
    useFactory: async (logger: Logger) => {
      return await createConnection(<ConnectionOptions>{
        ...pgConfig,
        name: "default",
        logging: process.env.NODE_ENV !== "prod",
        logger
      });
    },
    providers: [PgLoggerConfig]
  }) private connection: Connection;


  getEntityManager(): EntityManager {
    return this.connection.manager;
  }

  getCustomRepository<T>(entity: ObjectType<T>): T {
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

## Repository
Repository is just like EntityManager but its operations are limited to a concrete entity.

In following example you can see implementation of UserRepository
```ts
import {Injectable} from "@typeix/resty";
import {User} from "~/data-store/entity/user.entity";
import {EntityRepository, Repository} from "typeorm";

@Injectable()
@EntityRepository(User)
export class UserRepository extends Repository<User> {

}
```

TypeORM and Resty needs to be aware about custom repositories, so we initialize them in modules:
```ts
import {Module} from "@typeix/resty";
import {PgConfig} from "~/data-store/configs/pg.config";
import {UserRepository} from "~/data-store/repository/user.repository";

@Module({
  providers: [
    PgConfig,
    {
      provide: UserRepository,
      useFactory: config => config.getCustomRepository(UserRepository),
      providers: [PgConfig]
    }
  ],
  exports: [UserRepository, PgConfig]
})
export class PgModule {

}
```

## Transactions
A database transaction symbolizes a unit of work performed within a database management system against a database,
and treated in a coherent and reliable way independent of other transactions.

In following example you can see implementation of transactional request interceptor, and we can use it in controller:
```ts
import {
  addRequestInterceptor,
  Inject,
  Injectable,
  Injector,
  InterceptedRequest,
  RequestInterceptor
} from "@typeix/resty";
import {PgConfig} from "~/modules/datastore/configs/pg.config";
import {Repository} from "typeorm";


@Injectable()
class TransactionalInterceptor implements RequestInterceptor {

  @Inject() injector: Injector;
  @Inject() config: PgConfig;

  async invoke(request: InterceptedRequest): Promise<any> {
    const queryRunner = this.config.getConnection().createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction("READ COMMITTED");
    try {
      const type = request.args.type;
      const repository: Repository<typeof type> = await queryRunner.manager.getCustomRepository(type);
      this.injector.set(type, repository);
      await request.handler();
      await queryRunner.commitTransaction();
    } catch (err) {
      await queryRunner.rollbackTransaction();
    } finally {
      await queryRunner.release();
    }
  }

}

/**
 * Transactional
 * @param type
 * @constructor
 */
export function Transactional<T>(type: T) {
  return addRequestInterceptor(TransactionalInterceptor, {type});
}

```
As we can see below in controller we inject transactional UserRepository which is injected by Transactional Interceptor
```ts
@Controller({
  path: "/",
  providers: [],
  interceptors: []
})
export class AppController {

  @Inject() appService: AppService;
  @Inject() userRepository: UserRepository;

  @POST("users")
  @Transactional(UserRepository)
  createUser(@Inject() repository: UserRepository) { // repository is injected by transaction interceptor!
    const user = new User();
    user.age = 100;
    user.firstName = "Igor";
    user.lastName = "Surname";
    return repository.save(user);
  }
}
```
## Subscribers
With TypeORM subscribers, you can listen to specific entity events.

## Migrations
Migrations provide a way to incrementally update the database schema to keep it in sync with the
application's data model while preserving existing data in the database.
To generate, run, and revert migrations, TypeORM provides a dedicated CLI.
```json
{
  "scripts": {
    "migration:run": "typeorm -f src/ormconfig.json migration:run",
    "migration:create": "typeorm -f src/ormconfig.json migration:create -n",
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

## Services
TBD...

NOTE:
> Name services should be done using verbs instead of nouns! <br />
> There is no “user domain” and there should also be no “UserService”.  <br />
> Instead, we can have “UserRegistrationService” or “UserAuthenticationService”. <br />

By naming our service by the primary business action or process we also make it clear what is the behavior we want it to implement.


