---
template: main.html
title: Application modules for Typescript projects
---
# Modules
Package provides Modules API with Dependency Injection, A module is a class annotated with a `@Module()` decorator.
The `@Module()` decorator provides metadata information to ModuleInjector, that makes use of to organize application 
module dependency graph.

![Tree Traversal](../assets/module-object-tree.png){ align=left width=400 }
Injector uses deep-tree left traversal algorithm to create application module instances, 
that means that deepest three objects are created first. 

Objects would be created in order  <br />
A -> C -> E -> D -> B -> H -> I -> G -> F <br />
if we illustrate that would look like:
```text
       A       H
     /   \     |
    C     E    I
     \   /     |
       D       G
       |      /
       B     /
        \   /
          F
```
You can easily extend application module decorator to provide custom metadata API in own projects, 
due to large application graph application modules should be initialized once on application start or bootstrap time. 

Installing:
```bash
npm i @typeix/modules @typeix/di --save
npm i @types/node typescript --save-dev
```
## Decorators
`@Module()` decorator contains metadata information of import modules, export providers and providers to be created. <br />
`imports` - can only contain modules or module imports <br />
`exports` - providers to be exported which are created or imported at module creation <br />
`providers` - list of providers that are created at module creation 
```ts
export interface IModuleMetadata {
  imports?: Array<Function | IProvider>;
  exports?: Array<Function | IProvider>;
  providers: Array<Function | IProvider>;
}
```
Extending module decorator in custom projects can be done as following:
```ts
import {Module as AModule, IModuleMetadata as AIModuleMetadata} from "@typeix/modules";

export interface IModuleMetadata extends AIModuleMetadata {
    controllers?: Array<Function | IProvider>;
    path?: string;
}

export function Module(config: IModuleMetadata): ClassDecorator {
    if (!isArray(config.exports)) {
        config.exports = [];
    }
    return AModule(config);
}
```

## Usage
Once created modules can be accessed via ModuleInjector, in example we can see that `ApplicationModuleD`
create providers AService and BService and exports them to other modules, what that means is
that same object reference will be provided to `ApplicationModuleC` or module with imports `ApplicationModuleD`,
however `ApplicationModuleB` will receive same instance of service `BService` but `AService` will be newly created object
since is defined as provider in `providers: [AService]`

```ts
import {Module, ModuleInjector} from "@typeix/modules";
import {Injector, Injectable, Inject} from "@typeix/di";

@Injectable
class AService {
}

@Injectable
class BService {
  @Inject() aService: AService;
  doWhatever() {}
}

@Module({
  providers: [AService, BService],
  exports: [AService, BService]
})
class ApplicationModuleD {
  @Inject() bService: BService;
  @Inject() aService: AService;
}
```
ApplicationModuleC implements ApplicationModuleD and BService, AService are same references exported from ApplicationModuleD.
```ts
@Module({
  imports: [ApplicationModuleD],
  exports: [AService, BService],
  providers: []
})
class ApplicationModuleC {
  @Inject() bService: BService;
  @Inject() aService: AService;
}
```
ApplicationModuleB implements ApplicationModuleC and BService is same reference however AService is new instance 
of AService class, because it's provided as new provider on ApplicationModuleB.
```ts
@Module({
  imports: [ApplicationModuleC],
  providers: [AService]
})
class ApplicationModuleB {
  @Inject() bService: BService;
  @Inject() aService: AService;
}

```


## ModuleInjector
**NOTE:** <br/> By default all providers are immutable, however you can define mutable provider keys. <br />
If provider is not mutable, and you are trying to create new instance of same provider on current ModuleInjector instance,
injector will throw error.

### Async
Since version 8.x default Injector behavior is converted to async API, if you want to use sync api you need to use
`ModuleInjector.Sync.createAndResolve`  difference is that Async API supports Async providers, 
it's allowed to use of async/await in factory and return Promises in value provider!

`ModuleInjector.createAndResolve(Class, sharedProviders)` special sharedProviders property will create all providers
which are provided and visible to all modules, however if module have same provider provided in `providers` module metadata,
new instance will be delivered to that module.
```ts
class ModuleInjector {
    static createAndResolve(Class: Function | IProvider, sharedProviders: Array<Function | IProvider>, mutableKeys?: Array<any>): Promise<ModuleInjector>;
    get(Class: Function | IProvider): any;
    getInjector(Class: Function | IProvider): Injector;
    has(Class: IProvider | Function): boolean;
    remove(Class: Function | IProvider): boolean;
    getAllMetadata(): Map<any, IModuleMetadata>;
    createAndResolveSharedProviders(providers: Array<Function | IProvider>): Promise<Injector>;
    createAndResolve(Class: Function | IProvider, mutableKeys?: Array<any>): Promise<Injector>;
}
```


Once modules are created by ModuleInjector all objects and references can be accessed via API
```ts
const injector = await ModuleInjector.createAndResolve(ApplicationModuleB);
```

### Sync API
Sync api is accessible via `ModuleInjector.Sync.createAndResolve(Class, sharedProviders)` or by simply importing `SyncModuleInjector`
```ts
class SyncModuleInjector {
    static createAndResolve(Class: Function | IProvider, sharedProviders: Array<Function | IProvider>, mutableKeys?: Array<any>): SyncModuleInjector;
    get(Class: Function | IProvider): any;
    getInjector(Class: Function | IProvider): SyncInjector;
    has(Class: IProvider | Function): boolean;
    remove(Class: Function | IProvider): boolean;
    getAllMetadata(): Map<any, IModuleMetadata>;
    createAndResolveSharedProviders(providers: Array<Function | IProvider>): SyncInjector;
    createAndResolve(Class: Function | IProvider, mutableKeys?: Array<any>): SyncInjector;
}
```

In following example we can see sync api usage:
```ts
const injector = ModuleInjector.Sync.createAndResolve(ApplicationModuleB);
const dModule: ApplicationModuleD = injector.get(ApplicationModuleD);
const cModule: ApplicationModuleC = injector.get(ApplicationModuleC);
const bModule: ApplicationModuleB = injector.get(ApplicationModuleB);
bModule.myAction();
const bModuleInjector = injector.getInjector(ApplicationModuleB);
const bService = bModuleInjector.get(BService);
bService.doWhatever();
```
In example above injected service references are evaluated as true
```ts
const dModuleInjector = injector.getInjector(ApplicationModuleD);
const cModuleInjector = injector.getInjector(ApplicationModuleC);
const bModuleInjector = injector.getInjector(ApplicationModuleB);

bModuleInjector.get(BService) === cModuleInjector.get(BService)
bModuleInjector.get(BService) === dModuleInjector.get(BService)

cModuleInjector.get(AService) === dModuleInjector.get(AService)
bModuleInjector.get(AService) != cModuleInjector.get(AService)
bModuleInjector.get(AService) != dModuleInjector.get(AService)
```
