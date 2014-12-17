---
layout: post
title:  "IEE Software and SOA misconceptions"
date:   2014-12-17 22:10:00
categories:  SOA 
---
The Sept/October 2014 IEE Software has an article _Service-Oriented Architecture and Legacy Systems_,
which contains a number of surprising conclusions.

**Ultimately, the message in HTTP is plaintext, which any system or programming language can produce.**

Appears on page 16, describing how a legacy system can be wrapped in a SOA service.
This is an essentially correct statement, but neither helpful nor realistic. 
Given that statement, one might expect to find that all web services are written in [SNOBOL](http://en.wikipedia.org/wiki/SNOBOL) or raw perl.
It also neglects the several feet of paper defining all the standards to which the code must comply:XML, XML Schema, Unicode, SOAP, WSDL, HTTP and TCP.











