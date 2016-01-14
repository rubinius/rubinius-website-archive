---
layout: post
title: "What, How, Why? - Rubinius Logs, Metrics & Analysis"
author: Brian Shirai
twitter: brixen
---

Knowing how your application is functioning when it is running is critically important. How much memory is it using? What part of the code is running? How much time is it taking? What was the context of an error that occurred?

To answer these questions effectively requires a system specifically built for inspection. Just tossing in `set_trace_func` (or its "object-oriented" incarnation `TracePoint`) is not sufficient.

For Rubinius, we've been building in the system components for inspectability over the past year. These including logging, metrics, and the essential components for always-on analysis tools. These capabilities form the foundation for understanding what your application is doing, how well the application is doing it, and why the application is behaving a particular way.


## An Inspectable System

An inspectable system is one that is _designed_ to provide system information in a usable and efficient manner. Inspectability is not something that can be retrofit to an existing system.

DTrace is an example of an inspectable system. It is comprised of two parts: the DTrace subsystem and the application probes that provide an interface between the application and the DTrace subsystem.

The DTrace subsytem includes a kernel component that executes a very specifically constrained language to gather application runtime data. The application probes are added to provide the DTrace system with specific access to application data.

DTrace separates monitoring the system from the system's functionality, but still provides sufficient flexibility to gather relevant information in a specific context.

Three principles that guided the design of DTrace were: zero cost when not running, stability and security (or it would not be allowed in production), and reasonable cost when running (ie not just turning on a firehose that requires expensive post processing, but rather emiting only the specifically desired information and possibly doing initial data processing at the collection site.)


## Logs

If there were negligible overhead, all we would need are logs.

From logs, we can understand exactly what an application is doing at a partiular point in time. A typical log line gives 1. a timestamp, 2. some categorization of the event, and 3. some description.

From these we can derive _metrics_: periodicity, duration, frequency, and other measures. On these measures we could layer some sort of alerting system.

We could also derive sequence (in what order events occurred), adjacency (which events occurred together), and direct causality (which events state they triggered other events). This gives the ability to analyze the system behavior at a later time.

Unfortunately, the overhead of logging is definitely not negligible for most applications. This requires us to make trade-offs between knowing enough to understand basic application behavior and still getting reasonable application performance.

The first trade-off is one of content versus runtime expense. By eliminating most of the information in the logging of an event, we can replace _many_ log entries with a single value.

For example, to know how many items are in a queue, we could log every event adding an item to the queue, or we could simply count every time an item is added to a queue.

In this way, metrics give us extremely valuable insight into an application's performance, but at the expense of a loss of generality and loss of information (only specific events are counted and we lose the distribution of the events within a reporting interval).

The second trade-off is one of system complexity versus runtime expense. Logging events is probably the simplest (next to doing nothing) way to make a system inspectable. However, when the runtime cost is too great, replacing the analysis capability of logging requires many other, more complex, mechanisms.

We'll look at metrics and analysis in Rubinius in the next two sections.


## Metrics

Rubinius builds in a set of performance counters on various subsystems. You can view the counters currently defined [in the source code](https://github.com/rubinius/rubinius/blob/858ba3c28818fd77ae3780a4dd4dbea8f3b9626e/vm/metrics.cpp#L278-L393).

The Rubinius metrics are monotonic, which makes the data more robust under sampling.

The counters are contained in thread-local data to improve performance by eliminating data contention. A background thread runs asynchronously with the application running on Rubinius, constantly aggregating the counters and possibly emitting their values to either [StatsD](https://github.com/etsy/statsd) or the file system. (Writing an emiter is relatively easy and more emitters are planned.)

Emitting and consuming these metrics is extremely easy.

To emit the metrics, either use a Rubinius command line option, or put the option in the `RBXOPT` environment variable. In this case, I'm turning on the file emitter and setting the reporting interval to 500 milliseconds (see `rbx -Xhelp`):

```
$ rbx -Xsystem.metrics.target=./metrics.dat.1 \
      -Xsystem.metrics.interval=500 \
      some_script.rb
```

I've written [a quick script](https://gist.github.com/brixen/26ee9b8189ea7a65f6ff) that I'll be packaging up as the Rubinius Grapher gem to create a quick terminal ASCII groph from the data.

<div class="rubinius-logs-metrics-analysis">
<script src="https://gist.github.com/brixen/c6bc41ba15bcc3bf238d.js"></script>
</div>

There's a lot of work still needed on the grapher to improve the utility of the graphs, but it's a great tool to quickly visualize the behavior of your application (eg for getting a quick handle on some issue you are debugging).

For a more useful system in production, check out a previous post by Jose Narvaez, [Rubinius Metrics meet InfluxDB part II](http://rubinius.com/2015/01/05/rubinius-metrics-meets-influxdb-part2/) on using InfluxDB and Grafana in a Docker container.

(If anyone would like to write the third edition of this post that updates the Docker instructions and fixes up the Docker image to use the current Rubinius metrics, that would be a much-appreciated contribution.)

So, Rubinius has built-in, easily accessible metrics on system components. We're now focused on building out the analysis capability.


## Analysis

From the above discussion of inspectable systems, we have a clear view of what is required for Rubinius to provide good analysis support: no cost when not use, safe and secure, and reasonable overhead when in use.

We've started to build this system and it will be the topic of many future posts, so I'm only introducing it briefly here.

One of the major aspects of analysis in Rubinius rooted in how we have built our object memory. Typically, when we think of memory, we think of our Ruby object, but not usually of _how our Ruby methods are running_.

In Rubinius, the object graph (ie the Ruby objects and the graph created by these objects referencing each other) _also contains information about the ruby methods_. The _inline cache_ objects that record the type of object at a method call site are just plain Ruby objects. For example, in method `meow` below, the type of `repeat` is Fixnum (because we pass in 5 below), so when `meow` runs, the inline cache for `repeat.times` records that `repeat` is a Fixnum, and also where the `times` method is found.

{% highlight ruby linenos %}
def meow(repeat)
  repeat.times { puts "Meow" }
end

meow 5
{% endhighlight %}

Consequently, analyzing the execution of our example script is possible by analyzing the object graph. There is extremely powerful application analysis possible just by looking at which methods call other methods. If you're interested to learn more about this, take a look at [Elemental Design Patterns](http://www.amazon.com/Elemental-Design-Patterns-Jason-Smith/dp/0321711920/).

We hope this very brief introduction to application analysis in Rubinius will spark your curiosity. There's lots more to come.
