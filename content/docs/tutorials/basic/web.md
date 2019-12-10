---
layout: tutorials
title: HRPC Basics - Web
group: basic
short: Web
---

This tutorial provides a basic introduction on how to use
HRPC-Web from browsers.

By walking through this example you'll learn how to:

- Define a service in a .proto file.
- Generate client code using the protocol buffer compiler.
- Use the HRPC-Web API to write a simple client for your service.

It assumes a passing familiarity with [protocol
buffers](https://developers.google.com/protocol-buffers/docs/overview).

<div id="toc"></div>

<a name="why-grpc"></a>

### Why use HRPC and HRPC-Web?

With HRPC you can define your service once in a .proto file and implement
clients and servers in any of HRPC's supported languages, which in turn can be
run in environments ranging from servers inside Google to your own tablet - all
the complexity of communication between different languages and environments is
handled for you by HRPC. You also get all the advantages of working with
protocol buffers, including efficient serialization, a simple IDL, and easy
interface updating. HRPC-Web lets you access HRPC services built in this manner
from browsers using an idiomatic API.


<a name="setup"></a>

### Define the Service

The first step when creating a HRPC service is to define the service methods
and their request and response message types using protocol buffers. In this
example, we define our `EchoService` in a file called
[`echo.proto`](https://github.com/grpc/grpc-web/blob/0.4.0/net/grpc/gateway/examples/echo/echo.proto).
For more information about protocol buffers and proto3 syntax, please see the
[protobuf documentation][].

```protobuf
message EchoRequest {
  string message = 1;
}

message EchoResponse {
  string message = 1;
}

service EchoService {
  rpc Echo(EchoRequest) returns (EchoResponse);
}
```

### Implement HRPC Backend Server

Next, we implement our EchoService interface using Node in the backend HRPC
`EchoServer`. This will handle requests from clients. See the file
[`node-server/server.js`](https://github.com/grpc/grpc-web/blob/master/net/grpc/gateway/examples/echo/node-server/server.js)
for details.

You can implement the server in any language supported by HRPC. Please see
the [main page][] for more details.

```js
function doEcho(call, callback) {
  callback(null, {message: call.request.message});
}
```


### Configure the Envoy Proxy

In this example, we will use the [Envoy](https://www.envoyproxy.io/)
proxy to forward the HRPC browser request to the backend server. You can see
the complete config file in
[envoy.yaml](https://github.com/grpc/grpc-web/blob/master/net/grpc/gateway/examples/echo/envoy.yaml)

To forward the HRPC requests to the backend server, we need a block like
this:

```yaml
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 8080 }
    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
        config:
          codec_type: auto
          stat_prefix: ingress_http
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match: { prefix: "/" }
                route: { cluster: echo_service }
          http_filters:
          - name: envoy.grpc_web
          - name: envoy.router
  clusters:
  - name: echo_service
    connect_timeout: 0.25s
    type: logical_dns
    http2_protocol_options: {}
    lb_policy: round_robin
    hosts: [{ socket_address: { address: node-server, port_value: 9090 }}]
```

You may also need to add some CORS setup to make sure the browser can request
cross-origin content.


In this simple example, the browser makes HRPC requests to port `:8080`. Envoy
forwards the request to the backend HRPC server listening on port `:9090`.



### Generate Protobuf Messages and Service Client Stub


To generate the protobuf message classes from our `echo.proto`, run the
following command:

```sh
$ protoc -I=$DIR echo.proto \
  --js_out=import_style=commonjs:$OUT_DIR
```

The `import_style` option passed to the `--js_out` flag makes sure the
generated files will have CommonJS style `require()` statements.


To generate the HRPC-Web service client stub, first you need the HRPC-Web
protoc plugin. To compile the plugin `protoc-gen-grpc-web`, you need to run
this from the repo's root directory:

```sh
$ cd grpc-web
$ sudo make install-plugin
```

To generate the service client stub file, run this command:

```sh
$ protoc -I=$DIR echo.proto \
  --grpc-web_out=import_style=commonjs,mode=grpcwebtext:$OUT_DIR
```

In the `--grpc-web_out` param above:
  - `mode` can be `grpcwebtext` (default) or `grpcweb`
  - `import_style` can be `closure` (default) or `commonjs`

Our command generates the client stub, by default, to the file
`echo_grpc_web_pb.js`.


### Write JS Client Code

Now you are ready to write some JS client code. Put this in a `client.js` file.

```js
const {EchoRequest, EchoResponse} = require('./echo_pb.js');
const {EchoServiceClient} = require('./echo_grpc_web_pb.js');

var echoService = new EchoServiceClient('http://localhost:8080');

var request = new EchoRequest();
request.setMessage('Hello World!');

echoService.echo(request, {}, function(err, response) {
  // ...
});
```

You will need a `package.json` file

```json
{
  "name": "grpc-web-commonjs-example",
  "dependencies": {
    "google-protobuf": "^3.6.1",
    "grpc-web": "^0.4.0"
  },
  "devDependencies": {
    "browserify": "^16.2.2",
    "webpack": "^4.16.5",
    "webpack-cli": "^3.1.0"
  }
}
```

### Compile the JS Library


Finally, putting all these together, we can compile all the relevant JS files
into one single JS library that can be used in the browser.

```sh
$ npm install
$ npx webpack client.js
```

Now embed `dist/main.js` into your project and see it in action!


[protobuf documentation]:https://developers.google.com/protocol-buffers/
[main page]:/docs/
