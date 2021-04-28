---
template: main.html 
title: Logger for Typescript projects
---

# Logger

Resty comes with a built-in [Pino](https://github.com/pinojs/pino) logger wrapper which is used during application
bootstrapping such as displaying scanned routing info (i.e., system logging, bootstrap errors).

You can fully control the behavior of the logging system:

* disable logging entirely
* specify the log level of detail (e.g., display errors, warnings, debug information, etc.)
* customize the default logger by extending it
* create your own custom implementation, to log your own application-level events and messages.

For more advanced logging functionality, you can make use of any Node.js logging package, such
as [Winston](https://github.com/winstonjs/winston), to implement a completely custom, production grade logging system.

## Usage
```ts
interface LoggerOptions {
  options: pino.LoggerOptions;
  stream?: pino.DestinationStream;
}
const options: LoggerOptions = {
    options: {
        level: "error"
    }
};
const logger = new Logger(options);
logger.info({id: 1, whatever: 2}, "Some nice custom message");
```