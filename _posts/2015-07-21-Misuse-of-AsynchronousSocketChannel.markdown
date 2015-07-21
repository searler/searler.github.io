---
layout: post
title:  "Misuse of AsynchronousSocketChannel"
date:   2015-07-21 17:20:00
categories:  Java
---

Two colleagues of mine came across this 
[tutorial](http://examples.javacodegeeks.com/core-java/nio/channels/asynchronoussocketchannel/java-nio-channels-asynchronoussocketchannel-example/)
and decided to use it as a basis for a simple network server. Fortunately, they did not expend too much effort before 
I was able to redirect their approach. 

The colleague was puzzled when he noticed the 100% CPU utilization, which is caused by the following gem!

```
Future result = clientChannel.read(buffer);
while (! result.isDone()) {
   // do nothing
}

```

Another gem appears throughout the code and has the form:

```
Future future = client.connect(hostAddress);
future.get();

```

Which simply converts the asynchronous call into a blocking call. 

The entire example is simply writing a blocking network implementation
using the new asynchronous functionality provided in Java 7. It is not even a particularly good implementation
since the server accepts only one client connection.

The code compiles and runs, but exhibits no understanding of why this particular API exists nor how to apply it.
Presumably using AsynchronousSocketChannel looks fancier than simply using the original blocking socket API. 

The author is a "certified Java and Java EE developer", which makes one ponder the value of those certifications.

As always, do not blindly trust the Internet.

The website unfortunately does not allow comments, which might be for the best.










