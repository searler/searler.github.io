---
layout: post
title:  "Stream I/O in RxJava"
date:   2014-02-08 14:00:00
categories: RxJava Camel
---
The Netflix RxJava implementation [explicitly](https://github.com/Netflix/RxJava/issues/634) drops the .Net Reactive Extensions operations that are used for (async) I/O. The available documentation is essentially silent on how one might implement simple I/O operations, such as reading from stdin. 

A web search indicates that such operations are generally implemented using recursive schedulers, a topic which also has limited documentation. 

This code snippet illustrates how this might be implemented.
{% gist 8904778 %}

Obviously this code consumes a thread to handle the blocking I/O, which is provided by a self contained scheduler to avoid the performance [defect](https://github.com/Netflix/RxJava/issues/711).

[Camel RX](http://camel.apache.org/rx.html) integrates its existing Components with rxjava, allowing a very simple implementation
{% gist 8904954 %}
Without the `scanStream=true` option the output is delayed by one line.
