---
template: main.html
title: Getting started
---

# Getting started

## Prerequisites
Please make sure that Node.js (gte >= 12.9.0) is installed on your operating system.

## Description
Typeix supports both typescript and javascript development with babel compiler!


## Typescript
```shell
$ npm i -g @typeix/cli
$ typeix new project-name
$ cd project-name
$ typeix start --watch
```

The project directory will be created, node modules and a few other boilerplate 
files will be installed, and a src/ directory will be created and populated with several core files.

```text
src
 L app.controller.spec.ts
 L app.controller.ts
 L app.module.ts
 L app.service.ts
 L bootstrap.ts
```

| File                     | Description                             |
| :--                      | --:                                     |    
|`app.controller.spec.ts`  | The unit tests for the controller.      |
|`app.controller.ts`       | A basic controller with a single route. |
|`app.module.ts`           | The root module of the application.     |
|`app.service.ts`          | A basic service with a single method.   |
|`bootstrap.ts`            | Application Server Config File          | 

## Javascript 
Babel 7.0.0 introduced a decorators `@babel/plugin-proposal-decorators` plugin for javascript!
Typeix is fully compatible to develop application in javascript with babel compiler!

```shell
$ typeix new project-name -l js
$ cd project-name
$ typeix start --watch
```
The project directory will be created, and in src you can find boilerplate javascript files!

```text
src
 L app.controller.spec.js
 L app.controller.js
 L app.module.js
 L app.service.js
 L bootstrap.js
```
All decorators which are possible to use in typescript you can use them the same way in javascript!
All Typeix packages are compatible with javascript development: 
Resty, Dependency Injection, Metadata, Router, Logger and Resty Lambda!

| File                     | Description                             |
| :--                      | --:                                     |    
|`app.controller.spec.js`  | The unit tests for the controller.      |
|`app.controller.js`       | A basic controller with a single route. |
|`app.module.js`           | The root module of the application.     |
|`app.service.js`          | A basic service with a single method.   |
|`bootstrap.js`            | Application Server Config File          | 
