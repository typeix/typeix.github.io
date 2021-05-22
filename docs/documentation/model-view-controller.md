---
template: main.html
title: Model View Controller
---
# Model View Controller
Model–view–controller is a software design pattern commonly used for developing user interfaces 
that divides the related program logic into three interconnected elements. <br />

![Model View Controller](/assets/mvc.png)


In order to create an MVC app, we also need a template engine to render our HTML views.
Typeix resty support any templating engine, for this example we are going to use handlebars.

## Installation
Let's start by installing typeix cli and creating new project!
```bash
npm i -g @typeix/cli
```
Let's create new project:
```bash
typeix new app-mvc
cd app-mvc
```
Let's install handlebars:
```bash
npm i --save handlebars
npm i --save-dev @types/handlebars
```

Goal is to create following structure:
```text
src
 L services
    L template.service.ts
 L interceptors
    L render.interceptor.ts   
 L app.controller.ts
 L app.service.ts
 L app.module.ts
 L bootstrap.ts
views
 L main.hbs
```

## Templates
All templates we should keep outside source files, inside views folder:
```shell
mkdir views
```

Let's create custom template main.hbs:
```handlebars
<!DOCTYPE html>
<html lang="en">
<head>
  <link rel="icon" type="image/png" href="/favicon.png" />
  <link rel="stylesheet" href="/assets/css/main.css" />
  <meta charset="UTF-8">
  <title>{{title}}</title>
</head>
<body>
<img src="/assets/logo.png" width="100"/>
<h1>Headline: {{name}}</h1>
<p>
  Methods id: {{id}} name: {{name}};
</p>
</body>
</html>
```

## Service
Let's create a template service which is responsible for loading, compiling and rendering template from disk.

NOTE:
> In following example we are going to load and compile template and store it in memory
> for production use it's better to precompile template and loading it from disk as javascript file.


```ts
import {normalize} from "path";
import {readFile} from "fs";
import {Injectable} from "@typeix/resty";
import {compile} from "handlebars";


@Injectable()
export class TemplateEngineService {

  templates: Map<string, HandlebarsTemplateDelegate> = new Map();

  /**
   * Gets template path
   * @return {String}
   */
  static getTemplatePath(name: String): string {
    return normalize(process.cwd() + "/views/" + name + ".hbs");
  }

  /**
   * Read file from disk
   * @param template
   */
  async readFile(template: String): Promise<HandlebarsTemplateDelegate> {
    const path = TemplateEngineService.getTemplatePath(template);
    if (this.templates.has(path)) {
      return Promise.resolve(this.templates.get(path));
    }
    return new Promise((resolve, reject) => {
      readFile(path, {encoding: "utf8"},
        (err, data) => {
          if (err) {
            reject(err);
          } else {
            const tpl = compile(data);
            this.templates.set(path, tpl);
            resolve(tpl);
          }
        }
      );
    });
  }

  /**
   * Load template from disk
   * @param template
   * @param data
   * @returns {NodeJS.ReadableStream}
   */
  async compileAndRender(template: String, data: any): Promise<Buffer> {
    const tpl = await this.readFile(template);
    const html = tpl(data);
    return Buffer.from(html);
  }
}
```

## Controller
In controllers, we need to use template engine service to compile and render template with custom data.
```ts
import {Inject, Controller, GET, PathParam, ResolvedRoute, IResolvedRoute} from "@typeix/resty";
import {TemplateEngineService} from "~/services/templating-engine.service";
import {Render} from "~/interceptors/render.interceptor";

@Controller({
  path: "/"
})
export class HomeController {

  @Inject() engine: TemplateEngineService;
  
  @GET()
  async actionIndex(): Promise<Buffer> {
    return await this.engine.compileAndRender("main", {
      id: "NO_ID",
      name: "this is home page",
      title: "Home page example"
    });
  }

  @Render("main")
  @GET("/params/<id:(\\d+)>/<name>")
  async actionId(@PathParam("id") id: number, @PathParam("name") name: string): Promise<any> {
    return {
      id,
      name,
      title: "Template engine with typeix"
    };
  }
}
```
NOTE:
> In following example we did not use models, we just send data from controller as mock.

## Render Interceptor
TBD.. 
