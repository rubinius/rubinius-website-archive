---
layout: post
title: "Rubinius 3.0: The Third Epoch"
author: Brian Shirai
twitter: brixen
---

Happy New Year!

A little more than a year ago, I published a series of posts about the focus for Rubinius 3.0 ([part 1](http://rubinius.com/2014/11/10/rubinius-3-0-part-1-the-rubinius-team/), [part 2](http://rubinius.com/2014/11/11/rubinius-3-0-part-2-the-process/), [part 3](http://rubinius.com/2014/11/12/rubinius-3-0-part-3-the-instructions/), [part 4](http://rubinius.com/2014/11/13/rubinius-3-0-part-4-the-system-and-tools/), [part 5](http://rubinius.com/2014/11/14/rubinius-3-0-part-5-the-language/)). We spent a lot of time last year on some architecture improvements, so now it's time to get busy on those 3.0 goals. This post will catch you up to date.


## What's In A Version Number

I wrote two posts recently on the Rubinius [versioning scheme](http://rubinius.com/2015/09/22/major-minor-maximize-delivering-features-minimize-trouble/) and [release process](http://rubinius.com/2015/09/23/distributed-coding-distributed-releases/). Review those posts for details, but I'll summarize the main ideas here.

First, Rubinius uses a versioning scheme that associates a "version number", in the form of EPOCH.SEQ, with a particular git commit SHA via a git tag. The first part of the version number, the EPOCH, signifies "a period of time marked by notable events or particular characteristics" (see the dictionary definition). This post explains the Rubinius 3.x epoch.

The SEQ is a monotonically increasing number that has no other meaning than to signal that newer code is available. The Rubinius versioning scheme is emphatically _not_ [SemVer](http://semver.org).

A new Rubinius release should only do some combination of these three things: 1. introduce completely new code; 2. add a deprecation notice; 3. remove previously deprecated code.

Second, the Rubinius release process is fully automated and initiated by pushing a git tag of the form "vX.Y". The best part is that _any Rubinius contributor is now empowered to create a Rubinius release_.

Since the git tag does not exist independent of the git SHA that it references, a Rubinius release is a function from git tags (or the "release version" or label) to git SHAs. From math class, we may remember that a function can be represented by a set of pairs of the form `(input, output)`. In the case of Rubinius releases, that would be `(version label, git SHA)`.

The Rubinius versioning scheme makes trade-offs in favor of simplifying and accelerating the delivery of new features and fixes. The Rubinius release process makes trade-offs in favor of distributed, collaborative work without forcing people to synchronize and agree in advance. If a bad commit is tagged, any contributor can remove the git tag and push a new one. This provides resilience in the process and facilitates constant improvement.

Finally, the delta from pre-3.0 to 3.0 is inconsequential. Everyone should immediately update to the current 3.x and continue updating as quickly as we release new versions. As part of 3.x, we'll be introducing the facility to automatically update. _I'm so confident that this is the correct way to deliver software today that I'll be dog-fooding automatic updates in production._ If you haven't heard of working this way, I recommend checking out [Continuous Delivery: Reliable Software Releases through Build, Test, and Deployment Automation](http://www.amazon.com/Continuous-Delivery-Deployment-Automation-Signature/dp/0321601912/) and [Designing Delivery: Rethinking IT in the Digital Service Economy](http://www.amazon.com/Designing-Delivery-Rethinking-Digital-Service/dp/1491949880/).


## All-in On LLVM

With Rubinius 3.x, we are focusing exclusively a leveraging the amazing existing, and constantly improving, technology in [LLVM](http://llvm.org). We will also only support clang/clang++ for building Rubinius.

Rubinius has been using LLVM for a long time. Initially, it was complicated to do so because clang/clang++ were not yet mature and passing the C++ spec, and almost no platform had LLVM packages. That's changed dramatically in the past several years with clang/clang++ making huge strides, LLVM packages being nearly ubiquitous, and a massive on-going investment in LLVM by Apple to support the Swift programming language.

The top things we're focused on with LLVM are these:

1. Leveraging the full LLVM feature set to rewrite FFI to more completely interoperate with foreign code and to do so more naturally from Rubinius features (like the Rubinius instruction set).
2. Expanding the artifacts that we can produce with LLVM from only JIT-compiled methods to full ahead-of-time (AoT) executables. A lot of people really like Go because it generates executables, but there are not many actual differences between Go and Ruby. With a little help, we'll be generating executables for the same sorts of use cases that Go serves well. Don't expect to compile your massive Rails monolith, but if you're already working with simple, small services, you may be in luck.
3. Integrating LLVM functionality more completely into Rubinius. For example, integrating `lldb` to fluidly switch between Ruby source level debugging and machine code debugging (either external libraries or JIT-compiled code).
4. Creating a more useful set of intermediate representations (IRs) for describing the semantics of dynamic code before reducing to the semantics of LLVM's IR.

These areas of focus will help make Rubinius more useful across problems that often appear to be in opposition, like having an executable versus a flexible managed runtime that supports quick prototyping and experimentation. These usually represent _different phases_ in a program's evolution, and they tend to be cyclical, not linear. Being forced to choose up-front to use a compiled language or managed runtime is unnecessary.


## Elimating The Ruby

Two priorities for Rubinius 3.x are removing Ruby as a build dependency and de-coupling Ruby from the Rubinius core systems.

### Building Without Ruby

Using Ruby for the Rubinius build system was convenient and usually helpful, but it has a massive downside: installing Ruby is a major pain. So much so that [Homebrew](http://brew.sh) kicked Rubinius out as a project primarily because the Ruby build dependency was so difficult to manage. It has also frustrated the lives of numerous other package maintainers, from Gentoo to Arch to FreeBSD.

I've already started rewriting the build machinery to use Bash instead, which is a reasonable enough common denominator. The deploy automation is already using Bash. Once that is all working, we can look at supporting something like zsh, but for now Bash is way more ubiquitous and way easier to install than Ruby, so we're going to start there.

### Running Without Ruby

The second area we're removing Ruby is from the core components of Rubinius itself. It is already possible to boot Rubinius using a completely different core library instead of the Ruby one. However, there are still Ruby assumptions baked into the instruction set, managed object system (object memory and garbage collectors), and especially in the hundred or so _primitives_, C++ code that implements operations that are undefined in Ruby semantics (like adding two machine integers or doubles). As I'll explain below, these primitives will be completely removed.


## Performance, Memory Use, Startup Speed

This is the part that I'm most excited about. We will finally be seriously focused on performance and demonstrating the capabilities of Rubinius based on the years of work we've done to build a good architecture for efficiently running dynamic code with full support for multi-core parallelism.

### New Object Types

Things like bytes and primitive sequences of things don't need to "interoperate" and don't need to be "object-oriented". So we're adding non-object-oriented objects to the managed object system. This will support using "data" with "functions", enabling us to write better performing code in places and, more importantly, supporting languages with semantics that differ from Ruby.

### New Instructions

As mentioned previously, we're removing all the "primitives" in favor of an instruction set that completely maps to the language semantics we aim to support. This lets us work with those semantics across the entire compiler tool chain. Even more important, this lets us transfer _all_ our investment to another implementation of the instruction set. Two very promising targets for this are [WebAssembly](https://en.wikipedia.org/wiki/WebAssembly) and the [IBM J9 system](https://en.wikipedia.org/wiki/IBM_J9).

Ultimately, we are re-imaging the instruction set from an internal implementation detail to a [protocol](https://en.wikipedia.org/wiki/Communications_protocol) for communicating language semantics between the system supporting authoring the program and the mechanism executing the program.

As part of evolving the instruction set, we'll be splitting method dispatch, inline caching, and method invocation to support things like static method invocation, multi-method dispatch, and arbitrary method dispatch semantics. The inline caching mechanism is already the basis of our type-profiling JIT compiler and generalizing the caching will enable building more powerful, always-available code analysis.

### Better Garbage Collection

The Rubinius garbage collector is already _generational_ and _precise_, but the young generation semi-space collector has two downsides: 1. it requires a full stop-the-world phase to execute, and 2. it wastes half the memory allocated to it.

We'll be replacing the single young generation semi-space collector with a set of Immix collectors per Thread. (Immix is the collector we use in the mature generation.) The Immix-based collector has fast allocation, supports concurrent marking, can optionally compact, and compares very favorably in overall overhead with the semi-space collector.

### Evented Semantics

Rubinius already does threading very well, supporting excellent parallelism on multi-core hardware. However, threading isn't the only way to manage concurrency and as we've added more functionality, like the built-in performance counters that can stream out StatsD, we've seen a need for evented concurrency. So, we'll be building a first-class event loop into Rubinius with the ability to run multiple event loops, each on their own Thread.

### Blurring Writing and Running Code

One of the key components for supporting powerful runtime analysis, as well as the ability to AoT compile code is the CodeDB we're building. There's a [branch with some inital code](https://github.com/rubinius/rubinius/tree/codedb), as well as a [proof-of-concept](https://github.com/rubinius/rubinius/tree/lazy) I implemented almost five years ago.

The CodeDB assigns a unique ID to every executable context (in Ruby terms, scripts, class/module bodies, method bodies, block bodies). These IDs provide a "foreign key" of sorts to associate arbitrary dimensions of data (bytecode, LLVM IR, profiling data, coverage data, type profiles, etc.) with every executable context. This data can be used by other tools or analyzed to provide insight into program operation or performance.

The CodeDB also gives the ability to lazily load code and even to evict already loaded code that hasn't been used for some period of time, since reloading the code is possible at any time if it were to be needed.

We'll be building the CodeDB incrementally, focusing initially on lazy loading and basic program analysis features, then expanding the tooling support to enable more powerful analysis.


## Your Alternative for Ruby

When I started contributing to Rubinius over nine years ago, the alternatives for writing programs were much fewer. We've all witnessed the rise of languages and systems like Go, Rust, Clojure, Scala, Elixir, Crystal, Julia, Node.js and others. Each of these is driven by some need that existing systems did not adequately address (in the author's experience anyway).

Unfortunately, while these newer systems often have compelling features, using them requires discarding the previous investment in systems and applications. None of these systems encourage or support incrementally building better functionality in one portion of your existing system (unless you've already done the often significant work of building a microservice architecture).

In Rubinius, we want to provide a solid foundation for your existing investment, while opening up a world of possibilities to fix the most critical issues and continue to deliver value to your users or customers. Preserving your existing investment to the greatest extent possible maximizes the _new value_ you can create.

So, this is what the Rubinius 3.x epoch looks like. If you're interested, keep an eye on [the Rubinius releases](https://github.com/rubinius/rubinius/releases) and drop into [our Gitter channel for a chat](https://gitter.im/rubinius/rubinius).
