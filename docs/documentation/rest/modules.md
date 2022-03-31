---
template: main.html
title: Modules
---

# Modules
Each resty application needs to have only one `@RootModule` and it can have arbitrary number of 
`@Modules` which are imported at other modules or at root module definition.
A module is a class annotated with a `@Module()` or `@RootModule()` decorator. 

You can find about module package specifics in [`@typeix/modules`](/packages/typeix-modules) package.
However Resty framework have extra module specifics which are implemented on top of `@typeix/modules`
implementation.

There are two types of resty modules a module and a root module:
```typescript
interface ModuleMetadata {
  imports?: Array<Function | IProvider>;
  exports?: Array<Function | IProvider>;
  controllers?: Array<Function | IProvider>;
  path?: string;
  providers: Array<Function | IProvider>;
}
```
A `@Module` is a module which is imported by other module or by `@RootModule`.
A `@RootModule` is used only once in application, and it needs to be used by resty pipeServer function!
```typescript
interface RootModuleMetadata extends ModuleMetadata {
  controllers: Array<Function | IProvider>;
  shared_providers?: Array<Function | IProvider>;
}
```

## Feature modules
Feature modules are modules which are belonging to same application domain, specific business feature and 
helps to keep code more organized, manage complexity and to implement SOLID principles.

Let's take a look at one example:
```text
src
 L modules
    L admin
        L services
            L admin.service.ts
        L controllers
            L admin.controller.ts
        L admin.module.ts
```
Admin service:
```ts
import {Injectable} from "@typeix/resty";

@Injectable()
export class AdminService {
    
}
```
Admin controller:
```ts
import {Module} from "@typeix/resty";

@Controller({
  path: "/"
})
export class AdminController {
    
}
```
Admin module:
```ts
import {Module} from "@typeix/resty";
import {AdminController} from "./controllers/admin.controller";
import {AdminService} from "./services/admin.service";

@Module({
  path: "/admin",
  controllers: [AdminController],
  providers: [AdminService]
})
export class AdminModule {
    
}
```
Root Module Definition:
```ts
import {Module} from "@typeix/resty";
import {AdminModule} from "./modules/admin/admin.module";

@Module({
  imports: [AdminModule]
})
export class ApplicationModule {
    
}
```

## Sharing Providers
It's possible to share providers between modules in two ways, globally by defining them in `@RootModule`
or locally by exporting providers inside of module.

In following example **AdminService is exported to other module which imports this module!**
```ts
import {Module} from "@typeix/resty";
import {AdminController} from "./controllers/admin.controller";
import {AdminService} from "./services/admin.service";

@Module({
  path: "/admin",
  controllers: [AdminController],
  providers: [AdminService],
  exports: [AdminService]
})
export class AdminModule {
    
}
```
We can as well share providers inside of root module, but keep in mind that change is actually making 
that provider visible to all modules! <br />
Root Module Definition:
```ts
import {Module} from "@typeix/resty";
import {AdminService} from "./modules/admin/services/admin.service";

@Module({
  shared_providers: [AdminService]
})
export class ApplicationModule {
    
}
```
## Module exporting
It's possible to share providers by importing and exporting providers from modules.  <br />
Providers that must be exported to other modules needs to be defined in `exports` metadata
inside of module metadata.
```ts
import {Module} from "@typeix/resty";
import {AdminController} from "./controllers/admin.controller";
import {AdminService} from "./services/admin.service";

@Module({
  path: "/admin",
  providers: [AdminService],
  exports: [AdminService]
})
export class AdminModule {
    
}
```
Once module exports providers that wants to export, module that imports `GuestModule` must define
`imports` or modules to be imported! <br />
If you want that same service is re-exported it's possible to re-export by defining provider exports to
imported module!
```ts
import {Module} from "@typeix/resty";
import {AdminModule} from "./modules/admin/admin.module";
import {AdminService} from "./modules/admin/services/admin.service";

@Module({
  path: "/guest",
  imports: [AdminModule],
  exports: [AdminService]
})
export class GuestModule {
    
}
```

## Dynamic modules
Resty framework supports dynamic modules and controllers via resty routing, but we will tackle this 
topic in routing documentation!
