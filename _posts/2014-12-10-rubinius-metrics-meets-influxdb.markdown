---
layout: post
title: Rubinius Metrics meet InfluxDB
author: Jose Narvaez
---


##A little bit of background

Along with the release of 2.3.0 an exciting feature landed: the basic infrastructure for always-on metrics of Rubinius subsystems. And after reading the [2.3.0 release notes](https://github.com/rubinius/rubinius/releases/tag/v2.3.0) I just fired my IRB session and tried:

```ruby
irb(main):001:0> Rubinius::Metrics.data
=> #<Rubinius::Metrics::Data:0x8c94>
```

Sweet! Immediately went back to the release notes and again, tried to find how could I access the metrics inside that object:

```ruby
irb(main):001:0> Rubinius::Metrics.data.to_hash
=> {:"gc.young.total.ms"=>0, :"gc.immix.stop.total.ms"=>0, 
:"gc.large.sweep.total.ms"=>0, :"gc.young.last.ms"=>0, 
:"gc.immix.concurrent.total.ms"=>0, :"gc.immix.stop.last.ms"=>0, 
:"jit.methods.queued"=>0, :"gc.immix.concurrent.last.ms"=>0, 
:"jit.methods.compiled"=>0, :"memory.young.objects.total"=>0,
:"jit.time.last.us"=>0, :"memory.young.bytes.total"=>0, 
:"jit.time.total.us"=>0, :"memory.promoted.objects.total"=>0, 
:"memory.young.objects.current"=>0, ...}
```

And voila! JIT, GC, IO Subsystem, Memory, OS Signals (and more!) stats about the current Rubinius VM running my IRB at my fingertips. I was delighted to see there was so much data accessible in near-real-time through a simple **Ruby** object. A quick switch back to the release notes revealed something even more promising: built-in support for emitting the metrics to [StatsD](https://github.com/etsy/statsd) (a simple daemon for stats aggregation). Are you thinking the same as me? Lets do some graphing!

## Looking for a tool for graphing

Ironically, this step was the most difficult. After researching open source tools for graphing everything seemed to point in the same direction: 
[Graphite](http://graphite.wikidot.com/). The plan was to have Rubinius connected to StatsD, then connect StatsD with Graphite to make some graphs. All of this seemed straightforward except that getting Graphite up and running was in practice, a pain, and after even trying the [Docker Image](https://registry.hub.docker.com/u/dbiesecke/docker-graphite-statsd) I could not get it up and running. I'm lazy so I gave up on Graphite and started to look for other approaches.

Then, I remembered Brian talking about a time series database [InfluxDB](http://influxdb.com/) on the Rubinius IRC channel. I went to the site and totally fell in love with it, installation was easy, it was working out of the box and seemed that I was in the middle of a lucky strike because while reading InfluxDB documentation I saw a [Grafana Dashboards](http://grafana.org/) link, followed the rabbit hole and there it was: "An open source, feature rich metrics dashboard and graph editor for Graphite, InfluxDB & OpenTSDB." That sounded like a plan: Rubinius -> StatsD -> InfluxDB -> Graphana. The "StatsD -> InfluxDb" part seemed a little bit scary but after some googling I've had found this handy [StatsD backend](https://github.com/bernd/statsd-influxdb-backend). I had everything I could wish, lets do some X-RAYS to Rubinius!

## InfluxDB

### Getting InfluxDB

I'm using Mac OS X so for me was matter of:

```sh
$ brew install influxdb
```

But you can check the [installation](http://influxdb.com/docs/v0.8/introduction/installation.html) section in the InfluxDB documentation for your operating system. The version used at the time of the writing is v0.8.7.

### Starting InfluxDB

Again it can vary depending on the operating system but for Mac OS X you can start it like this:

```sh
$ ln -sfv /usr/local/opt/influxdb/*.plist ~/Library/LaunchAgents
$ launchctl load ~/Library/LaunchAgents/homebrew.mxcl.influxdb.plist
```

### Creating the InfluxDB databases

As I mentioned before, all the metrics will be sent to InfluxDB by StatsD to one database (to keep things relatively simple) in our case the "rbx". One of the neat things of InfluxDB is it's admin UI listening at http://localhost:8083. You can read more about how to login into InfluxDB [here](http://influxdb.com/docs/v0.8/introduction/getting_started.html) but the default credentials are user: "root" and password: "root" and should be enough should go there and create a two new databases one named "rbx" and other named "grafana" as I explain later is used by Grafana to store the dashboards.

## StatsD

### Getting StatsD

StatsD is a NodeJS package so it can be installed in your system with [NPM](https://www.npmjs.com/):

If you don't have NPM installed on your system and you are on OS X you can install NPM (which will install NodeJS which comes with NPM bundled) with this:

```sh
$ brew install npm
```

If you are using another operating system you may find [this](http://blog.npmjs.org/post/85484771375/how-to-install-npm) helpful.

After you have NPM you can:

```sh
$ npm -g install statsd
```

### Getting the InfluxDB StatsD backend:

```sh
$ npm install -g statsd-influxdb-backend
```

### Creating a configuration file for StatsD

We need a bit of configuration tweaking, but you can use this one:

```sh
$ wget -O config.js http://git.io/qiVTGw
```

This config file has the InfluxDB settings such as enabling the InfluxDB backend, credentials (just use the default ones) and the InfluxDB database to which you will be forwarding the metrics ("rbx"). I've also enabled the debugging option for StatsD because, you know, is extremely difficult for a developer not knowing what's happening ;).

### Starting StatsD and testing it 

You can now:

```sh
$ statsd config.js 
```

You should see something like:

```sh
$ 9 Dec 13:11:37 - reading config file: config.js
$ 9 Dec 13:11:37 - DEBUG: Loading server: ./servers/udp
$ 9 Dec 13:11:37 - server is up
$ 9 Dec 13:11:37 - DEBUG: Loading backend: ./backends/graphite
$ 9 Dec 13:11:37 - DEBUG: Loading backend: statsd-influxdb-backend
```

Also, you can run a small test to StatsD like this:

```sh
$ echo "foo:1|c" | nc -u -w0 127.0.0.1 8125
````

After that you can just go to InfluxDB UI follow the "Explore Data" link at the right of the "rbx" database and run the "list series" query. Hopefully, if everything is OK you should be able to see you "foo.counter" there.

## Grafana

### Getting Grafana

To get Grafana just go to http://grafana.org/ download the compressed file, place it in the directory of your preference and you can either put the files into the public static files directory of your web server or as I my case just open index.html with your web browser.

For the lazy ones:

```sh
$ wget http://grafanarel.s3.amazonaws.com/grafana-1.9.0.zip
$ unzip grafana-1.9.0.zip
```

### Configuring Grafana

Grafana has the concept of Datasources from where it pulls the data. It ships with a sample config file we could tweak but as we are lazy [here](https://gist.github.com/goyox86/b83e1c08586b06933656) is the one I'm using in my machine. There is nothing esoteric in there we just define two sources: one "rbx" pointing to our "rbx" InfluxDB database (where StatsD is dropping the metrics (remember? :)) and other called "grafana" used internally by Grafana to persist the dashboards.

So the process would be:

```sh
$ cd grafana-1.9.0
$ wget -O config.js http://git.io/QZRR1w
$ open index.html
```

### Creating some dashboards

You are almost done! Now you have to go to the Grafana documentation and learn how to make some graphing! (I'm kidding, There is a sample dashboard for seeing Rubinius metrics your only work is to import [this dashboard](https://gist.github.com/goyox86/ed798af0f2d82cb96a20)
in Grafana (by clicking the small folder 'Open' button in the top right section of the UI, then clicking in 'Import') and start looking at those amazing graphs.

Something like this:

```sh
$ wget -O RBX-DASH http://git.io/Ol2m4A
```

## Running Rubinius with StatsD support

There will be nothing to draw if you don't start the StatsD metrics emitting in your Rubinius process. You can do this by passing some -X flags to rbx command.

For example (and the most simple would be to fire up an IRB sesion):

```sh
$ RBXOPT="-Xsystem.metrics.target=statsd -Xsystem.metrics.statsd.server=0.0.0.0:8125" rbx
```

Or if you are intrepid just clone the (Rubinius Benchmarks) repo and run some benchmarks and see the X-RAYS in action \o/:

```sh
$ git clone git@github.com:rubinius/rubinius-benchmark.git
$ cd rubinius-benchmark
$ RBXOPT="-Xsystem.metrics.target=statsd -Xsystem.metrics.statsd.server=0.0.0.0:8125" ./bin/benchmark -t /Users/goyox86/.rubies/rbx/bin/rbx core
```

Or even if you are more intrepid start any of your rails apps like this:

```rbx
$ RBXOPT="-Xsystem.metrics.target=statsd -Xsystem.metrics.statsd.server=0.0.0.0:8125" rbx -S bundle exec rails server
```

## Conclusion

There is a bright future coming for all logging, monitoring and analysis for Rubinius, the fact that we now support StatsD directly opens a whole world of possibilities since we can send metrics to anything supported by StatsD, allowing you to X-RAY your Rubinius processes. Imagine sending your Rubinius metrics to NewRelic, Skylight out of the box? Please don't hesitate in sharing your ideas about this even requesting built-in support for other metrics or other targets. Happy Graphing! 

