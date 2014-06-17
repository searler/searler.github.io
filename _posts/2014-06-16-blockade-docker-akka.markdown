---
layout: post
title:  "Exercising Akka via blockade and docker"
date:   2014-06-16 21:00:00
categories: akka docker blockade
---

This [project](https://github.com/mhamrah/akka-docker-cluster-example) provides a useful example of how to
deploy an [Akka cluster](http://akka.io/) into [docker](http://www.docker.com/).

Akka provides sophisticated [testing tools](http://doc.akka.io/docs/akka/2.3.3/dev/multi-node-testing.html#multi-node-testing). It could be
argued that the test environment is not the same as a real deployment, and hence do not necessarily reflect the true behavior.  This would
be particularly true for code that manages network failures.

[blockade](http://blockade.readthedocs.org/en/latest/) provides a simple mechanism to run several docker instances and control their 
network connectivity. The above project can be run under blockade, using the following `blockade.yml` file

```yaml
containers:
  c1:
    image: clustering
    ports: [10000]

  c2:
    image: clustering
    links: {"c1":"seed"}

  c3:
    image: clustering
    links: {"c1":"seed"}
```    


A noisy network connection to c2 could then be simulated by `blockade flaky c2`, or a partition by `blockade partition c1,c2 c3`.


