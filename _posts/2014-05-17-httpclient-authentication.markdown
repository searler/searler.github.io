---
layout: post
title:  "Surprising apache httpclient authentication behavior"
date:   2014-05-17 20:20:00
categories:  java httpclient 
---

[httpclient](http://hc.apache.org/httpcomponents-client-ga/index.html) is a widely used Java library
for accessing web servers. The complexity of the API indicates the http protocol is not quite as
simple as might be generally thought and there are certainly some edge cases. 

My use case involved screen scraping the web UI from an embedded device, using https and basic authentication.
The implementation is a little unusual in that the client _owns_ the password and the web server in the device
is configured to match the client. (i.e. the reverse of the normal case). This works well until the password
in the device is changed via its front panel to the incorrect password. The httpclient then never again authenticates
against the device, even once the password is restored to the correct value!

The HttpContext state contains the authentication credentials, a cache and other state information. 
The failure to authenticate evidently changes that state so as to prohibit further authentication. I could
only recover by creating a new instance, freshly loaded with the unchanged credentials. 

As far as I can tell, the documentation does not describe this surprising behavior. 








