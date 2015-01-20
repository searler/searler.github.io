---
layout: post
title:  "Spark SQL against streaming data in memory "
date:   2015-01-19 20:50:00
categories:  scala spark streaming sql
---

Slides 67 to 73 from this [presentation](http://www.slideshare.net/deanwampler/why-scala-is-taking-over-the-big-data-world)
provide a simple Spark streaming application that uses Spark SQL to process the data.

A little effort is required to make this into a functional program for Spark 1.2

{% highlight scala %}
import org.apache.spark.streaming._
import org.apache.spark.sql._
import org.apache.spark._
import org.apache.spark.storage.StorageLevel

object Experiment extends App {
  val conf = new SparkConf()
    .setMaster("local[*]")
    .setAppName("Example")
    .set("spark.executor.memory", "1g")
    .set("spark.cleaner.ttl", "30")
    .set("spark.streaming.receiver.maxRate", "400000")

  val sparkContext = new SparkContext(conf)
  val streamingContext = new StreamingContext(sparkContext, Seconds(10))
  val sqlContext = new SQLContext(sparkContext)
  import sqlContext._

  case class Flight(
    number: Int,
    carrier: String,
    origin: String,
    destination: String)

  object FlightParser {
    def parse(str: String): List[Flight] =
      str.split(" ").grouped(4).
        collect { case Array(n, c, o, d) => Flight(n.toInt, c, o, d) }.toList
  }

  val server = "localhost"
  var port = 12345

  val dStream = streamingContext.socketTextStream(server,
    port, StorageLevel.MEMORY_ONLY)

  val flights = for {
    line <- dStream
    flight <- FlightParser.parse(line)
  } yield flight

  flights.foreachRDD { (rdd, time) =>
    rdd.registerTempTable("flights")
    sql(s"""
   SELECT $time, carrier, origin, destination, COUNT(*) AS c
   FROM flights
   GROUP BY carrier,origin,destination
   ORDER BY c DESC
   LIMIT 20""").foreach(println)
  }

  streamingContext.start()
  streamingContext.awaitTermination()
  streamingContext.stop()
}
{% endhighlight %}

Note that application will not run in Eclipse and must be forked when run from SBT, due to
this [issue](https://issues.apache.org/jira/browse/SPARK-5281).

The code was tested by running ```yes 12 a b c 13 x y z |nc -kl 12345``` in another console.

This code added the following parameters so as avoid disk I/O and overflow of both
disk and memory space.

1. StorageLevel.MEMORY_ONLY
2. set("spark.executor.memory", "1g")
3. set("spark.cleaner.ttl", "30")
4. set("spark.streaming.receiver.maxRate", "400000")

The maxRate parameter is unfortunately needed since Spark does not implement the necessary flow control.
That value was empirically derived so as to achieve ~97% CPU load across all 6 cores of an AMD Phenom II X6 1100T.

The web UI indicates a median rate of 400K records/second but the output indicates 800K record/second.
In other words, the rate is number of lines read from the socket, rather than the number of object written
into the DStream.

```
[info] [1421724610000,x,y,z,4000125]
[info] [1421724610000,a,b,c,4000125]
```







