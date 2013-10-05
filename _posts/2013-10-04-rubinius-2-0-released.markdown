---
layout: post
title: Rubinius 2.0 Released
author: Brian Shirai
---

We are thrilled to announce the release of Rubinius 2.0. There are many
exciting things to share. We'll review what Rubinius is, look at what you can
expect from this release and future releases, and talk about plans for the
future.

## Many Thanks!

Rubinius 2.0 would not be possible without the tremendous support and
contributions from so many amazing people.

Thanks Matz for creating Ruby. Thanks David Heinemeier Hansson for creating
Ruby on Rails, which spread Ruby happiness around the world. Thanks Evan
Phoenix for creating Rubinius and inspiring us to think beyond current
limitations.

Thanks to the hundreds of contributors for their time, efforts, and frustration
while improving Rubinius and RubySpec. There isn't room to list them all, but
their indelible mark on Rubinius lives on in the source code.

Thanks to the Ruby developers, students, and businesses that all contribute to
making Ruby better. Thanks to Engine Yard for financially supporting Rubinius
development and sponsoring all those rad tshirts and stickers people loved so
much.

All of these people and organizations, not just Ruby developers, are _the Ruby
community_. Rubinius owes you a huge debt of gratitude.

## The 2.0 Story

With the 2.0 release, Rubinius regains a laser focus on supporting the future
of Ruby. Rubinius 2.0 is expected to be compatible with Ruby 2.1.

While MRI hasn't released 2.1 yet, Rubinius will continue improving
compatibility as more features are finalized. Last RubyConf, Matz urged people
to upgrade as soon as possible to Ruby 2.0. Significant effort has been
dedicated to making the upgrade from 1.9 as simple as possible. Rubinius
supports the effort to move Ruby into the future.

Rubinius started life with the goal of bringing modern technology to Ruby's
implementation, giving developers more power, and businesses who rely on Ruby a
faster, more stable and more efficient platform on which to build products and
services.

Over time, we've tried to support multiple Ruby language versions, many
different projects, old and new code, code that abuses every corner of MRI's
*ad hoc* semantics, and every random, undocumented MRI C function with the
Rubinius C-API compatibility layer. Unfortunately, this is unsustainable and
not in the best interests of Ruby or Rubinius.

Starting with 2.0, Rubinius will concentrate on providing exceptional Ruby
support for building modern concurrent and distributed applications. Not every
legacy Ruby program or quirky Ruby feature will be suitable for Rubinius.
Instead, we'll prioritize the performance and stability of concurrent
applications to make Ruby competitive with Go, Erlang, Clojure, Scala, and
Node.

## Versions and Releases

Starting with Rubinius 2.0, we're changing the way releases are done.

Every week or so, we'll release a version of Rubinius. We are not following a
pre-determined release schedule. We will continue to keep the master branch
extremely stable, as we have done for years. If there are only bug fixes on
master since the last *X.Y.Z* release, the new version will be *X.Y.Z+1*. If
there are other changes, the new version will be *X.Y+1.0*.

We're moving to this release process to get updates into your hands as quickly
as possible. No matter how much work we do, there is always more. A release
never seems _ready_. Releases are painful to get right. So we are following the
advice, "If something is painful, push it to the front and work to reduce the
pain." There will be bugs with our Ruby 2.1 compatibility. If something is
broken for you, please file an issue. Hopefully it will be fixed and released
within days.

The goal is to [semantically version](http://semver.org) the Rubinius core
starting with version 3.0. During the 2.x to 3.0 transition, we'll be very
careful about introducing breaking changes, but we'll do so when the benefits
outweigh the risks. Obviously, the more we know about how Rubinius is being
used, the better we can evaluate these decisions.

We've added a subdomain [http://releases.rubini.us](http://releases.rubini.us)
for hosting release tarballs. We expect that over the next few days the many
Ruby installers and switchers will be updated to install Rubinius.

Below, in the section on future plans, I'll explain what we hope to accomplish
with these ambitious release plans.

## Rubinius Parts

Many people have heard that Rubinius is an implementation of the Ruby
programming language. There's a lot wrapped up in that simple description.
Let's review the major pieces of technology in Rubinius today. Later, we will
look at plans to improve these in the future.

The Rubinius architecture is fairly standard for a modern language runtime.

The bytecode virtual machine (VM) runs the bytecode produced by the Ruby
compiler. A notable feature is that every Ruby method essentially gets its own
interpreter. This enables powerful features like the full-speed built-in
debugger. Only the specific Ruby method with debugger breakpoints runs the
"debug" interpreter while the rest of the methods run at normal speed.

The generational garbage collector (GC) has a very fast young generation
collector, usually pausing for less than 15 ms to complete a collection.
Applications running on Rubinius typically see shorter GC pauses times and many
fewer noticeable GC pauses because the entire heap needs to be collected far
less often. Rubinius also has a partially concurrent mark phase for the mature
generation which further reduces the GC pause times when a full collection is
required.

Rubinius implements native operating system threads for concurrency and has no
global interpreter lock (GIL). Ruby code can run in parallel on multi-core or
multi-CPU hardware.

The Rubinius just-in-time compiler (JIT) turns Ruby bytecode into machine code.
The JIT thread is mostly independent of the Ruby threads so the JIT operation
doesn't impact the running code's performance. The JIT framework tracks which
methods are often used and what types of objects are seen. Using this runtime
data, the JIT is able to combine application methods and core library methods,
generating highly optimized machine code that runs several times faster than
the bytecode interpreter.

The Rubinius core libraries (e.g. Array, Hash, Range, etc.), as well as
Rubinius tools like the bytecode compiler, are written in Ruby. The Rubinius
systems treat them just like Ruby application code (e.g. the JIT combining core
library methods and application methods to best optimize running code). This
consistency also improves understanding of the entire Rubinius system. Ruby
developers can contribute to significant parts of Rubinius by simply by writing
Ruby code.

## Plans, Meet Future

For Rubinius to have a place in future application development, it must serve
the needs of those applications.

The world is rapidly changing and the rate of change is accelerating. Building
software today is different than it was just five years ago. Continuous
delivery and A/B testing are becoming commonplace. Businesses must experiment
to discover how to compete in changing conditions. Driving down the cost of
experimenting is essential.

More now than ever, time is money. The time scale to deliver features must be
hours or days, not weeks or months. To meet the required velocity, building
concurrent and distributed applications in a heterogenous environment is no
longer optional, it is essential.

Ruby is more suited than many languages to rapidly deliver features, reducing
the cost to experiment and the time needed to begin engaging customers.
Unfortunately, Ruby development has not kept pace with the software as a
service revolution.

Ruby became popular because Ruby on Rails accelerated the delivery of value by
an order of magnitude. This influence is rapidly declining. Efficiencies that
Rails introduced, things like _convention over configuration_ and full-stack
integration, also encouraged monolithic application architectures. Applications
built this way are difficult to change and difficult to scale, which means that
under changing conditions, their costs tend to quickly outweigh any value they
deliver. Businesses are rapidly learning this lesson.

## Future, Meet Plans

Roadmaps are notoriously painful because, oddly enough, predicting the future
continues to be an inexact science. Given the future that Rubinius wants to
support&mdash;concurrent and distributed applications&mdash;the following are
specific areas we plan to improve in the coming weeks.

1. Rubinius has no global interpreter lock, but we can significantly improve
concurrency coordination in the system. During some phases of garbage
collection, some operations of the JIT, and during fork/exec, we have to stop
all the threads. We intend to improve this significantly so that less
coordination is required.
1. In place of the GIL, Rubinius uses finer-grained locks internally in various
places. These can be reduced further, improving multi-core efficiency, by using
modern lock-free concurrent data structures.
1. While the garbage collector has reasonably low pauses, we can improve this
by making the GC more concurrent and parallel.
1. The JIT compiler already often improves Ruby code performance by 2-4x over
executing bytecode. We can do even better. We will improve communicating Ruby
semantics to the JIT to avoid unnecessarily allocating objects and doing
unnecessary bookkeeping that slows performance. We will expose more of the JIT
framework to Ruby to enable rapidly coding and testing new ideas for optimizing
Ruby.

## Gems as Components

The ability to compose independent components is one of the best means to
manage high complexity. In Ruby, the natural way to package, distribute, and
compose components is gems.

Rubinius wants to bring the advantages of continuous delivery and the
"evergreen browser" idea to Ruby developers. The Rubinius approach to this is
to fully leverage gems.

Rubinius itself has been dramatically simplified. Major components, like the
bytecode compiler, Ruby parser, debugger, etc. have been moved to gems. These
components can be updated easily and quickly without requiring a Rubinius
release. These components participate in the Ruby ecosystem, for example
[Bundler](http://bundler.io) or [dep](https://github.com/cyx/dep), like all the
gems that Ruby developers are familiar with.

In Rubinius 2.0, the [Ruby standard library](http://rubysl.github.io) has also
been converted to gems. The Ruby standard library is some of the oldest Ruby
code that exists. Due to being bundled with MRI all these years, it was far
more difficult to change than a library or gem. Every change required MRI to
accept it, and any changes required waiting for a new release of MRI, something
which did not happen frequently.

It's not surprising that many people simply went around these obstacles and
made alternate libraries. Instead of being the pinnacle of excellent Ruby
design and idioms, parts of the standard library bit-rotted and retained
anachronisms like embedded tests at the end of files. Further, many of the
critical libraries were written as C extensions, making them unusable by JRuby,
Topaz, IronRuby, Opal, and early MagLev (which now has some C-API support).

In Rubinius 2.0, the components and standard library are _just gems_. There is
nothing special about them. They are installed as gems. They participate in gem
sets and Bundler workflow as gems. There are some challenges required to
bootstrap the gems, but that requires an internal Rubinius command, not changes
to the RubyGems infrastructure.

Providing the standard library as gems opens the opportunity to rebuild it _in
Ruby_ and improve the code rapidly so that multiple implementations can share
it. However, we are also not bound to using any of these libraries. Since they
are just gems, they can be put in a Gemfile, _or not_, if there are better
libraries available. There's no point spending time "improving" things that no
one wants. Supporting newer libraries with better code and APIs may be much
more beneficial. This is now an option.

We'll be monitoring what people do and where pain points are. Building on all
the gem infrastructure, we open up a world of possibilities for Ruby
developers.

## Rubinius Inspirations

As 2.0 is a big milestone and transition, it's interesting to reflect on some
significant Rubinius contributions to the Ruby community.

Rubinius has inspired a number of projects that have benefited Ruby far beyond
Rubinius. The best known of these is [**RubySpec**](http://rubyspec.org), which
is used by every significant Ruby implementation, including promising new ones
pushing the limits, like [Topaz](http://www.topazruby.com) and
[Opal](http://opalrb.org).

Rubinius also created the initial **FFI** spec that opened the world of native
libraries to Ruby with a simple API across implementations and without needing
to write C-extenions.

Evan created [**Puma**](http://puma.io) to meet the need for a fast web server
that would promote the Rubinius parallel thread support. Puma also works well
on MRI and JRuby. It provides Ruby applications excellent performance and
multi-core scaling, especially when there's no global interpreter lock.

Rubinius as a language platform has inspired many
[**projects**](http://rubini.us/proceecs) and encouraged people who may have
thought language design was beyond their skills to experiment and discover the
tremendous joy in creating [**their own programming
language**](https://github.com/queenfrankie/lani).

It is a joy that Rubinius has been a part of these efforts and we will continue
improving developer experience in these areas.

## Ready, Set, Ruby!

Concurrent and distributed applications aren't the future anymore, they are the
present. They are vital to business success. The many talented developers that
are passing over Ruby for Erlang, Go, Clojure and Node are draining Ruby of
talent and vitality.

Ruby is an excellent language. Rubinius is dedicated to providing Ruby
developers with excellent tools and technology competitive with these other
languages. Developers who are happy writing Ruby shouldn't be forced to leave
it because of technical limitations.

If you are one of these developers, let's build the future.
