---
template: main.html
title: Dependency Injection for Typescript projects
---
# Dependency Injection
Dependency Injection (DI) is a design pattern used to implement IoC. It allows the creation of dependent objects outside
of a class and provides those objects to a class through different ways. Using DI, we move the creation and binding of
the dependent objects outside of the class that depends on them, typeix uses decorator pattern to implement 
dependency injection. 

In object-oriented programming, the decorator pattern is a design pattern that allows behavior 
to be added to an individual object, dynamically, without affecting the behavior of other objects from the same class.

The decorator pattern is often useful for adhering to the Single Responsibility Principle, 
as it allows functionality to be divided between classes with unique areas of concern.

Decorator use can be more efficient than subclassing, because an object's behavior can be 
augmented without defining an entirely new object.


## Implementation
In order to create an object of some class you need to decorate class with  @Injectable and injection points via @Inject,
then when creating an instance you need to provide DI what to construct and which providers to create.
```typescript
import {Injectable, Injector, Inject} from "@typeix/di";

@Injectable()
class NameService {

    @Inject("name") private name: string;
    
    getName() {
        return this.name;
    }
}

@Injectable()
class ChildService {

    @Inject() nameService: NameService;

    getName(): string {
        return this.nameService.getName();
    }
}

Injector.createAndResolve(ChildService, [
  {provide: "name", useValue: "Igor"},
  NameService
]).then(injector => {
  let service = injector.get(ChildService);
  return service.getName();
});
```



## Decorators

| Decorator           | Info                                 |
| ------------------- | ------------------------------------ |
| `@Injectable()`     | specifies that Typeix can use this class in the DI system. <br /> decorators can be used on classes only. |
| `@Inject()`         | specifies DI system, which object to provide when object is created, <br /> if there is no provider defined DI will throw error with explanation. |
| `@AfterConstruct()` | specifies DI system, what to execute immediately after object is created. |
| `@CreateProvider()` | decorator specifies DI system, to create new provider on injection point, <br /> this will create new object even if there is a provider defined in parent Injector. |

## Provider Types
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



## Interceptors
An interceptor pattern is a software design pattern that is used when software systems or frameworks want to offer a way
to change, or augment, their usual processing cycle.
```ts
import {Injectable, Injector, Inject, createMethodInterceptor} from "@typeix/di";

@Injectable()
class LoggerInterceptor implements Interceptor {

    @Inject() logger: Logger;

    async invoke(method: Method): Promise<any> {
        const result = await method.invoke();
        this.logger.info(method.args.message, result);
        return result;
    }
}

export function Logger(message) {
    return createMethodInterceptor(Logger, LoggerInterceptor, {message});
};

@Injectable()
class ChildService {
    @Logger("Logging ChildService.getName result: ")
    getName(): string {
        return this.name;
    }
}
```
