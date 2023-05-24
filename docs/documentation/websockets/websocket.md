---
template: main.html
title: Using websockets over http or https server
---

# Quick Start
WebSocket is a computer communications protocol, providing full-duplex communication channels over a single TCP connection.
The WebSocket protocol was standardized by the IETF as RFC 6455 in 2011. 
Typeix resty websocket implementation is a wrapper over [ws](https://www.npmjs.com/package/ws){:target="_blank"} node.js package.

Resty fully supports integration with ws using `@typeix/resty-ws` wrapper library.
You can find full example in [resty starters websockets repository](https://github.com/typeix/resty-starters/tree/master/websockets){:target="_blank"}.

## Installation

Start by installing the required packages:

```bash
$ npm i -g @typeix/cli
$ typeix new ws-project
$ cd ws-project
$ npm i @typeix/resty-ws 
$ npm i --save-dev @types/ws
$ typeix start --watch
```

## Controller
For each websocket connection typeix create new instance of websocket controller and it's isolated from each other, on each socket connection close
controller and it's resources are destroyed.

```ts
import {IAfterConstruct, Inject, Logger} from "@typeix/resty";
import {Arg, Subscribe, WebSocketController, WebSocket} from "@typeix/resty-ws";
import {IncomingMessage} from "http";

@WebSocketController({
  providers: [],
  socketOptions: {
    path: "/ws"
  }
})
export class AppControllerSocket implements IAfterConstruct {
    
  @Inject() logger: Logger;
  @Inject() socket: WebSocket;
  @Inject() request: IncomingMessage;
  
  @Subscribe("message")
  onMessage(@Arg() buffer: Buffer, @Arg() isBinary: boolean) {
    this.logger.debug({
      message: buffer.toString(),
      isBinary
    }, "MESSAGE SENT");
    this.socket.send(JSON.stringify({
      message: "RECEIVED: " + buffer.toString()
    }));
  }
  
  afterConstruct(): void {
    this.logger.info({
      headers: this.request.headers
    });
  }
}
```

## Decorators

| Decorator                  | Info                                                                                                                        |
|----------------------------|-----------------------------------------------------------------------------------------------------------------------------|
| `@WebSocketController()`   | defines websocket controller, for each socket connection new controller is instantiated and providers defined on controller |
| `@Subscribe()`             | subscribe to web socket event `"error", "message", "open", "ping", "pong", "redirect", "upgrade", "unexpected-response"`    |
| `@Arg()`                   | inject argument of subscribed event handler each event type has different argument types and number of arguments            |
| `@Args()`                  | inject list of all handler arguments that are you subscribing, each handler type has different number of arguments          |

## Socket Events
Event types that you can subscribe to with `@Subscribe()` decorator and arguments that you can inject, all injectables from Injector will work as well.

### Message
Emitted when a message is received. data is the message content. isBinary specifies whether the message is binary or not.
```ts
@Subscribe("message")
onMessage(
  @Inject() serviceD: MyCustomServiceD,
  @Arg() buffer: Buffer, 
  @Arg() isBinary: boolean
)
```
### Close
Emitted when the connection is closed. code is a numeric value indicating the status code explaining why the connection has been closed. 
reason is a Buffer containing a human-readable string explaining why the connection has been closed.
```ts
@Subscribe("close")
onClose(
  @Arg() code: number, 
  @Inject() serviceA: MyCustomService,  
  @Arg() reason: Buffer
)
```
### Error
Emitted when an error occurs
```ts
@Subscribe("error")
onError(@Arg() err: Error)
```
Errors may have a `.code` property, matching one of the string values defined below under Error codes.

| Error Code                                  | Info                                                                                                                             |
|---------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------|
| `WS_ERR_EXPECTED_FIN`                       | A WebSocket frame was received with the FIN bit not set when it was expected.                                                    |
| `WS_ERR_EXPECTED_MASK`                      | An unmasked WebSocket frame was received by a WebSocket server.                                                                  |
| `WS_ERR_INVALID_CLOSE_CODE`                 | A WebSocket close frame was received with an invalid close code.                                                                 |
| `WS_ERR_INVALID_CONTROL_PAYLOAD_LENGTH`     | A control frame with an invalid payload length was received.                                                                     |
| `WS_ERR_INVALID_OPCODE`                     | A WebSocket frame was received with an invalid opcode.                                                                           |
| `WS_ERR_INVALID_UTF8`                       | A text or close frame was received containing invalid UTF-8 data.                                                                |
| `WS_ERR_UNEXPECTED_MASK`                    | A masked WebSocket frame was received by a WebSocket client.                                                                     |
| `WS_ERR_UNEXPECTED_RSV_1`                   | A WebSocket frame was received with the RSV1 bit set unexpectedly.                                                               |
| `WS_ERR_UNEXPECTED_RSV_2_3`                 | A WebSocket frame was received with the RSV2 or RSV3 bit set unexpectedly.                                                       |
| `WS_ERR_UNSUPPORTED_DATA_PAYLOAD_LENGTH`    | A data frame was received with a length longer than the max supported length (2^53 - 1, due to JavaScript language limitations). |
| `WS_ERR_UNSUPPORTED_MESSAGE_LENGTH`         | A message was received with a length longer than the maximum supported length, as configured by the `maxPayload` option.           |

### Upgrade
Emitted when response headers are received from the server as part of the handshake. This allows you to read headers from the server, for example 'set-cookie' headers
```ts
@Subscribe("upgrade")
onUpgrade(@Arg() request: IncomingMessage)
```

### Open
Emitted when the connection is established
```ts
@Subscribe("open")
onOpen()
```

### Ping
Emitted when a ping is received
```ts
@Subscribe("ping")
onPing(@Arg() buffer: Buffer)
```

### Pong
Emitted when a pong is received
```ts
@Subscribe("pong")
onPong(@Arg() buffer: Buffer)
```

### Unexpected Response
Emitted when the server response is not the expected one, for example a 401 response. 
This event gives the ability to read the response in order to extract useful information. 
If the server sends an invalid response and there isn't a listener for this event, an error is emitted
```ts
@Subscribe("unexpected-response")
onUnexpectedResponse(
  @Arg() request: ClientRequest, 
  @Arg() response: IncomingMessage
)
```

## Testing
You can fully do integration test for your implementation of sockets.
```ts
import {createServer, IncomingMessage} from "http";

describe("WebSocket", () => {
  it("Integration test", async () => {

    @RootModule({
      shared_providers: [],
      controllers: [AppControllerSocket]
    })
    class WebSocketApplication {

    }
    
    const server = createServer();
    await pipeWebSocket(server, WebSocketApplication);
    await pipeServer(server, WebSocketApplication);

    return await new Promise((resolve) => {
      server.listen(0, () => {
        const address: AddressInfo = <AddressInfo>server.address();
        const ws = new WebSocket("ws://localhost:" + address.port + "/ws", {
          headers: {
            Authorization: "Basic " + Buffer.from("admin:admin").toString("base64")
          }
        });
        ws.on("open", () => ws.send(message));
        ws.on("message", data => messages.push(data.toString()));
        
        setTimeout(() => {
          expect(messages).toContain(message);
          ws.terminate();
          server.close();
          resolve(true);
        }, 1000);
      });
    });
  });
});
```
