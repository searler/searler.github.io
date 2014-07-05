---
layout: post
title:  "React vs Swing"
date:   2014-07-5 15:00:00
categories: react.js
---
This [code]({% post_url  2014-06-29-react-component-lookup %}) is part of an experiment to replace Swing user interfaces.

$JOB exclusively uses Swing for user interfaces.
The primary justifications are "real time" updates and interaction complexity.

The latest project displays a schematic of hardware state(up/down, CPU temperature, etc.) and a few operations(shutdown/reboot).
The user interface is almost 7000 lines of commented Java code, which seemed a little large. 

The experiment implements perhaps 40% of the functionality:

* 227 uncommented lines of Javascript
* 129 uncommented lines of [Less](http://lesscss.org/) 
* Same performance (before optimizing the React change detection)

Getting the layout correct via CSS was the hardest part (which is one reason for the size of the Java implementation).

