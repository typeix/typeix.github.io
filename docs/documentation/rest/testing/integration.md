---
template: main.html
title: Integration Testing
---

# Integration testing
Unlike unit testing, which focuses on individual modules and classes, 
end-to-end testing covers the interaction of classes and modules.

In the following example we will write a test which starts fake server and will
execute `GET /` route and compare results which is actually sent to client!
```ts
import {Injector, fakeHttpServer} from "@typeix/resty";
import {HomeController} from "@app/controllers/home.controller";
import {TemplateEngine} from "@app/components/templating-engine.service";
import {ApplicationModule} from "@app/application.module";

describe("Home controller", () => {
  let fakeServer;
  
  beforeEach(async () => {
    fakeServer = await fakeHttpServer(ApplicationModule);
  }); 
  
  test("GET /", async () => {
    const injector = Injector.createAndResolve(TemplateEngine, []);  
    const templateEngine = injector.get(TemplateEngine);
    const body = await templateEngine.compileAndRender("home_id", {
      id: "NO_ID",
      name: "this is home page",
      title: "Home page example"
    });
    
    const result = fakeServer.GET("/");
    expect(result.getBody().toString()).toEqual(body.toString());
  });
  
});
```
As you can see there is no extra magic except using dependency injection or `Injector` 
to create instance of templating engine and `fakeServer` to bootstrap application
and execute `GET /` route.
