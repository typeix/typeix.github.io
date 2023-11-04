---
template: main.html 
title: Resty - Quick Start
---

# Resty
Build REST Apis with Typeix Resty, REST stands for Representational State Transfer, is an architectural style for designing networked applications. 
It was introduced by Roy Fielding in his doctoral dissertation in 2000 and has since become a widely adopted approach 
for building APIs(Application Programming Interfaces) on the internet.

**Key characteristics of REST include:**

**Statelessness**: 
In REST, each request from a client to a server must contain all the information needed to understand and process the request. 
This means that the server does not store any information about the client's state between requests.

**Client-Server Architecture**: 
REST separates the concerns of the client and the server. 
The client is responsible for the user interface and user experience, while the server is responsible for processing and storing data.

**Uniform Interface**: 
RESTful APIs provide a standardized way for clients to interact with resources. 
This includes the use of well-defined endpoints (URIs), standard methods (GET, POST, PUT, DELETE), and consistent data formats (usually JSON or XML).

**Resource-Based**: 
Resources are the core entities that a RESTful API deals with. 
These can be anything from users and articles to products and orders. 
Each resource is uniquely identified by a URI.

**Stateless Communication**: 
Communication between the client and server is stateless, meaning that each request from the client to the server must contain 
all the information needed to understand and process the request.

**Stateless Responses**: 
Responses from the server to the client do not contain any information about the client's state. 
Instead, they provide information about the current state of the requested resource.

**Stateless Operations**: 
RESTful APIs utilize standard HTTP methods (GET, POST, PUT, DELETE) to perform operations on resources. 
These methods correspond to basic CRUD (Create, Read, Update, Delete) operations.

RESTful APIs have become the standard for building web services due to their simplicity, scalability, and compatibility with the HTTP protocol. 
They are widely used in web and mobile applications to enable communication between clients and servers over the internet. 
However, it's important to note that while REST provides guidelines, there is room for interpretation and variation in its implementation.

## Quick Start
```shell
$ npm i -g @typeix/cli
$ typeix new project-name
$ cd project-name
$ typeix start --watch
```

The project directory will be created, node modules and a few other boilerplate
files will be installed, and a src/ directory will be created and populated with several core files.

```text
src
 L app.controller.spec.ts
 L app.controller.ts
 L app.module.ts
 L app.service.ts
 L bootstrap.ts
```

| File                     | Description                             |
| :--                      | --:                                     |    
|`app.controller.spec.ts`  | The unit tests for the controller.      |
|`app.controller.ts`       | A basic controller with a single route. |
|`app.module.ts`           | The root module of the application.     |
|`app.service.ts`          | A basic service with a single method.   |
|`bootstrap.ts`            | Application Server Config File          | 
