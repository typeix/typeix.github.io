---
template: main.html
title: Testing
---
# Testing
Automated testing is crucial part of any serious software development life cycle.
Quality and performance tests should take part in automated process. Fast feedback loops and 
coverage reports should be delivered to developers, which increases the productivity and ensures quicker 
release cycles and that quality is meet to organisation goals.

Typeix strives to promote best practices and include effective testing, 
typeix uses [jest](https://jestjs.io/) as default testing framework!


## Unit testing
What is unit testing ? It's a writen code that has a purpose to validate every 
single unit of application software that actually preforms as designed.

Let's take a look resty controller code:
```ts
@Controller({
  path: "/"
})
export class HomeController {
  @Inject() engine: TemplateEngine;

  @GET()
  async actionIndex() : Promise<Buffer> {
    return await this.engine.compileAndRender("home_id", {
      id: "NO_ID",
      name: "this is home page",
      title: "Home page example"
    });
  }
}
```

In the following example we will write a test for `actionIndex` in `HomeController`:
```ts
import {Injector} from "@typeix/resty";
import {HomeController} from "@app/controllers/home.controller";
import {TemplateEngine} from "@app/components/templating-engine.service";

describe("Home controller", () => {
  it("Should test index action", async () => {
    let templateMock = {
      compileAndRender: () => {}
    };
    const injector = Injector.createAndResolve(HomeController, [
      {
        provide: TemplateEngine,
        useValue: templateMock
      }
    ]);
    const controller = injector.get(HomeController);
    const templateSpy = jest.spyOn(templateMock, "compileAndRender");
    await controller.actionIndex();
    expect(templateSpy).toHaveBeenCalledWith("home_id", {
      id: "NO_ID",
      name: "this is home page",
      title: "Home page example"
    });
  });
});
```
In example, you can see that we actually ignore result because we are testing single unit
`actionIndex` and it's behavior!

## End-to-end testing
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
