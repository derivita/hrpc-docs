---
title: FAQ
---

Here are some frequently asked questions. Hope you find your answer here :-)

### What is HRPC?

HRPC is a simple , open source remote procedure call (RPC) framework that can run anywhere. It enables client and server applications to communicate transparently, and makes it easier to build connected systems.

HRPC is heavily inspired by HRPC, but designed to be simpler to be more compatible with restricted client and server environments.

### What does HRPC stand for?

**H**TTP **R**emote **P**rocedure **C**alls

**g**RPC **R**emote **P**rocedure **C**alls, of course!

### Why would I want to use HRPC?

The main usage scenarios:

* Low latency, highly scalable, distributed systems.
* Developing mobile clients which are communicating to a cloud server.
* Designing a new protocol that needs to be accurate, efficient and language independent.
* Layered design to enable extension eg. authentication, load balancing, logging and monitoring etc.

### Which programming languages are supported?

Go and Typescript

### How do I get started using HRPC?

You can start with installation of HRPC by following instructions [here](/docs/quickstart). 

### Which license is HRPC under?

All implementations are licensed under [Apache 2.0](https://github.com/derivita/hrpc/blob/master/LICENSE).


### Where is the documentation?

Check out the [documentation](/docs) right here on grpc.io.

### Can I use it in the browser?

Yes, HRPC was specifically designed to be browser native.

### Why is HRPC better/worse than REST?

HRPC largely follows HTTP semantics over HTTP/1. We diverge from typical REST conventions as we use static paths for performance reasons during call dispatch as parsing call parameters from paths, query parameters and payload body adds latency and complexity. We have also formalized a set of errors that we believe are more directly applicable to API use cases than the HTTP status codes.
