---
layout: post
title:  "Setting up docker 1.0 for blockade"
date:   2014-06-15 15:00:00
categories: docker blockade
---

[blockade](http://blockade.readthedocs.org/en/latest/) currently [requires](http://blockade.readthedocs.org/en/latest/install.html) the LXC driver.

Achieving the required configuration on Fedora 20 was surprisingly difficult.

Adding DOCKER_OPTS to the configuration file /etc/sysconfig/docker had no effect.

It is necessary to directly add `-e lxc` to /usr/lib/systemd/system/docker.service.



