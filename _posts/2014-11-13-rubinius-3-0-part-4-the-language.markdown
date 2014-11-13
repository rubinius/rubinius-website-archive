---
layout: post
title: "Rubinius 3.0 - Part 4: The System & Tools"
author: Brian Shirai
---

Yesterday, I presented a look into the new instruction set in Rubinius 3.0. Today, I want to talk about changes to some of the bigger systems, like the garbage collector, just-in-time compiler, and the Rubinius Console. This will be the shortest post but we'll be writing a lot more about these parts in the coming weeks.

<em>I hope you're enjoying these posts and finding them inspiring. I'm excited to be bringing these ideas to you. In case you've been thinking about contributing or joining the Rubinius Team but are unsure if you want a lot of public attention, I wanted to share a book I've been reading: [Invisibles: The Power of Anonymous Work in an Age of Relentless Self-Promotion](http://www.amazon.com/Invisibles-Power-Work-Relentless-Self-Promotion/dp/159184634X).

To summarize, we're not recruiting rock stars. If you are one, that's great. For the rest of us, the online world can be very hostile at times, especially with the harassment of women and minorities that we are seeing on a daily basis. We must work hard to end these harmful actions, and at the same time give people safe places to work from. If you'd like to contribute but stay anonymous, we completely support you.</em>

## Rubinius System

The phrase "virtual machine" is most often used to refer to a system like Rubinius whose primary purpose is execute a program written in some programming language. The phrase is quite vague. The main subsystems in Rubinius are the garbage collector, the just-in-time compiler, and instruction interpreter (which we discussed in Part 3), and code that coordinates these components and starts and stops native threads. All of these are quite common in a system like Rubinius.

In this post, we also look at a set of tools that are deeply integrated into the rest of the Rubinius components. Sometimes these sort of tools are considered an after-thought. In Rubinius 3.0, we are approaching these tools as fundamental parts of the system.

## Garbage Collection

There are two changes coming to the Rubinius garbage collector. It will move toward a fully concurrent implementation and we'll be working on implementing near-realtime guarantees on any critical pauses. Also, we'll add a new type of memory structure that I'm calling a "functional object".

Right now in Rubinius, every object basically looks like the schematic below:

    +------------------------------+
    |       Object header          |
    +------------------------------+
    |       Object class           |
    +------------------------------+
    |       Instance variables     |
    |       ...                    |
    +------------------------------+
    |       Optional Object        |
    |       reference fields       |
    +------------------------------+
    |       Object-specific        |
    |       data                   |
    +------------------------------+

In Rubinius 3.0, there is an additional type of object:

    +------------------------------+
    |       Header                 |
    +------------------------------+
    |       Optional Object        |
    |       reference fields       |
    +------------------------------+
    |       Data                   |
    +------------------------------+

We are already using the second kind of object in Rubinius now, primarily for Bignum memory management, but we are formalizing and expanding our use of it to many other contexts.

## Just-in-Time Compiler

The Rubinius just-in-time compiler processes bytecode to generate native machine code "functions" that execute the methods in your Ruby program. The JIT leverages the [LLVM](http://llvm.org) compiler library to generate the machine code. Because most of the JIT is currently implemented in C++ to easily interface with LLVM libraries, it is distant from Ruby and not easy to work with.

The most important change for the Rubinius JIT is that we'll move it as deeply into Ruby as possible. The extremely difficult aspects of generating machine code, like register allocation, instruction selection, and instruction scheduling, will still be handled by LLVM.

The other changes to the JIT are architectural. Right now, when a method is called a lot, it will eventually be queued for the JIT to compile. During compilation, the JIT will use runtime type information to combine (or inline) not just the single method itself, but a chain of methods along the call path. There are several problems with this approach that are addressed by the changes below:

1. Multi-phase: A running program does not have exactly the same behavior at every point during its execution. Recognizing that programs have distinct phases of execution, the Rubinius JIT will adjust decisions to be more appropriate for that phase of the program.
2. Multi-paradigm: There are two very broad categories of JIT compiler: one essentially compiles a complete method at a time while the other compiles a specific _execution trace_. The Rubinius JIT is currently a method JIT. In some cases, especially hot loops, a tracing JIT may be more appropriate.
3. Multi-faceted: There is more than one way to improve performance of a piece of code, but right now the Rubinius JIT only has one way to do this. A multi-faceted JIT, on the other hand, will use many different approaches. Some methods may be transformed at the bytecode level, with new methods written from optimizing the original bytecode. Or methods may be split into many different ones depending on the types of values they see. The multi-faceted approach is not a set of tiers where higher tiers are better. It's the idea of better tailoring the kinds of optimizations to the features in the code.

The feature of the new Rubinius JIT that I'm most excited about is the _JIT planner_. Similar to a query planner in an RDBMS, the JIT planner will provide a way to record and analyze JIT decisions.

## Metrics

There are many moving pieces in Rubinius. Making sense of how they are performing and interacting is important. To support this, Rubinius includes a number of low-cost counters that capture key metrics of the Rubinius subsystems. Metrics make it possible to observe the actual effects of system changes in the context of production code.

The Rubinius metrics subsystem currently has a built-in emitter for StatsD. Other emitters can be provided if they are useful. Already, Yorick is sending Rubinius metrics to New Relic's Insights API to monitor an application's behavior.

This has tremendous value to us as we develop Rubinius. Many times we are asked about an issue in Rubinius and when we inquire about the application source code, we're told it's proprietary. We understand the need for protecting intellectual property, but it severely limits our ability to investigate. The metrics output makes it possible to share non-critical application information in a way that will help us improve our ability to address issues that you encounter with Rubinius.

## User Interface

The _user interface_ is not something we often discuss about programming languages. As language designers, we may think and talk about it. But I have seen far fewer such discussions between language designers and language users, and almost no serious, extensive studies of usability in language design. (If you have references, please send them!) 

One common discussion of usability that we do hear about in programming is the _Unix tools philosophy_, or the idea of _doing one thing well_. A simple program that does one thing well can be composed with other programs to do more complex things. I don't object to these ideas, but there's another side of the story no one talks about: _What is the system underneath that makes it possible to pipe output from one program to another?_

In art we may talk about figure and ground, and in building tools we must consider both the pieces the user interacts with, as well as the system behind those pieces. Over-emphasize the pieces in front and the user gets a bag of disparate fancy things that are collectively junk. Over-emphasize the system underneath, and the user gets a rigid, unwieldy block of granite that is equally unusable.

The Rubinius::Console is a set of tools combined with a coherent, integrated, systematic set of features that enable the tools to perform and coordinate well. I'll briefly talk about the main components below.

### Console

All the tools are built on the foundation of the REPL, or little-c _console_. A REPL generally takes commands, executes them, and displays the results. All of the tools here are part of the Rubinius::Console component. I want to give a brief introduction to each one for now, but we'll be writing a lot more about them soon.

### Inspector

The _inspector_ is a collection of features that enable tracing through the execution of a program and inspecting the program state. It can show the value of local variables, what methods are currently executing, and other aspects of the running program. These features are usually included in a separate tool called a _debugger_. We think these features should be available at any time, whether running in development mode on your local machine, or in production mode on a remote server.

### Measurements

When investigating program behavior, sometimes it is helpful to measure how long a particular piece of code takes to run. Typically, this requires setting up a separate run with separate tools to do a _benchmark_. We think the essence of a benchmark is simply measurement and it should be available at any time. 

### Relative Measurements

Another type of measurement is the relative measurement of multiple components, usually the chain of methods invoked to perform some computation. This is usually called a _profile_ and the focus is on the relationship between the measurements so that the most costly ones can be improved.

### Analysis

A running program has both an object graph and an execution graph. The object graph is the relationship between all the objects in the system. The execution graph includes all the call paths that have been run during the program execution. The execution graph is _not_ just the current stack of methods executing.

The analysis tools are available to investigate allocation issues or unwanted retention of references to objects, something often referred to as a _memory leak_. They can also investigate the execution graph to find relationships between code that are not visible in the source code due to Ruby's dynamic types.

## CodeDB

While a Ruby program is running, an enormous amount of important and useful data is generated in Rubinius. When the program exits, almost all that data is dropped on the floor. The _CodeDB_ will preserve that data, enabling it to be used at many points in the lifetime of a program, from the first line written to inspection of a problem in a running production system.

The CodeDB is more of a functional description than a specific component or piece of code. In Rubinius today, we store the bytecode from compiling Ruby code and read from this cache instead of recompiling the Ruby code. However, we still load all the code regardless of whether it is used. In Rubinius 3.0, we will only load code that is used, which will improve load time and reduce memory usage from storing unused code.

As we covered in the last post, the bytecode is merely one representation of Ruby. The CodeDB will enable us to store many representations of Ruby code across many invocations of the program, and potentially across many computers. The representations of Ruby code combined with the rich data created by running programs gives us the foundation for even more exciting tools. One of these may be a _refactoring editor_, which seems to be the holy grail of every object-oriented programmer. We think there are even more interesting tools than automated refactoring and are excited to tell you more about them.

Tomorrow, we will finally tie some of these pieces together in _Rubinius 3.0 - Part 5: The Language_.

## Acknowledgments

<em>I want to thank the following reviewers: Chad Slaughter, Joe Mastey, and the Rubinius Team, Sophia, Jesse, Valerie, Stacy, and Yorick.</em>

