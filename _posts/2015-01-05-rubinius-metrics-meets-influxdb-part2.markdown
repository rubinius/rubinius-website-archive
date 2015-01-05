---
layout: post
title: Rubinius Metrics meet InfluxDB part II
author: Jose Narvaez
twitter: goyox86
---

We live in a containerized world now and after seeing the complexity of setting all up in [the first part](http://rubini.us/2014/12/10/rubinius-metrics-meets-influxdb/) Mr [Joe Eli McIlvain](https://github.com/jemc) kindly created [this](https://github.com/rubinius/influxdb-grafana) this awesome [Docker](https://www.docker.com/) image with everything ready for seeing some neat graphs about the Rubinius VM.

## Setup

If you are on Mac OS X like me you can install docker with the [Homebrew](http://brew.sh/) package manager. It's worth to mention that Docker setup in OS X these days is a bit trickier because it depends on Linux kernel features. In order to run it in non-Linux operating systems you have to install [boot2docker](http://boot2docker.io/) (which in essence is a Linux based [VirtualBox](https://www.virtualbox.org/) virtual machine in which Docker will run). Having said that, here is an outline of the whole process:

1. Install VirtualBox.
2. Install boot2docker and Docker.
3. Setup boot2docker and Docker.
4. Setup and start the Rubinius Influxdb-Grafana Docker container.
5. Run Rubinius enabling the StatsD metrics.


## Installing VirtualBox

You can use the [Homebrew-cask](https://github.com/caskroom/homebrew-cask) brew extension to install applications distributed as binaries. You can install it using brew itself:

```sh
$ brew install caskroom/cask/brew-cask
```

And after that use Homebrew-cask to install VirtualBox:

```sh
$ brew cask install virtualbox
```

Alternatively you can download VirtualBox for your operating system from [here](https://www.virtualbox.org/wiki/Downloads) or install it using your package manager.

## Installing boot2docker and Docker

Here is the command for installing both Docker and boot2docker:

```sh
$ brew install boot2docker
```

If you are wondering why there isn't an explicit section in how to install Docker is because it is a dependency of boot2docker in Homebrew. However, you can check Docker's [official installation guide](https://docs.docker.com/installation/) for detailed information about how to install Docker in your OS or try installing it with your package manager.

## Setting up boot2docker and Docker

On a Linux based operating system you have to start the docker daemon and of course, you can skip this step.

On Mac OS X the first time you have to initialize the boot2docker virtual machine:

```sh
$ boot2docker init
```

The first time the ISO image will be fetched, so probably you will have to wait a bit. If everything goes well you are now able to start the boot2docker VM:

```sh
$ boot2docker start
```

You should see something like this:

```sh
Waiting for VM and Docker daemon to start...
........................ooooooooooooooooooooo
Started.
Writing /Users/goyox86/.boot2docker/certs/boot2docker-vm/ca.pem
Writing /Users/goyox86/.boot2docker/certs/boot2docker-vm/cert.pem
Writing /Users/goyox86/.boot2docker/certs/boot2docker-vm/key.pem

To connect the Docker client to the Docker daemon, please set:
    export DOCKER_HOST=tcp://192.168.59.103:2376
    export DOCKER_CERT_PATH=/Users/goyox86/.boot2docker/certs/boot2docker-vm
    export DOCKER_TLS_VERIFY=1
```

The last three lines are important because the Docker client uses it to connect to the Docker daemon. You can either copy and paste these lines in your current terminal session or put them in your shell's init file.

## Setting up the rubinius/influxdb-grafana Docker container.

When everything is in place you can go to the [homepage](https://github.com/rubinius/influxdb-grafana) of the Rubinius InfluxDB-Grafana container read a bit the instructions to start it or just be lazy like me and run:

If you are on a Linux based OS:

```sh
 docker run -d \
  -p 8125:8125/udp \
  -p 8086:8086 \
  -p 80:80 \
  rubinius/influxdb-grafana
```

Or a Mac OS X one:

```sh
docker run -d \
  -p 8125:8125/udp \
  -p 8086:8086 \
  -p 80:80 \
  -e INFLUXDB_SERVER=http://192.168.59.103:8086 \
  rubinius/influxdb-grafana
```

Notes for Mac OS X users:

Why an environment variable on Mac OS X?

From rubinius/influxdb-grafana container [site](https://github.com/rubinius/influxdb-grafana):

"Because docker relies on features of the Linux kernel, it does not run containers natively in Mac OS X - it hosts containers inside of a Linux VM called boot2docker. One consequence of this is that ports mapped to the docker host from containers are not mapped to localhost of OS X, but to the boot2docker host. Therefore, in all of the above commands, OS X users should replace localhost with the IP address given by running boot2docker ip."

We have to be able to know the boot2doecker virtual machine ip address in order to allow Grafana (a client side application) to hit the dockerized InfluxDB server.

How to get that boot2docker ip?

```sh
$ boot2docker ip
```

## Run Rubinius enabling the StatsD metrics

Enable Rubinius StatsD metrics emitting for your Rubinius process like this:

On Linux based systems:

```sh
RBXOPT="-Xsystem.metrics.target=statsd \
        -Xsystem.metrics.statsd.server=localhost:8125" \
  rbx # (your app here)
```

On Mac OS X:

```sh
RBXOPT="-Xsystem.metrics.target=statsd \
        -Xsystem.metrics.statsd.server=192.168.59.103:8125" \
  rbx # (your app here)
```

Where "192.168.59.103" is the boot2docker VM ip address.

Some screenshots while running [Rubinius Benchmarks](https://github.com/rubinius/rubinius-benchmark):

![RBX Dashboard.png](/images/rubinius-dash-screenshot.png)

![RBX Dashboard 2.png](/images/rubinius-dash-screenshot2.png)

![RBX Dashboard 3.png](/images/rubinius-dash-screenshot3.png)

Happy Graphing!
