---
template: main.html 
title: Providers
---

# Providers

Dependencies are services or objects that a class needs to perform its function.x
Providers are a fundamental concept in Typeix Dependency Injection, Typeix uses 
[`@typeix/di`](/packages/typeix-di) package which is delivered and exported 
directly with `@typeix/resty` package.


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

## Scopes
Resty provides two scopes of providers, providers which are created at bootstrap time or also known as
application providers and providers that are created at each request are request providers.
Big difference is that application providers are singletons where request provider is
created and injected at each controller request!

### Application
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

### Request
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

## Injection Points
Once your provider is created by dependency injection system you can use `@Inject()` decorator to 
deliver your object instance to arbitrary provider. 

### Property Injection
`@Inject(token: Type)` decorator is used to inject dependency at property injection point.
If you provide token type, DI will ignore type of property, so in example below
we can se that `@Inject(CustomerService)` will actually inject CustomerService
for iCustomerService property, all injected properties are created on object context,
they are not created on prototype of object! Prototype holds only metadata information!
`@Inject() customerService: CustomerService;` typeix di will inject correct type as long 
as property is actual object type and not an interface!
```ts
import {Controller, Inject} from "@typeix/resty";

@Controller({
  path: "/customers"
})
export class CustomerController {

  @Inject() customerService: CustomerService;
  @Inject(CustomerService) iCustomerService: InterfaceCustomerService;
}
```

### Parameter Injection
All parameter injections are method scoped, and only visible to actual method!
Keep in mind that property injection points are delivered at object creation where
parameters are delivered to method at actual method invocation.
```ts
import {Controller} from "@typeix/resty";

@Controller({
  path: "/customers"
})
export class CustomerController {
    
  indexAction(@Inject() customerService: CustomerService) {

  }
}
```

## Create Provider 
If you want that your provider is created at creation time, you can use 
`@CreateProvider(Function | IProvider)` decorator to automatically create provider for you inside
of Service, Controller, Module or Interceptor!
It works with all three provider types!
```ts
import {CreateProvider, Injectable} from "@typeix/resty";

@Injectable()
export class AwesomeService {
    
  @CreateProvider() customerService: CustomerService;
}
```
Factory Provider
```ts
import {CreateProvider, Injectable} from "@typeix/resty";

@Injectable()
export class AwesomeService {
    
  @CreateProvider({
    provide: CustomerService,
    useFactory: (a, b) => {
        return new CustomerService(a, b);
    },
    providers: [ProviderA, ProviderB]
  }) customerService: CustomerService;
}
```
Value Provider
```ts
import {CreateProvider, Injectable} from "@typeix/resty";

const singleton = new CustomerService();

@Injectable()
export class AwesomeService {
  @CreateProvider({
    provide: CustomerService,
    useValue: singleton
  }) customerService: CustomerService;
}
```

class Provider
```ts
import {CreateProvider, Injectable} from "@typeix/resty";

@Injectable()
export class AwesomeService {
    
  @CreateProvider({
    provide: CustomerService,
    useClass: CustomerService
  }) customerService: CustomerService;
  // is same as
  @CreateProvider(CustomerService) customerService: CustomerService;
  // or even shorter
  @CreateProvider() customerService: CustomerService;
}
```

## Shared Providers
Shared providers are singletons, created once at bootstrap time and visible to all application modules!
Shared provider is just a normal provider which is included in shared providers list
at \@RootModule metadata definition. <br />
You can read more about modules in [modules section](/documentation/modules).
```ts
import {RootModule, AwesomeService} from "@typeix/resty";

@Injectable()
export class AwesomeService {
    
}

@RootModule({
  providers: [],
  shared_providers: [AwesomeService]
})
export class ApplicationModule {
    
 
}
```
