---
template: main.html
title: Packages Introduction
---
# Introduction
Typeix is Fast, unopinionated, minimalist framework for building efficient and scalable applications and libraries.

In 2016 inspired by Angular first version of typeix was born, which provides an application architecture to allow effortless 
creation of highly testable, scalable, loosely coupled and easily maintainable applications.


## Packages
| Package                                               | Comment                              |
| ----------------------                                | ------------------------------------ |
| `@typeix/utils`                                       | :material-check: Helper library used by @typeix packages internally <br />but nobody stops you from using it :fontawesome-solid-smile: |
| [`@typeix/metadata`](packages/typeix-metadata.md)     | :material-check: High level metadata processor api, you can create own decorators of all types,<br /> library takes care that all values are properly stored & inherited |
| [`@typeix/di`](packages/typeix-di.md)                 | :material-check: Extensive Dependency Injector library inspired by Angular <br />which supports custom extensions and method interceptors |
| [`@typeix/modules`](packages/typeix-modules.md)       | :material-check: Module definition library which comes with ModuleInjector and allows you to<br /> create modules  take care of imports, exports & module providers out of the box |
| [`@typeix/router`](packages/typeix-router.md)         | :material-check: Low level server and routing api, which supports http, https, http2 servers<br /> and comes with dynamic routing options & custom error handlers on any route |
| [`@typeix/logger`](packages/typeix-logger.md)         | :material-check: Wrapper around pino logger library |
| [`@typeix/resty`](packages/resty/typeix-resty.md)     | :material-check: High level REST server implementation |
| [`@typeix/resty-aws-lambda`](packages/resty/typeix-resty-aws-lambda.md)  | :material-check: One line AWS Lambda integration with resty server |
