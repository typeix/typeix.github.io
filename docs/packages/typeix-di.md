---
template: main.html 
title: Dependency Injection for Typescript projects
---
# Dependency Injection
Dependency Injection (DI) is a design pattern used to implement IoC. <br />
[Learn more about dependency injector in fundamentals section](/documentation/fundamentals/di). <br />
You can use DI lib in any of your projects.

## Installing

```bash
npm i @typeix/di --save
```


## Injector API
All providers are immutable, however you can define mutable provider keys. <br />
If provider is not mutable, and you are trying to create new instance of same provider on current Injector instance, 
injector will throw error.

`Injector.getProviders(provider: IProvider, propertyKey?: string)` will return all providers which needs to be injected, 
if propertyKey is not defined injector will return providers for constructor.

`Injector.getAllMetadataForTarget(provider: IProvider)` will return all metadata, injector cache them internally. 
Works with class provider only.

```typescript
import {Injectable, Injector, Inject} from "@typeix/di";

Injector.createAndResolve(ServiceA, [
  ServiceB
]).then(injector => {
  let service = injector.get(ServiceA);
  return service.getName();
});
```

Async API:
```typescript

class Injector {
    constructor(_parent?: Injector, keys?: Array<any>);
    static getProviders(provider: IProvider, propertyKey?: string): Array<IProvider>;
    // returns all metadata for provider
    static getAllMetadataForTarget(provider: IProvider): Array<IMetadata>;
    // creates new child injector, provider and providers
    static createAndResolveChild(parent: Injector, Class: MixedProvider, providers: Array<MixedProvider>): Promise<Injector>;
    // creates new injector, provider and providers+-
    static createAndResolve(Class: MixedProvider, providers: Array<MixedProvider>): Promise<Injector>;
    // creates new provider and providers
    createAndResolve(provider: IProvider, providers: Array<IProvider>): Promise<any>;
    // clens current injectables and all child injectors & providers
    destroy(): void;
    // check if providier exists on current Injector instance
    has(key: any): boolean;
    // get provider value from current Injector if not found bubble parrent's
    // if not found on any of parents exception is thrown.
    get(provider: string, Class?: IProvider): any;
    get<T>(provider: Type<T>, Class?: IProvider): T;
    // set provider and value
    set(key: any, value: Object): void;
    getParent(): Injector;
    setParent(injector: Injector): void;
    setName(provider: IProvider): void;
    hasChild(injector: Injector): boolean;
    addChild(injector: Injector): this;
}
```

## Sync Injector API
Since version 8.x default Injector behavior is converted to async API, if you want to use sync api you need to use
`Injector.Sync.createAndResolve` or `Injector.Sync.createAndResolveChild` difference is that Async API supports
Async providers, it's allowed to use of async/await in factory and return Promises in value provider!

```typescript
import {Injectable, Injector, Inject} from "@typeix/di";

let injector = Injector.Sync.createAndResolve(ServiceA, [
  ServiceB
]);

let service = injector.get(ServiceA);
service.getName();
```

Sync API:
```ts
class SyncInjector {
  constructor(_parent?: Injector, keys?: Array<any>);
  static getProviders(provider: IProvider, propertyKey?: string): Array<IProvider>;
  // returns all metadata for provider
  static getAllMetadataForTarget(provider: IProvider): Array<IMetadata>;
  // creates new child injector, provider and providers
  static createAndResolveChild(parent: Injector, Class: Function | IProvider, providers: Array<MixedProvider>): Injector;
  // creates new injector, provider and providers
  static createAndResolve(Class: Function | IProvider, providers: Array<MixedProvider>): Injector;
  // creates new provider and providers
  createAndResolve(provider: IProvider, providers: Array<IProvider>): any;
  // clens current injectables and all child injectors & providers
  destroy(): void;
  // check if providier exists on current SyncInjector instance
  has(key: any): boolean;
  // get provider value from current SyncInjector if not found bubble parrent's
  // if not found on any of parents exception is thrown.
  get(provider: string, Class?: IProvider): any;
  get<T>(provider: Type<T>, Class?: IProvider): T;
  // set provider and value
  set(key: any, value: Object): void;
  getParent(): Injector;
  setParent(injector: Injector): void;
  setName(provider: IProvider): void;
  hasChild(injector: Injector): boolean;
  addChild(injector: Injector): this;
}
```
