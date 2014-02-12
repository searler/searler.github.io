---
layout: post
title:  "Trival HTTP implementation using RxJava and Camel"
date:   2014-02-11 22:00:00
categories: RxJava Camel
---

The earlier experimentation with stdin was extended to a more realistic scenario, using a HTTP endpoint that accepts a POST request, which is ignored. The goal was to explore how to specify the wiring and ensure there are no performance surprises.

The first implementation uses a conventional Camel route.
```java
public static class NoopBean {
    public void accept(String string){
    }
}


CamelContext camelContext = new DefaultCamelContext();
      
RouteBuilder builder = new RouteBuilder() {
   public void configure() {
      from("jetty:http://localhost:6060/camel").bean(NoopBean.class);;
   }
};
     
camelContext.addRoutes(builder);
     
camelContext.start();   
```

The second implementation uses RxJava for wiring. 
```java
CamelContext camelContext = new DefaultCamelContext();
ReactiveCamel rx = new ReactiveCamel(camelContext);
Observable<String> observable = rx.toObservable(
     "jetty:http://localhost:7070/rxjava", String.class);
observable.toBlockingObservable().forEach(new Action1<String>() {
    @Override
       public void call(String m) {
         }
    });
```

A simplistic benchmark against these implementations indicates RxJava is actually a little faster than the straight Camel route!

