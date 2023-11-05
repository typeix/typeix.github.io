---
template: main.html
title: Dependency Injection for Typescript projects
---
# Dependency Injection
Dependency Injection (DI) is a design pattern used in software development to promote loose coupling and 
improve the maintainability, testability, and scalability of applications. It's a technique where a component's dependencies 
(i.e., the objects or services it relies on) are provided to it from an external source, rather than being created within the component itself.

**Here are key concepts of Dependency Injection:**

**Inversion of Control (IoC):** 
DI is a specific implementation of the more general IoC principle. In IoC, the control over the instantiation and 
management of dependencies is inverted or "inverted" from the component itself to an external entity (often a container or framework).

**Components and Dependencies:** 
In a software system, components are pieces of code or classes that perform specific functions. 
Dependencies are the objects, services, or resources that a component needs to accomplish its tasks.


**Dependency Injection Container (DIC):** 
A DIC is a framework or container that manages the instantiation and wiring of components and their dependencies. 
It simplifies the process of configuring and injecting dependencies, reducing the manual effort required.

**Benefits of DI:**

**Testability:** 
DI makes it easier to write unit tests for components because dependencies can be replaced with mock or stub objects for testing purposes.

**Flexibility:** 
It allows for easy swapping of implementations or configurations without modifying the component's code.

**Maintainability:** 
DI promotes cleaner, more modular code by explicitly defining dependencies and their relationships.

**Reduced Coupling:** 
Components are not tightly bound to specific implementations of their dependencies, making it easier to change or extend functionality.

**Drawbacks of DI:**

**Learning Curve:** 
Implementing DI may require developers to understand and adopt new patterns and practices.

**Potential Complexity:** 
In large applications, managing a large number of dependencies and their configurations can become complex without the use of a DI container.

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
