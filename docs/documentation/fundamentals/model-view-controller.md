---
template: main.html
title: Model View Controller
---
# Model View Controller
The Model-View-Controller (MVC) is a widely used software architecture pattern that separates
the concerns of an application into three main components: the Model, the View, and the Controller.
This pattern aims to enhance the maintainability, scalability, and flexibility of software systems.

Model (M): The Model represents the data and business logic of the application. 
It encapsulates the data structures, database interactions, and algorithms that handle the application's core functionality.
The Model is independent of the user interface and can be modified without affecting the other components.

View (V): The View is responsible for presenting the user interface to the end-users. 
It renders the data from the Model and displays it in a format that is understandable and accessible to the user.
Views can include graphical user interfaces, command-line interfaces, web pages, or any other presentation medium.

Controller (C): The Controller acts as an intermediary between the Model and the View.
It receives input from the user (e.g., through a GUI or a web page), processes it, and triggers the necessary actions in the Model.
The Controller also updates the View to reflect any changes in the Model's state.

By separating the application into these three components, the MVC pattern promotes modularity, making it easier to
develop, test, and maintain software systems. Additionally, it allows for parallel development, as different teams can
work on different components independently. This pattern is widely used in various software development frameworks and
is a fundamental concept in building robust and scalable applications.


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
}
```
NOTE:
> In following example we did not use models, we just send data from controller as mock.

## Render Interceptor
In order to make repeatable TemplateEngineService code obsolete, we will move that code into method interceptor.
```ts

import {
  createMethodInterceptor,
  Inject,
  Injectable, Interceptor, Method
} from "@typeix/resty";
import {TemplateEngineService} from "~/components/templating-engine.service";

@Injectable()
export class RenderInterceptor implements Interceptor {
  @Inject() engine: TemplateEngineService;
  async invoke(method: Method): Promise<any> {
    const data = await method.invoke();
    const result = await this.engine.compileAndRender(method.decoratorArgs.value, data);
    return await method.transform(result);
  }
}

/**
 * Asset loader service
 * @constructor
 * @function
 * @name Render
 *
 * @description
 * RenderInterceptor template
 */
export function Render(value: string) {
  return createMethodInterceptor(Render, RenderInterceptor, {value});
}
```
In following example you can see that Injecting TemplateEngineService is no longer required
because we created custom decorator, and we decorated our controller action.
```ts
import {Inject, Controller, GET, PathParam, ResolvedRoute, IResolvedRoute} from "@typeix/resty";
import {Render} from "~/interceptors/render.interceptor";

@Controller({
  path: "/"
})
export class HomeController {

  @GET()
  @Render("main")
  async actionIndex(): Promise<Buffer> {
    return {
      id,
      name,
      title: "Template engine with typeix"
    };
  }
}
```
