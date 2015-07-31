---
layout: post
title: "The Next 10 Million Programmers"
author: Brian Shirai
twitter: brixen
---

_This post is based on a talk given 2015 July 23 at [So Coded 2015](http://2015.socoded.com) in Hamburg, Germany. [The video](https://youtu.be/9ccfPCg9A6s) and [the slides](http://slidr.io/brixen/the-next-10-million-programmers#1) are also available. Thanks to [Enova](https://enova.com) for sponsoring me to speak._

---

We live in an _amazing_ time.

Recently, Pluto has been getting a lot of attention. In about 100 years, we went from the Wright brothers and the beginning of manned flight to sending a robotic spaceship billions of miles to fly within a few thousand miles of Pluto, capture around 50gb of data, and start sending it back to Earth. Ponder that for a moment, about 100 years.

Each of us with a smart phone holds more power for digital computation in our pockets than existed in the entire solar system 100 years ago. That means there are _hundreds of millions_ of times more power for digital computation than existed 100 years ago.

But, we're _not_ hundreds of millions of times smarter, healthier, happier, or wealthier. We have definitely made some progress, but by comparison to the power we have at our disposal, it doesn't even seem like modest progress. And in some cases, it's not progress at all. [U.S. median household income in 2013 was 1% _lower_ than it was in 1989](https://twitter.com/fordfoundation/status/621052045961793536).

Furthermore, the opinion that we're not very good at building computer systems is not particularly controversial. We are constantly reminded of our failures. It seems like every day there is news of some company's systems being compromised. Just recently it was vulnerabilities in Jeep's systems that would allow an attacker to remotely gain full control of an operating vehicle. Terrifying. This situation demands critical and effective changes in the way we build software systems.

So, this is the problem that faces us: How do we more effectively use this massive power of computation to help people be healthier, happier, and wealthier, making the world a better place to live?

My answer to these problems is that we need to foster a Cambrian explosion of programming languages. The [Cambrian explosion](https://en.wikipedia.org/wiki/Cambrian_explosion) was a period in Earth's history where most of the genetic diversity (and actually much more) originated. There are multiple theories about why this happened, but it's clear that some aspect of the biosphere, whether only evolution or whether geological or climatic change (or all of them), supported it.

There are two parts to my proposal: 1. programming languages themselves; and 2. increasing the support for people to experiment on the design of programming languages.

Before we get to that, let's first look at where we spend most of our effort today working with computer systems. Then we'll look at why we should focus on programming languages, and finally, ideas for how to foster this work based on my experiences working on Rubinius.

## Programming Software Systems

As programmers, a rather small portion of our time is spent strictly writing lines of code in some particular programming language. Much of our time is spent interacting with or working on other aspects of software systems:

1. New architectures: The current focus tends to be on a refinement of Service-orieted Architecture called "microservices". This architecture does improve the flexibility of systems but introduces its own set of complexities. Wholesale changes to a system's architecture are disruptive and costly. Also, architecture is about the arrangement of parts; the parts themselves must still be built well, and building these require languages. By itself, improvements in architecture will not help us build better software systems. However, better languages may help us build more effective architectures.
2. New infrastructures: distributed systems, "Cloud", containers, container OSes, and commodity hardware are all providing opportunities for reduced cost and greater availability and reliability of compute resources. The emphasis here is on opportunity because they don't magically have these benefits, they must be used and managed effectively. They also have significant risk, the most important being that you are dependent on vast areas of the system not under your control when using someone else's "Cloud" infrastructure or platform-as-a-service. As with architecture, this significant potential benefit doesn't help us build better software systems by itself.
3. New platforms: iOS and Android native apps and "one-page" JavaScript apps are increasingly common today. However, the different platforms impose substantial organizational cost. Different programming languages are needed, typically Java for Android, and Objective-C or Swift for iOS. This likely means different developers for different platforms. Utility libraries are often platform specific. There is also the increased complexity of designing and testing apps for these multiple platforms.
4. New frameworks: (Or anti-frameworks--the small modules approach currently popular in some areas of the Node/JavaScript community). Frameworks are critically language dependent and consequently are not typically suitable for evolving a system. For example, you won't use Rails and Django together on the same application. So it's not possible to leverage a feature in Django without re-creating it in Rails. Even when frameworks coexist, like Rails and Ember, they do so by staying on opposite sides of the room as it were.
5. New development methodologies: All development methodologies are substitutes for running code. Sometimes this is a good thing, but usually it's a bad thing. People appear to want to do anything to avoid shipping running code. Activities that help clarify the assumptions about a problem and sketch experiments to validate or invalidate the assumptions should be the limit of development methodology. The artifacts of the development methodology present their own language problem. The point is, development methodologies have only a minor role to play in building improved software systems.
6. New "general-purpose" programming languages: These must balance competing interests and resolve that tension in a reasonably generic manner. There are fundamental trade-offs in computation, like space versus time, and different circumstances are often best served with different compromises. General purpose languages can't play favorites. Due to their general nature, they typically provide many more language features than are needed for a particular task, but all those features need to be consistent and interoperate, increasing the cost and time to implement a general purpose language.

Despite their limitations, in each of these areas we see progress in more effectively and efficiently solving problems. There are two big issues I see that tend to be tightly coupled to these different areas: 1. the existence of "enshrined problems": components or processes that are "Too Big To Replace" and continually impose the cost of working around their failures; and 2. they don't mix well, forcing wholesale replacement to move forward. For example, you won't successfully build microservices with traditional enterprise architecture or waterfall development methodologies.

## Cost Gravity

Looking at these six areas, we can see that languages are an essential aspect of every one. What's the significance of this? Let's step back and ask the question: Why do we have hundreds of millions of times more compute power than 100 years ago?

The answer to that is this idea of _cost gravity_, an idea I learned from Pieter Hintjen's book [_Culture & Empire: Digital Revolution_](http://www.amazon.com/Culture-Empire-Revolution-Pieter-Hintjens-ebook/dp/B00GF48Z4S/). You've likely heard of the idea, too, expressed as Moore's law: the number of transistors on a chip doubles approximately every 18 months.

The idea is simple: over time, the cost of things goes down according to an exponential graph. The reason for that is because living things _process and share_ information. Every living thing does this. And _that_ is why we need to focus on languages; our _languages_ are the most powerful tool we have for _processing information_.

Elaborating on this idea of cost gravity a bit, there are three key inflection points to consider:

1. Impossible to possible;
2. Possible to general purpose; and
3. General purpose to specialized.

Computers are a good example of this dynamic. At one point, they were essentially impossible as the cost was too high. This doesn't mean just the material costs or cost to manufacture one, but also the cost to do the R&D necessary to even build one. At some point, the cost dropped enough that IBM, for instance, could build custom mainframes for certain companies. Eventually, the cost dropped enough that a middle-class household could afford a "general purpose" personal computer. Costs continue to drop and now a typical computer, for example, your smart phone, is actually a composition of multiple computers: the CPU, GPU, Flash memory controllers, accelerometer, etc.

Programming languages are stuck in the general purpose section of the cost gravity graph. I say, _stuck_, because at least in the last ten years, programming languages are behind the cost curve. It may be five years or it may be a little more than ten years, but I'm convinced that the cost to create a programming language today is far less than the cost we are paying to create one. Cost gravity assures us that the cost has fallen, but we are not capitalizing on this reduction in cost. In other words, we're paying a tax on all the things we do with computers by not taking advantage of the falling costs to innovate with programming languages.

Why are programming languages stuck behind the cost curve? I argue that they are behind the cost curve because we don't have enough people working on programming languages.

Recently, Business Insider ran a short piece on [the 12 most influential programmers today](http://www.businessinsider.com/most-influential-programmers-2015-7). Here they are:

![12 most influential programmers](/images/12-most-influential-programmers.jpg)

What's wrong with this picture? It's basically twelve white guys. There are certainly no people of color and especially no women of color. _You_, whoever you are, are also not in this picture. Equality is extremely important for the benefit of everyone. Diversity of viewpoints is important, but diversity of _problems_, and empowering people to solve their problems, is even more important.

So, here's where we are: We have this amazing power for processing information and we need languages to use that power effectively. How do we foster creating new languages? There are two parts: 1. _who_ do we need to make programming languages? and 2. _what_ tools do we provide?

The answer to the question of "Who?" is simple: we need as diverse a group of people as possible.

The answer to the question of "What tools?" is also pretty simple: we're mostly already using them and have been for the past 60 years or so. These include parsers, compilers, and virtual machines.

## Who We Need To Experiment With Programming Languages

Starting with who, how do we make this technology accessible to the people we want to reach and how do we structure interactions to promote their success? The answers is to build a distributed network of people who collaborate using the _advice process_.

### The Advice Process

I learned this idea from a book, [_Reinventing Organizations_](http://www.amazon.com/Reinventing-Organizations-Creating-Inspired-Consciousness-ebook/dp/B00ICS9VI4/) by Frederic Laloux. It's a simple and radical idea. Instead of needing some arbitrary authority to make decisions, people are generally empowered to make decisions guided by the advice process:

1. one needs to describe the problem;
2. one needs to explain their solution (decision);
3. one must seek _input_ from people who would be affected by the decision; and
4. one must seek _advice_ from people with _relevant expertise_.

### A Network Of Collaboration

In a typical hierarchical organization, one's authority to make decisions (ie solve problems) derives from one's position in the structure. Near the bottom of the hierarchy, one has very little authority. As one's position moves up towards the top of the hierarchy, one has more authority. Changing to a hub-and-spoke structure where one or a few people are at the center and the rest are mostly of equal level around that core doesn't fundamentally change the situation. In fact, it probably distributes even less authority than a typical hierarchy. Open source projects often have this hub-and-spoke structure.

A distributed network contrasts with the typical hierarchy or hub-and-spoke structure of an organization. One's position in the network shows how one collaborates with others. People are explictly empowered to make decisions and supported by a process that helps them do so effectively. If someone makes a poor decision (ie the outcome is not positive), that person or any other person can help fix the problem.

### Roles

Along with the distributed network structure and the advice process for making decisions, we also need to make roles explicit. Defining the roles doesn't mean that only certain people can do them, nor does it mean that they are fixed and unchanging. One or more roles can be concerned with helping to define roles!

Roles are necessary to give people an opportunity to see if they relate to the work and have, or want to learn, the necessary skills. Some roles may collaborate often and some not as much. Explicit roles are also needed to counter the tendency to see coding as the only important activity. In fact, most of the important roles have nothing to do with coding.

Some examples of roles that I think are needed:

1. Diversity Architect
2. Community Wellness Custodian
3. Accessibility Advocate
4. Readability Guardian
5. Under-represented Groups Seeker
6. Roles Designer
7. Discussion Facilitator

So, the distributed network structure, decision making with the advice process, and roles help with Who? Now, what tools do we need?

## What Tools Foster Programming Language Experimentation

First, here is a list of things that are common to general-purpose programming languages that I _don't_ want:

1. _Don't_ force choosing between static and dynamic typing unnecessarily;
2. _Don't_ force using a single type system;
3. _Don't_ force choosing between compiled to machine code ahead-of-time and using a managed runtime;
4. _Don't_ force choosing a programming paradigm unnecessarily (eg everything is an object or pure, lazy FP)

These four limitations of most current systems promote certain facilities that are useful in certain contexts above all others and force every solution to a problem to confirm to their limitations. That is wasteful and unnecessary. They also feed endless, and mostly useless, debates and fights about "the right way to do it".

What I _do_ want to promote are these three things:

1. Enabling a gradual process, over and over again, to build systems that track the cost gravity curve instead of repeatedly building systems that deviate significantly from it over time. In other words, _optimize for learning_;
2. Enabling low cost experiments with existing systems rather than needing to completely step outside the system to experiment; and
3. Building a reliable system out of unreliable parts that we know will have failures and bugs.

These are the things that I think are fundamental to achieving this:

### Instruction Set

A "machine" with an instruction set is a ridiculously useful abstraction that we have invented. However, people usually, incorrectly, fixate on an irrelevant attribute of a virtual machine: whether it has a stack or register based design.

In fact, stack and register machines are equivalent, but each has a slight advantage in certain contexts. Stack machines are very easy to write compilers for and the bytecode tends to be somewhat smaller than register bytecode due to the implicit stack locations. Register bytecode tends to be easier to write analysis tools for due to the explicit register locations. In Rubinius we can have both.

There are actually five distinct categories of instructions needed:

1. Stack
2. Register
3. Parsing
4. Assertion
5. Instrumentation

Parsing instructions provide optimized features to process a stream of bytes into a structure suitable for generating a stream of instructions. Having first-class parsing instructions significantly simplifies building a system up from simpler parts by parsing a simple language to bootstrap a more complex language.

Assertion instructions can halt the virtual machine or perform some other activity based on the correctness of the computation being performed. They cannot otherwise affect the computation. This makes them suitable for use during development or selectively in production, with the guarantee that the computation itself cannot be changed.

Likewise, instrumentation instructions can provide other dimensions of information _about_ the computation, for example, tracing values or interactions in the code, again with the guarantee that the computation is not being changed. By ensuring these introduce very little performance impact, and not being subject to failure modes like lock inversion, re-entrancy, or unexpected recursion, these instrumentation instructions are safe to use _dynamically_ on production code.

### Data and Objects

_Functions are for data, objects are for interactions_. Parsing a stream of bytes representing HTML or JSON into a structure to manipulate is a thing that functions excel at. You don't need to interoperate with bytes. They don't change. Furthermore, the function is completely defined for any possible byte input.

On the other hand, an agent in a complex system needs to have the absolute minimum coupling with other parts of the system or adaptation of the system as a whole won't be possible. Consider the analogy of ordering your latte at the coffee shop. If the you and the barista must have exactly the same opinions about every aspect of coffee to process your order, you're never going to get your drink.

Both of these abstractions are useful in particular contexts to solve particular problems and switching between them should be easy. Experimenting with a solution may use the loose coupling of objects and once the solution is well understood, it may be possible to define data and functions and isolate this now rigid component in a way that works well within the overall system.

### Universal AST and Intermediate Representations

The Clojure compiler doesn't share anything with the Haskell compiler, which doesn't share anything with the Go compiler, which doesn't share anything with the Elixir compiler, and on and on. This is ridiculously wasteful. Fundamentally, compiling Clojure has more in common with compiling Go or Elixir or whatever than any differences. To reduce the cost of language experimentation, we must make the tools needed to build the languages as broadly accessible, and sharable, as possible.

### Code Database

The data structure that is created to execute a program (eg machine code, bytecode, expression tree) is only one representation of the code. A code database enables associating an arbitrary number of dimensions of related data. This can support, for example, much more extensive analysis by associating runtime data with a dynamically typed program.

### Switching Between Machine Code and Managed Runtime

The path of developing a program is never a straight line. It's a chaotic thread of experimentation, learning, mistakes, and insights. Forcing this complex process to only function within the confines of a completely compiled program wastes too much time. The system should support fluidly, and repeatedly, switching between compiling a program to a single, machine-code executable or experimenting in a managed-code runtime.

### Ubiquitous Tools

Tools for doing language analysis and exploring the behavior of running code, without favoring a particular programming paradigm, are essential for programming language experimentation. Measuring, profiling, debugging, monitoring, and analyzing code must be available at all times, during development and in production.

## Arguments Against Specialized Languages

The only argument I've heard against language experimentation (besides the obviously false one that language X is all we need) is the idea that if people build many languages, the cost of learning them will be too high. This fear derives from the fallacy that a world of specialized languages is just like our world of "general purpose" languages multiplied N times.

First, most people will never learn most languages, and that's fine. You may create a language to solve a particular problem you have and that's it. And greater experimentation will inevitably finds its way into languages that many people do use.

Second, if the cost for a group to learn the language is more than offset by the increased utility (ie less cost to accomplish the work), then it learning the language is the best thing to do. This is why we learn languages now.

Third, we all already know many languages and dialects even when we don't recognize them as such. Using a CLI is using a language; using a GUI is using a language.

## No Free Lunch

There is no free lunch. Programmers seem to love this idea that there's this _one weird trick_ that can make everything work perfectly. Functional programming, static typing, containers, register virtual machines, etc. There is certainly optimization possible, but there are fundamental limits always. You're not going to exceed the speed of light. You can't infinitely compress information. You can't search a document arbitrarily fast. The fascinating thing to me is that in many areas we don't know what these limits are. The way we find out is by bringing to bear more powerful tools. Languages are the most powerful tools we have for processing information. Let's really make them powerful.

And while there is no free lunch, there is this amazing platform we have created called the internet. With the recent announcement of [WebAssembly](https://www.w3.org/community/webassembly/), we have the basis for taking everything I've described here _anywhere_ and to almost _any_ device. That is awesome. Here's to the next 10 million programmers making the world a better place.
