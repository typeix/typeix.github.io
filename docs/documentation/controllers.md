---
template: main.html
title: Controllers
---
# Controllers
Controllers are responsible for handling incoming requests and returning responses to the client, 
on each request new controller instance is created.

The routing mechanism controls which controller receives which requests. 
Frequently, each controller has more than one route, and different routes can perform different actions.

In order to create a basic controller, we use classes and decorators. 
Decorators associate classes with required metadata.

`@Controller(metadata: IControllerMetadata)` decorator,  is required to define a basic controller.
`path` - actual routing path <br />
`interceptors` - controller level request interceptors <br />
`providers` - providers which are created on each request
```ts
export interface IControllerMetadata {
    path: string;
    interceptors?: Array<RequestInterceptorConstructor>;
    providers?: Array<IProvider|Function>;
}
```

## Routing
In an example below we will create a `@Controller()` with route path of customers.
Using a path in a `@Controller()` decorator allows us to easily group a set of related routes, 
and minimize repetitive code. 

For example, we may choose to group a set of routes that manage interactions with a customer 
entity under the route /customers. 

In that case, we could specify the path customers in the `@Controller()` decorator so that 
we don't have to repeat that portion of the path for each route in the file.
```ts
import { Controller, GET } from "@typeix/resty";

@Controller({
    path: "/customers"
})
export class CustomerController {
  @GET()
  findAll() {
    return "This action returns all customers";
  }
}
```

`@GET()` HTTP request method decorator before the `findAll()` method tells Resty to create 
a handler for a specific endpoint for HTTP requests. 

The endpoint corresponds to the HTTP request method (`GET` in this case) and the route path. 

**What is the route path?** <br />
The route path for a handler is determined by concatenating the
declared for the controller, and any path specified in the request decorator. 

Since we've declared a prefix for every route customers, and haven't added any path 
information in the decorator, Typeix will map `GET` `/customers` requests to this handler.

A path of customers combined with the decorator `@GET("profile")` would produce a route 
mapping for requests like `GET` `/customers/profile`.

In our example above, when a `GET` request is made to this endpoint, Resty routes 
the request to our user-defined `findAll()` method. 

Note that the method name we choose here is completely arbitrary. 
We obviously must declare a method to bind the route to, but Resty 
doesn't attach any significance to the method name chosen.

## Request Object
Resty is a lightweight wrapper around http, https, http2 server, framework provides you access to request and response
objects via dependency Injection.

```ts
import { Controller, GET, Inject } from "@typeix/resty";
import { IncomingMessage, ServerResponse } from "http";

@Controller({
    path: "/customers"
})
export class CustomerController {

  @Inject() response: ServerResponse;  
  @Inject() request: IncomingMessage;
  @Inject() customerService: CustomerService;
    
  @GET()
  findAll() {
    return this.customerService.findAll();
  }
}
```
Those are native node.js [request](https://nodejs.org/api/http.html#http_class_http_incomingmessage) and 
[response](https://nodejs.org/api/http.html#http_class_http_serverresponse) objects.

## Resources
Earlier, we defined an endpoint to fetch the customer resource (GET route). 
We'll typically also want to provide an endpoint that creates new records. <br />
For this, let's create the POST handler:
```ts
import { Controller, GET, Inject, BodyAsBufferInterceptor } from "@typeix/resty";
import { IncomingMessage, ServerResponse } from "http";

@Controller({
    path: "/customers"
})
export class CustomerController {

  @Inject() response: ServerResponse;  
  @Inject() request: IncomingMessage;
  @Inject() customerService: CustomerService;
    
  @GET()
  findAll() {
    return this.customerService.findAll();
  }

  @POST()
  @addRequestInterceptor(BodyAsBufferInterceptor)
  create(@Inject() body: Buffer) {
    const entity = JSON.parse(body.toString());  
    return this.customerService.create(entity);
  }
}
```
Resty provides decorators for standard HTTP methods: 
`@GET()`, `@POST()`, `@PUT()`, `@DELETE()`, `@PATCH()`, `@OPTIONS()`, `@HEAD()`, `@TRACE()` and `@CONNECT()`. <br />
In addition `@OnError("*")` defines route for a custom error handler on any route that handles all of them.
Resty supports custom error handlers on any route, you just need to define witch matches.

## Path Params
Resty supports any regex patterns in defining routes including name capturing regexes as path params which are unsupported in javascript.
Routes are defined by resty in order (one to nth) route, routes are also executed by resty in that order.
```ts
import { Controller, GET, Inject, BodyAsBufferInterceptor, PathParam } from "@typeix/resty";
import { IncomingMessage, ServerResponse } from "http";

@Controller({
    path: "/customers"
})
export class CustomerController {

  @Inject() customerService: CustomerService;
    
  @GET("/<id:(\\d+)>/<type:(\\w+)>")
  findByIdAndType(
      @PathParam("id") id: number, 
      @PathParam("type") customerType: string
  ) {
    return this.customerService.findByIdAndType(id, customerType);
  }
  
  @GET("/blog/<path:(.*)>")
  findPage(@PathParam("path") path: string) {
    return this.customerService.findBlogPostByPath(path);
  }
}
```
As you can see above you can define custom name capturing as path params, however you don't necessary need to define name capturing to use regex.

## Status code and Headers
Changing status codes can easily be done via low level node.js api, or you can simply create custom DI method interceptors for that.
```ts
import { Controller, GET, Inject, BodyAsBufferInterceptor } from "@typeix/resty";
import { IncomingMessage, ServerResponse } from "http";

@Controller({
    path: "/customers"
})
export class CustomerController {

  @Inject() response: ServerResponse;  
  @Inject() request: IncomingMessage;
  @Inject() customerService: CustomerService;

  @POST()
  @addRequestInterceptor(BodyAsBufferInterceptor)
  create(@Inject() body: Buffer) {
    this.response.setHeader("Cache-Control", "none");
    const entity = JSON.parse(body.toString());
    const result = this.customerService.create(entity);
    this.response.writeHead(200, { "Content-Type" : "text/plain" });
    return result;
  }
}
```

Creating decorators as method interceptors to provide nicer custom api can be done with a power of Typeix dependency injection.
In example below you can find a custom method interceptor in which you can inject any of your custom service:
```ts
import { Injectable, Inject, Interceptor, Method, createMethodInterceptor } from "@typeix/resty";
import { ServerResponse } from "http";

@Injectable()
class SetHeaderInterceptor implements Interceptor {

  @Inject() response: ServerResponse;

  invoke(method: Method): any {
    this.response.setHeader(method.decoratorArgs.key, method.decoratorArgs.value);
  }
}
```

After defining interceptor we need to define custom decorator:
```ts
export function SetHeader(key: strin, value: string) {
  return createMethodInterceptor(SetHeader, SetHeaderInterceptor, {key, value});
}
```

Simple usage of custom decorators in controllers:
```ts
import { Controller, GET, Inject, BodyAsBufferInterceptor } from "@typeix/resty";
import { IncomingMessage, ServerResponse } from "http";

@Controller({
    path: "/customers"
})
export class CustomerController {

  @Inject() response: ServerResponse;  
  @Inject() request: IncomingMessage;
  @Inject() customerService: CustomerService;

  @POST()
  @SetHeader("Cache-Control", "none")
  @SetHeader("Content-Type", "application/json")
  @addRequestInterceptor(BodyAsBufferInterceptor)
  create(@Inject() body: Buffer) {
    return this.customerService.create(entity);
  }
}
```
As you can see above it eliminates repeatable code and provides you nice and clean custom api!

## Redirection
Redirections can as well be done via low level API or you can create custom method interceptor!
```ts
import { Controller, GET, Inject, BodyAsBufferInterceptor } from "@typeix/resty";
import { IncomingMessage, ServerResponse } from "http";

@Controller({
    path: "/customers"
})
export class CustomerController {

  @Inject() response: ServerResponse;  
  @Inject() request: IncomingMessage;

  @GET()
  redirect() {
    this.response.setHeader("Location", "/mypage");
    this.response.writeHead(307);
    this.response.end();
  }
}
```

