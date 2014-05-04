---
layout: post
title:  "Data Communication Protocol implementation using Scala - Background"
date:   2014-05-04 15:00:00
categories:  scala 
---

$JOB requires support of [MIL-STD-188-171](http://en.wikipedia.org/wiki/MIL-STD-188).
My Ada experience gave me the task of porting the canonical 1982 Ada implementation to Linux. 
That port was surprisingly straightforward, consisting largely of removing the compiler pragmas that
allocated memory in quantities of 1 kilobyte.

The code is quite large (~10k sloc) and complex.
The [specification](http://www.everyspec.com/MIL-STD/MIL-STD-0100-0299/MIL-STD-188_171_4538/) does not
use state machines but rather a huge flow chart and perhaps a dozen state variables. The lack of clarity is
not improved when one of the flow chart off-page connectors references a non existent page!
The Ada code is essentially a 1-1 transcription of the flow chart.

I decided to create a state machine design, using a functional approach in Scala.
The primary goal was a better understanding of the protocol, with a secondary goal of improving
my FP knowledge. 

The end result was quite successful:

*  The code was fully validated against the Ada implementation, flushing out a 32 year old defect in that code!
*  1/10th the code size
*  10x peak performance

The performance was very gratifying since the code was written for clarity and simplicity, not performance.
A single received character could allocate up to 3 objects on the heap, resulting in fairly high GC pressure.
The Ada code performed no memory allocation at all and was originally targeted to a [Intel 286](http://en.wikipedia.org/wiki/Intel_80286) processor.





