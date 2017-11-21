[![Build Status](https://travis-ci.org/salesforce/reactive-grpc.svg?branch=master)](https://travis-ci.org/salesforce/reactive-grpc)
[![codecov](https://codecov.io/gh/salesforce/reactive-grpc/branch/master/graph/badge.svg)](https://codecov.io/gh/salesforce/reactive-grpc)

What is reactive-grpc?
======================
Reactive gRPC is a suite of libraries for using gRPC with [Reactive Streams](http://www.reactive-streams.org/) programming libraries. Using a protocol buffers
compiler plugin, Reactive gRPC generates alternative gRPC bindings for each reactive technology.
The reactive bindings support unary and streaming operations in both directions. Reactive gRPC also builds on top of gRPC's
back-pressure support, to deliver end-to-end back-pressure-based flow control in line with Reactive Streams
back-pressure model.

Reactive gRPC supports the following reactive programming models:

* [RxJava 2](https://github.com/ReactiveX/RxJava)
* [Spring Reactor](https://projectreactor.io/)
* (Eventually) [Java9 Flow](https://community.oracle.com/docs/DOC-1006738)

Usage
=====
See the readme in each technology-specific sub-directory for usage details.

Back-pressure
=============
Reactive gRPC stubs support bi-directional streaming with back-pressure. Under the hood, Reactive gRPC is built atop the vanilla
gRPC service stubs generated by protoc. As such, they inherit gRPC's HTTP/2-based back-pressure model.

Internally, gRPC and Reactive gRPC implement a pull-based back-pressure strategy. At the HTTP/2 layer, gRPC maintains a
buffer of serialized protocol buffer messages. As frames are consumed on the consumer side, the producer is signaled
to transmit more frames. If this producer-side transmit buffer fills, the HTTP/2 layer signals to the gRPC messaging
layer to stop producing new messages in the stream. Reactive gRPC handles this signal, applying back-pressure to Reactive Streams
using the `Publisher` api. Reactive gRPC also implements `Publisher` back-pressure on the consumer side of a stream. As messages
are consumed by the consumer-side `Publisher`, signals are sent down through gRPC and HTTP/2 to request more data.

An example of back-pressure in action can be found in [BackpressureIntegrationTest.java](https://github.com/salesforce/reactive-grpc/blob/master/rx-java/rxgrpc-test/src/test/java/com/salesforce/rxgrpc/BackpressureIntegrationTest.java).

Exception Handling
==============
Exception handling with Reactive gRPC is a little strange due to the way gRPC deals with errors. Servers that produce an error
by calling `onError(Throwable)` will terminate the call with a `StatusRuntimeException`. The client will have its
`onError(Throwable)` subscription handler called as expected.

Exceptions going from client to server are a little less predictable. Depending on the timing, gRPC may cancel
the request before sending any messages due to an exception in the outbound stream.
