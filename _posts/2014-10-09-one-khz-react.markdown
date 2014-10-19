---
layout: post
title:  "React.js at one kilohertz"
date:   2014-10-09 18:20:00
categories:  react
---
This earlier [post]({% post_url  2014-06-29-react-component-lookup %}) describes
an elegant React based mimic display.

The data source is "real time", which raises questions of latency and performance. 
This can be illustrated with a simplistic benchmark of 40 entities, each with 3 values. 

All is well with 1Hz update rate, with each update completing well before the next tick.
At higher rates, the update no longer completes in time and the display starts to lag.
One solution is to only apply the new state at a fixed rate, rather than with every new
datum.

The following code updates the state with each SSE message, but only hands off the 
state to React every second.
{% highlight javascript %}
componentWillMount: function(){
   var stream = new EventSource("/stream");
   stream.addEventListener("message",this.process,false);
   setInterval(this.refresh, 1000);
},
refresh: function(){
   this.setState({nodes: this.state.nodes});
},
{% endhighlight %}

There is no visible degradation in the currency of the display.
It might be [appropriate](http://www.nngroup.com/articles/response-times-3-important-limits/) to increase the rate to 10Hz,
which would require some optimization of the redraw.

The test display remains functional even when the server delivers updates at 1 khz, approximately 10 MByte/second of data.








