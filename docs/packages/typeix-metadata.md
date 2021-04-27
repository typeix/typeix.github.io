---
template: main.html
title: High level Typescript decorators package
---

With the introduction of Classes in TypeScript and ES6, there now exist certain scenarios that require additional 
features to support annotating or modifying classes and class members.
Decorators provide a way to add both annotations and a meta-programming syntax for class declarations and members.
Decorators are a stage 2 proposal for JavaScript and are available as an experimental feature of TypeScript.

## Installing
```bash
npm i @typeix/metadata --save
```

## Creating Decorators
Defining custom decorators has not been easier, API will take care that metadataKeys are created correctly 
and passed to low level metadata api.
```ts
import {
    createClassDecorator,
    createParameterDecorator,
    createPropertyDecorator,
    createParameterAndPropertyDecorator, 
    createMethodDecorator
} from "@typeix/metadata";

const Injectable = () => createClassDecorator(Injectable);
const Inject = (token?) => createParameterAndPropertyDecorator(Inject, {token});
const Produces = (type) => createMethodDecorator(Produces, {type});
const Render = (type) => createMethodDecorator(Produces, {type});
const PathParam = (value) => createParameterDecorator(Produces, {value});

@Injectable()
class AService {
}

@Injectable()
class BService {
}

@Injectable()
class RootController {
    @Inject() aService: AService;
    @Inject() bService: BService;

    @Produces("application/json")
    actionIndex(@Inject() first: AService, @Inject() second: BService) {
    }
}

@Injectable()
class HomeController extends RootController {
    
    @Render("home")
    actionHome(
        @Inject() first: AService,
        @Inject(BService) second: BService,
        @PathParam("name") name: string
    ) {
    }
}

const metadata: Array<IMetadata> = getAllMetadataForTarget(HomeController);
```
## Interface IMetadata
```ts
export interface IMetadata {
    args: any; // custom arguments passed via decorator arguments
    metadataKey: string; // unique Decorator key autoganerated by api
    type?: string; //  class, method, property, parameter
    decoratorType?: string; // same as type + two custom mixed types
    decorator?: Function; // actual reference to decorator function
    propertyKey?: string | symbol; // user defined method name or constructor
    paramIndex?: number; // it's defined only in parameter case
    designType?: any;  // typescript design:type
    designParam?: any; // typescript design:paramtypes
    designReturn?: any; // typescript design:returntype
}
```
## IMetadata implementation
All metadata for target can be requested via getAllMetadataForTarget function, which returns list of metadata
example you can find below:
```js
{
    args: {
        token: BService
    },
    decoratorType: "mixed",
    decorator: Inject,
    type: "parameter",
    metadataKey: `@typeix:parameter:Inject:1:${getDecoratorUUID(Inject)}`,
    paramIndex: 1,
    propertyKey: "actionHome",
    designParam: [
        AService,
        BService,
        String
    ]
}
```