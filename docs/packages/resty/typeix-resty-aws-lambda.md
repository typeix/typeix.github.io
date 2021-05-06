---
template: main.html
title: Resty AWS Lambda Adapter
---
# Resty
Using lambda adapter you can simply transform your application to simply run as AWS lambda server.

You can deploy your build with terraform on top of lambda you should run api gateway as proxy which 
forwards requests to lambda, make sure that you assigned correct permissions. 

I will add complete code example and schematics starter!

## Usage
In example below you can find basic application server starter:
```ts
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
export handler = lambdaServer(ApplicationModule, {});
```