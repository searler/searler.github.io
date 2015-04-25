---
layout: post
title:  "A surprise from Spark streaming checkpoint directory"
date:   2015-04-25 14:20:00
categories:  scala spark
---

This is really a _Doh! moment_ since the behavior is obvious in retrospect. 

I created a Spark program that used ```updateStateByKey``` and thus required a checkpoint directory.

After running the program, there were a few errors to be corrected.
Subsequent runs still exhibited the same behavior, leading to some concern that sbt and Eclipse were conflicting, etc.

Of course, the sunsequent runs read the contents of the checkpoint directory.
That includes serialization representation of the impacted classes!

Deleting the checkpoint directory resolves the problem.








