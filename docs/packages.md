---
template: main.html
title: Philosophy
---

In 2016 inspired by Angular first version of typeix was born, which provides an application architecture to allow effortless 
creation of highly testable, scalable, loosely coupled and easily maintainable applications.

Typeix today provide set of tools which can be used independently in projects or combined as Node.js application server in 
[@typeix/resty](typeix-resty) package.

## Core Packages
| Package                       | Comment                              |
| ----------------------        | ------------------------------------ |
| `@typeix/utils`               | :material-check: Helper library used by @typeix packages internally <br />but nobody stops you from using it :fontawesome-solid-smile: |
| `@typeix/metadata`            | :material-check: High level metadata processor api, you can create own decorators of all types,<br /> library takes care that all values are properly stored & inherited |
| `@typeix/di`                  | :material-check: Extensive Dependency Injector library inspired by Angular <br />which supports custom extensions and method interceptors |
| `@typeix/modules`             | :material-check: Module definition library which comes with ModuleInjector and allows you to<br /> create modules  take care of imports, exports & module providers out of the box |
| `@typeix/router`              | :material-check: Low level server and routing api, which supports http, https, http2 servers<br /> and comes with dynamic routing options & custom error handlers on any route |
| `@typeix/logger`              | :material-check: Wrapper around pino logger library |
| `@typeix/resty`               | :material-check: High level REST server implementation |
| `@typeix/resty-aws-lambda`    | :material-check: One line AWS Lambda integration with resty server |

## Consulting
With official support, you can get expert help straight from Typeix core team. We provide dedicated technical support, migration strategies,
advice on best practices (and design decisions), PR reviews, and team augmentation.


## Stay in touch
* Author - [Igor Ivanovic](https://twitter.com/igorzg1987)
* Website - [https://typeix.com](https://typeix.com)
* Twitter - [@typeixframework](https://twitter.com/typeixframework)