---
layout: guides
title: HRPC Concepts
description: |
  This document introduces some key HRPC concepts with an overview of HRPC's architecture and RPC life cycle.
---

It assumes that you've read [What is HRPC?](/docs/guides). For
language-specific details, see the Quick Start, tutorial, and reference
documentation for your chosen language(s), where available (complete reference
docs are coming soon).

<div id="toc" class="toc mobile-toc"></div>

### Overview

#### Service definition

Like many RPC systems, HRPC is based around the idea of defining a service,
specifying the methods that can be called remotely with their parameters and
return types. By default, HRPC uses [protocol
buffers](https://developers.google.com/protocol-buffers/) as the Interface
Definition Language (IDL) for describing both the service interface and the
structure of the payload messages. It is possible to use other alternatives if
desired.

```proto
service HelloService {
  rpc SayHello (HelloRequest) returns (HelloResponse);
}

message HelloRequest {
  string greeting = 1;
}

message HelloResponse {
  string reply = 1;
}
```


HRPC lets you define unary RPCs where the client sends a single request to the server and gets a
  single response back, just like a normal function call.

```proto
rpc SayHello(HelloRequest) returns (HelloResponse){
}
```

HRPC doesn't support any of gRPC's streaming methods.


#### Using the API surface

Starting from a service definition in a .proto file, HRPC provides protocol
buffer compiler plugins that generate client- and server-side code. HRPC users
typically call these APIs on the client side and implement the corresponding API
on the server side.

- On the server side, the server implements the methods declared by the service
  and runs a HRPC server to handle client calls. The HRPC infrastructure decodes
  incoming requests, executes service methods, and encodes service responses.
- On the client side, the client has a local object known as *stub* (for some
  languages, the preferred term is *client*) that implements the same methods as
  the service. The client can then just call those methods on the local object,
  wrapping the parameters for the call in the appropriate protocol buffer
  message type - HRPC looks after sending the request(s) to the server and
  returning the server's protocol buffer response(s).

#### Synchronous vs. asynchronous

Synchronous RPC calls that block until a response arrives from the server are
the closest approximation to the abstraction of a procedure call that RPC
aspires to. On the other hand, networks are inherently asynchronous and in many
scenarios it's useful to be able to start RPCs without blocking the current
thread.

The HRPC programming surface in most languages comes in both synchronous and
asynchronous flavors. You can find out more in each language's tutorial and
reference documentation (complete reference docs are coming soon).

### RPC life cycle

Now let's take a closer look at what happens when a HRPC client calls a HRPC
server method. We won't look at implementation details, you can find out more
about these in our language-specific pages.

#### Unary RPC

First let's look at the simplest type of RPC, where the client sends a single request and gets back a single response.

- Once the client calls the method on the stub/client object, the server is
  notified that the RPC has been invoked with the client's [metadata](#metadata)
  for this call, the method name, and the specified [deadline](#deadlines) if
  applicable.
- The server can then either send back its own initial metadata (which must be
  sent before any response) straight away, or wait for the client's request
  message - which happens first is application-specific.
- Once the server has the client's request message, it does whatever work is
  necessary to create and populate its response. The response is then returned
  (if successful) to the client together with status details (status code and
  optional status message) and optional trailing metadata.
- If the status is OK, the client then gets the response, which completes the
  call on the client side.


#### Deadlines/Timeouts

TODO.
HRPC should allow clients to specify how long they are willing to wait for an RPC to
complete before the RPC is terminated with the error `DEADLINE_EXCEEDED`. On
the server side, the server can query to see if a particular RPC has timed out,
or how much time is left to complete the RPC.

How the deadline or timeout is specified varies from language to language - for
example, not all languages have a default deadline, some language APIs work in
terms of a deadline (a fixed point in time), and some language APIs work in
terms of timeouts (durations of time).

#### RPC termination

In HRPC, both the client and server make independent and local determinations of
the success of the call, and their conclusions may not match. This means that,
for example, you could have an RPC that finishes successfully on the server side
("I have sent all my responses!") but fails on the client side ("The responses
arrived after my deadline!"). It's also possible for a server to decide to
complete before a client has sent all its requests.

