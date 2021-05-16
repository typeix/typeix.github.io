---
template: main.html
title: TypeORM integration
---
# TypeORM
TypeORM is an ORM that can run in NodeJS, and can be used with TypeScript. 

TypeORM supports both Active Record and Data Mapper patterns, unlike all other JavaScript ORMs 
currently in existence, which means you can write high quality, 
loosely coupled, scalable, maintainable applications the most productive way.

TypeORM provides support for many relational databases: PostgreSQL, Oracle, Microsoft SQL Server,
SQLite, and even NoSQL databases like MongoDB.

Resty framework fully supports integration with typeorm without any extra library or adapter.
You can find full example in [resty starters repository](https://github.com/typeix/resty-starters/tree/master/api-typeorm-sql).

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
Inside data-store module let's create ormconfig.json file:
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

TBD...

