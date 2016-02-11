---
layout: post
title:  "Testing maven plugins"
categories: sonatype release
---

JTransc has a maven plugin that hooks the package maven phase and transcompiles
all the code into the target app. Since you can call maven from console I just
wanted to debug easily on intelliJ. It ended being pretty easy.

Instead of calling `mvn` command, there is a `mvnDebug` command:

{% highlight xml %}
mvnDebug package
{% endhighlight %}

It will be waiting, listening to port `8000` by default.

In intelliJ you must create a Remote run configuration, and just have to change the port to `8000`.
While mvnDebug is waiting you can execute the run configuration.
And breakpoints should just work, if you have the sources of the plugin.
