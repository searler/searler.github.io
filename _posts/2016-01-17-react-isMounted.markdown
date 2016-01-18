---
layout: post
title:  "isMounted deprecation"
date:   2016-01-17 18:00:00
categories:  react 
---


The isMounted function is now [deprecated](http://facebook.github.io/react/blog/2015/12/16/ismounted-antipattern.html).
This lead to much [discussion](https://github.com/facebook/react/issues/3417), 
mostly revolving around cancellable promises. Currently, this requires a wrapper around the promise that arranges 
for the result to ignored once cancellation has been requested.

A [RxJS](http://reactivex.io/) would be particularly convenient since the code can simply unsubscribe from the Observable. 








