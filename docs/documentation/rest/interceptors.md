---
template: main.html
title: Interceptors
---
# Interceptors
What is interceptor? <br />
Interceptor is a class annotated with `@Injectable` which have access to intercepted methods, 
it can control execution, catch exceptions, bind extra logic and transform data that are returned 
in original method.

Typeix support two type of interceptors, a request interceptor and dependency injection interceptor. <br />

## Request
Request interceptors are implemented in resty framework package and thy can be used only with resty.

 * Executed in order
 * Is request scoped
 * Can be assigned at controller level
 * Can be defined at singe request level
 * First return wins - if you have multiple interceptors at controller level, first one 
in order that returns result will stop execution of other interceptors!
   
```ts
interface InterceptedRequest {
  handler: () => any;
  injector: Injector;
  route: IResolvedRoute;
  request: IncomingMessage | Http2ServerRequest;
  response: ServerResponse | Http2ServerResponse;
  args: any;
}
```   
Caching interceptor example:
```ts
import {Injectable, Inject, RequestInterceptor, InterceptedRequest} from "@typeix/resty";

@Injectable()
export class CacheInterceptor implements RequestInterceptor {
  @Inject() cacheProvider: InMemoryCache;

  async invoke(method: InterceptedRequest): Promise<any> {
    if (await this.cacheProvider.has(method.route.path)) {
      return await this.cacheProvider.get(method.route.path);
    } else {
      this.cacheProvider.set(method.route.path, await method.handler(), 120);
    }
  }
}
```
Controller implementation:
```ts
import {Controller, PathParam} from "@typeix/resty";

@Controller({
  interceptors: [CacheInterceptor]
})
export class PostsController {

    @Render("posts")
    @GET("/<path:(.*)>")
    getPost(@PathParam("path") path: string)  {
        return this.postsService.getPostsByPath(path);
    }
}
```

## Method 
Method interceptors are implemented in dependency injection package, and it can be used with Injector.

 * Executed in order
 * All interceptors are executed
 * Return can be transformed
 * Can be defined at any method (service, module, controller etc.)
```ts
 interface Method {
  invoke: () => any;
  transform: (data: any) => any;
  readonly injector: Injector;
  readonly decoratorArgs: any;
}
```

Typeix is un opinionated framework, and it's not bundled with any templating engine, in 
order to use any templating engine you need to install it, create a service and using
handy decorator you can remove boilerplate code!
```ts
@Injectable()
export class RenderInterceptor implements Interceptor {
  @Inject() engine: TemplateEngine;
  async invoke(method: Method): Promise<any> {
    const data = await method.invoke();
    const result = await this.engine.compileAndRender(method.decoratorArgs.value, data);
    return await method.transform(result);
  }
}

export function Render(value: string) {
  return createMethodInterceptor(Render, RenderInterceptor, {value});
}

```

Setting header use case:
```ts
import { Injectable, Inject, Interceptor, Method, createMethodInterceptor, isObject, isString } from "@typeix/resty";
import { ServerResponse } from "http";

@Injectable()
class SetHeaderInterceptor implements Interceptor {

  @Inject() response: ServerResponse;

  invoke(method: Method): any {
    if (isObject(method.decoratorArgs.key)) {
      Object.keys(method.decoratorArgs.key).forEach(key => {
        this.response.setHeader(key, Reflect.get(method.decoratorArgs.key, key));
      });
    } else if (isString(method.decoratorArgs.key) && isString(method.decoratorArgs.value)) {
      this.response.setHeader(method.decoratorArgs.key, method.decoratorArgs.value);
    }
  }
}
```
After defining interceptor we need to define custom decorator:
```ts
export function SetHeader(key: string | {[key:string]: string}, value?: string) {
  return createMethodInterceptor(SetHeader, SetHeaderInterceptor, {key, value});
}
```
