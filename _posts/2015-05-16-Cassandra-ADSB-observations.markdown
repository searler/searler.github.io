---
layout: post
title:  "ADSB broadcast rollup using Cassandra"
date:   2015-05-16 16:32:00
categories:  ADSB  Cassandra
---

There are many articles and presentations that describe how well Cassandra handles time series. 
It would thus seem reasonable that Cassandra could be used to store streams of ADSB mode-S broadcasts.

One candidate use case would use the broadcasts to generate a periodic accounting of the aircraft flight path and thence
determine charges for carbon taxes, ATC user fees, etc. Batch processing sometime after all the data capture is then a perfectly
reasonable design. 

The [canonical timeseries example](http://planetcassandra.org/getting-started-with-time-series-data-modeling/) assumes
the data will be queries by entity and uses a design of the form
```
CREATE TABLE data (
entity_id text,
event_time timestamp,
event_data text,
PRIMARY KEY (entity_id,event_time)
);
```

In our use case, the entity would be an aircraft. There are many millions of aircraft and we have no knowledge as to
which aircraft might be captured by our system.

If we assume the batch processing occurs daily then a different design is more appropriate
```
CREATE TABLE IF NOT EXISTS adsb.modes (
 day int,
 aircraft_id int, 
 source int, 
 event_time timestamp, 
 data blob, 
 PRIMARY KEY ((day),id,event_time);
```

Our batch code can then retrieve the rows for last two days (since no commercial flight has a duration exceeding 24 hours)
and extract all complete flights. 

The relevant query would then be
```
select day, id,source,event_time,data from adsb.scode where day in (?,?)
```

This query works well but returns only 4 million rows. 
This is insufficient since 16 thousand aicraft airborne at any time with a broadcast rate of 5Hz generates 7 billion rows per day.
(These numbers are a little overstated but certainly within the correct order of magnitude)
Explicitly specifying a limit larger than 4 million has no effect. The documentation indicates the query might be truncated by a timeout
but no errors are reported or logged. I was unable to find any relevant configuration parameters that would modify this behavior. 
This restriction would require windows that are only 50 seconds wide, so changing ```day``` to ```minute``` would be more appropriate.

A simple load test:

Item             | Value
-----------------|----------------------------------------------------
Node|AMD 1100T, 12 Gbyte, single SSD
Load 40 million rows asynch API|1200 seconds
Read all rows using 2000 sync calls |352 seconds
Disk space|1.6 Gbyte

The daily processing would then require 58 node-hours to load and 34 node-hours to read, ignoring
conflict between read and write. Total disk space (before replication) would be 560 Gbyte.

Seems 3 -5 nodes would suffice to cover the load and permit data replication.







