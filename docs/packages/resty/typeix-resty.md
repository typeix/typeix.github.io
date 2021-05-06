---
template: main.html
title: REST Node.js Server for Typescript projects
---
# Resty
Fast, unopinionated, minimalist REST framework for building efficient and scalable applications.
It uses modern TypeScript and combines elements of OOP, Functional Programming and Reactive Programming.

Resty has unique features:

* Dependency Injection
* Method Interceptors
* Modular Application Design
* Request Interceptors
* Routing (Dynamic & Static)
* AWS Lambda Adapter
* Supports MVC Structure


Docs and starting point you can find in [getting started](/getting-started) page.

Documentation will be updated and each decorator and interface will be explained in separate section.

## Usage
In example below you can find basic application server starter:
```ts
import {
  pipeServer, Controller, Inject, ResolvedRoute, 
  GET, POST, OnError, RootModule, Logger, Router, 
  addRequestInterceptor, BodyAsBufferInterceptor
} from "@typeix/resty";
import {IncomingMessage, ServerResponse, createServer} from "http";
// resty supports http, https, http2

@Controller({
  path: "/",
  interceptors: [], // controller request interceptors executed in order
  providers: []  // providers created on each request
})
class HomeController {

  @Inject()
  private request: IncomingMessage;

  @Inject()
  private response: ServerResponse;

  @ResolvedRoute()
  private route: IResolvedRoute;

  @GET()
  actionGet() {
    return this.route.method.toUpperCase() + " ACTION";
  }

  @POST()
  @addRequestInterceptor(BodyAsBufferInterceptor)
  actionAjax(body: Buffer) {
    return JSON.stringify(body.toString());
  }

  // will match all routes on this controller
  @OnError("*") 
  errorCase() {
    return "FIRE ERROR CASE";
  }

  @GET("redirect")
  actionRedirect() {
    this.response.setHeader("Location", "/mypage");
    this.response.writeHead(307);
    this.response.end();
  }
}

// DEFINE MODULE 
@RootModule({
  imports: [], // import other modules, created at application bootstrap
  shared_providers: [
    {
      provide: Logger,
      useFactory: () => new Logger({ options: {level: "debug"}})
    },
    Router
  ],
  providers: [], // providers created at application bootstrap
  controllers: [HomeController] // define controllers
})
class ApplicationModule {}

// START SERVER
const server = createServer();
pipeServer(server, ApplicationModule);
server.listen(4000);
```