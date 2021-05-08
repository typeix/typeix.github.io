---
template: main.html title: Providers
---

# Providers

Providers are a fundamental concept in Typeix Dependency Injection, Typeix uses [`@typeix/di`](/packages/typeix-di)
packages which is delivered and exported directly with `@typeix/resty` package.

Providers are basically classes decorated with `@Injectable()` decorator which are used by DI system to create those
objects on application bootstrap or on request instance, basically you need to define
`@Controller()` provider if you want that your provider is created at request level. All other `@Module()` providers are
created at application bootstrap!

There are three type of providers class, value and factory provider, functions inside providers array are converted to
class providers.

There are three types of providers in total: <br />
`{provde, useClass}` is a class provider. <br />
`{provde, useFactory}` is a factory provider.  <br />
`{provde, useValue}` is a value provider. <br />

Dependent `providers?: Array<Function | IProvider>` inside of provider
(class and factory) are extra dependencies that are created before provider and delivered as dependencies to provider
itself.

```typescript
interface IProvider {
  provide: any;
  useValue?: any;
  useClass?: Function;
  useFactory?: Function;
  providers?: Array<Function | IProvider>;
}
```

## Services
In code below you can see application scope created services:
```ts
import {Injectable, Module, Controller, Inject, GET} from "@typeix/resty";

@Controller({
  path: "/customers"
})
export class CustomerController {
  
  @Inject() customerService: CustomerService;  
    
  @GET()
  findAll() {
    return "This action returns all customers";
  }
}
```
As you can see CustomerService is provided within module providers that means that CustomerService
class will be created at application bootstrap time and always provided as singleton object.
```ts
@Injectable()
export class CustomerService {
  @Inject() dataStore: DataStore;

  findAll() {
    return this.dataStore.query("SELECT * FROM Customers");
  }
}

@Module({
  controllers: [CustomerController],
  providers: [CustomerService]
})
export class ApplicationModule {
    
}
```
However it's possible to create request scoped providers by simply providing them as controller
providers. All request scope providers are memory safe, that means that those objects are automatically 
cleaned up when response body is sent to client! So in that case you don't need to take care if you actually 
are pushing some data inside service context as long as they are not referenced to some live objects
everything is automatically garbage collected.
```ts
import {Injectable, Module, Controller, Inject, GET} from "@typeix/resty";

@Controller({
  path: "/customers",
  providers: [CustomerService]
})
export class CustomerController {

  @Inject() customerService: CustomerService;

  @GET()
  findAll() {
    return "This action returns all customers";
  }
}

@Module({
  controllers: [CustomerController]
})
export class ApplicationModule {}
```


