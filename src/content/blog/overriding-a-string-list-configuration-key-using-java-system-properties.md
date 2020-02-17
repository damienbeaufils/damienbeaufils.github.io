---
title: "Overriding a String List configuration key using Java System properties"
date: "2016-02-02"
draft: false
tags: [Play, sbt, configuration, bash]
featured: false
---

With Play Framework, you can [override configuration keys by specifying them as Java System properties](https://www.playframework.com/documentation/2.4.x/ProductionConfiguration#Overriding-configuration-with-system-properties). Example:

```
activator run -Dsome.key="some.value"
```

But if you have a String List key, you cannot override it like in the configuration file. Example using the trusted proxies configuration:

```
activator run -Dplay.http.forwarded.trustedProxies=\["127.0.0.0/8","10.0.0.0/8"\]
```

You will have this runtime error:

```
Configuration error\[system properties: play.http.forwarded.trustedProxies has type STRING rather than LIST\]
```

To avoid this, you have to specify each entry of the list using the entry index:

```
activator run -Dplay.http.forwarded.trustedProxies.0=127.0.0.0/8 -Dplay.http.forwarded.trustedProxies.1=10.0.0.0/8
```

Note that there are no double quotes around the values. I encountered some issues when using quotes, and it works fine without them.
