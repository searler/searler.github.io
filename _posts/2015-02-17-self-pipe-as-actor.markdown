---
layout: post
title:  "Implementing actors in shell scripts using self-pipe"
date:   2015-02-17 09:20:00
categories:  scala akka camel reactive
---

This [discussion](http://www.sitepoint.com/the-self-pipe-trick-explained/) of the self pipe
and how it is used to reliably handle signals is very interesting. A neat *nix technique that
I had not previously encountered.

This technique is actually implementing an actor.
* Each signal corresponds to a message
* Each signal handler corresponds to a message handler
* The pipes provides the inbox
* The select implements the message processing (Akka receive)

The goals are identical: map asynchronous requests into a thread safe process.












