---
template: main.html
title: Unit Testing
---

# Unit testing
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
