---
layout: post
title:  "Enriching POJO JSON"
date:   2014-06-29 15:00:00
categories: java jackson json
---

[Jackson](https://github.com/FasterXML/jackson) provides a simple binding mechanism to create json from a Java POJO.
It also provides annotations to control the generated json, but these are not always sufficient or appropriate.

Consider a use case where the POJO does not contain all the data required by the consumer of the json;
and the POJO cannot be changed. 

A wrapper POJO could be created around the original POJO, but that is clumsy and adds additional structure to the json.
A simpler approach would generate a json tree from the POJO and then manipulate the tree.

For example, add the classname and some location metadata:

```
ObjectNode tree = mapper.valueToTree(data);
tree.put("type", data.getClass().getCanonicalName());
tree.put("location", locationLookup.getLocation(data));
String result = tree.toString();
```



