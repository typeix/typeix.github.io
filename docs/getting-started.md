---
template: main.html
title: Getting started
---

# Getting started

## Prerequisites
Please make sure that Node.js (gte >= 12.9.0) is installed on your operating system.

```shell
$ npm i -g @typeix/cli
$ typeix new project-name
$ cd project-name
$ typeix start --watch:
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

