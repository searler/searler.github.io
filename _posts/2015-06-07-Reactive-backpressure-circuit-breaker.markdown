---
layout: post
title:  "Reactive stream back pressure and circuit breakers"
date:   2015-06-07 18:20:00
categories:  scala reactive streams akka
---

The [circuit breaker](http://martinfowler.com/bliki/CircuitBreaker.html) forms an important part of 
distributed systems. The literature for circuit breakers 
is generally covers RPC systems, with an [implementation](https://github.com/Netflix/Hystrix/wiki/How-it-Works#CircuitBreaker) that
returns a fallback (or error marking) value when the breaker opens.  

One might thus wonder how this design would be mapped to a [reactive streams](http://www.reactive-streams.org/) (RS) design, which does
not have the call-response structure. 

A key aspect is [back pressure](http://mechanical-sympathy.blogspot.com/2012/05/apply-back-pressure-when-overloaded.html)
which serves to ensure that overload does not occur. Management of overload corresponds to a circuitbreaker (CB) that monitors
the load on the referenced server, either indirectly by measuring response times or directly via instrumentation. 
The behavior seen by the caller is quite different, since the CB will generate some response allowing the caller to proceed. 
The RS implementation will simply fail to request more data from the caller, leaving it unable to proceed. The back pressure
can cascade back through a chain of callers, but will eventually reach the system edge. The entities outside of the system (people,
sensors, etc) will then be forced to discard work. It does allow them to make that decision for themselves but provides no information
with which to make those decisions. A CB protecting an HTTP implementation can provide a 503, with a descriptive body details and a Retry-After header to tell 
the caller how long it needs to wait. 

The [Reactor circuit breaker](http://projectreactor.io/docs/reference/#recipes-circuitbreaker) resolves a different problem.
Clients are consuming a continuous stream of data and the CB masks failures in that producer by providing an alternative stream.
One might argue that CB is not really the best name for this concept. 

The [Akka](http://doc.akka.io/docs/akka-stream-and-http-experimental/1.0-RC3/stream-design.html) design documentation describes a _recovery element_ that 
blocks the propagation of onError back through the stream and replaces the failed stream. This could form part of a reactive CB.

Note that the Reactor implementation expects multiple calls to onError, whereas the Akka design document indicates only one would be seen,
since the stream then collapses. The [specification](https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.0/README.md#specification) calls
for the Subscriber to be cancelled when onError is signalled, which means the stream is stopped (collapsed). In other words,
Reactor is not compliant with the specification.









