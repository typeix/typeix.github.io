---
template: main.html
title: Providers
---
# Providers
Providers are a fundamental concept in Typeix Dependency Injection, Typeix uses [`@typeix/di`](/packages/typeix-di) packages
which is delivered and exported directly with `@typeix/resty` package.

There are three type of providers class, value and factory provider,
functions inside providers array are converted to class providers.

There are three types of providers in total: <br />
`{provde, useClass}` is a class provider. <br />
`{provde, useFactory}` is a factory provider.  <br />
`{provde, useValue}` is a value provider. <br />

Dependent `providers?: Array<Function | IProvider>` inside of provider
(class and factory) are extra dependencies that are created before provider and
delivered as dependencies to provider itself.
```typescript
interface IProvider {
    provide: any;
    useValue?: any;
    useClass?: Function;
    useFactory?: Function;
    providers?: Array<Function | IProvider>;
}
```
