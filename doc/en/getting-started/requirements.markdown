---
layout: doc_en
title: Requirements
previous: Getting Started
previous_url: getting-started
next: Building
next_url: getting-started/building
---

Ensure you have the following programs and libraries installed. Also see the
subsections below for special requirements for your particular operating
system.

The following are suggestions for getting more information about the programs
and libraries needed to build Rubinius. Your operating system or package
manager may have other packages available.

  * [GCC and G++ 4.x](http://gcc.gnu.org/)
  * [GNU Bison](http://www.gnu.org/software/bison/)
  * [MRI Ruby 2.0.0+](http://www.ruby-lang.org/) If your system does not have
    Ruby 2.0.0 installed, consider using
    [ruby-install](https://github.com/postmodern/ruby-install) to install it.
  * [Rubygems](http://www.rubygems.org/)
  * [Git](http://git.or.cz/)
  * [ZLib](http://www.zlib.net/)
  * pthread - The pthread library should be installed by your operating system
  * [gmake](http://savannah.gnu.org/projects/make/)
  * [bundler](http://bundler.io/) `[sudo] gem install bundler`
  * [editline](http://thrysoee.dk/editline/) when using newer versions of LLVM


### Apple OS X

The easiest way to get a build environment on Apple OS X is to install the
XCode Tools and Utilities. Once installed, you can enable developer mode crash
reporting at: /Developer/Applications/Utilities/CrashReporterPrefs.app


### Debian/Ubuntu

Just run this to install required packages:

    $ sudo apt-get install -y build-essential bison ruby-dev rake zlib1g-dev \
        libyaml-dev libssl-dev libreadline-dev libncurses5-dev llvm llvm-dev \
        libeditline-dev

This is tested using fresh-installed Ubuntu 12.10. There may be minor differences with older
Ubuntu, Debian or other Debian-based distributions. For example, on Debian sid, you
should also install the `libedit-dev` package.

### Fedora/CentOS

  * ruby-devel
  * readline-devel
  * zlib-devel
  * openssl-devel
  * libedit-devel

### FreeBSD

Rubinius has a port in FreeBSD ports tree. It's called `lang/rubinius`. You
can find information about this port on [FreshPorts](http://www.freshports.org/lang/rubinius/). Once being
installed the port installs all the dependencies automagically.
