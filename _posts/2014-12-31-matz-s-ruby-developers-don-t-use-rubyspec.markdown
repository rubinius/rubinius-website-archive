---
layout: post
title: "Matz's Ruby Developers Don't Use RubySpec and It's Hurting Ruby"
author: Brian Shirai
twitter: brixen
---

Matz's Ruby team released version 2.2.0 a few days ago. This release fails or errors on multiple RubySpecs, and at least one spec causes a [segmentation fault](https://gist.github.com/brixen/8a12928e5fe0e63b340b), which is a serious error that can sometimes be a security vulnerability. All of these issues could have been easily avoided if MRI developers used RubySpec.

<script src="https://gist.github.com/brixen/416d681828f350ccddf1.js"></script>

[Full output of MRI 2.2.0 running RubySpec](https://gist.github.com/brixen/8a12928e5fe0e63b340b)

Ruby is an extremely complex language. It has no formal definition. I created the RubySpec project to provide this definition and to help implementations check correctness. MRI developers have released versions 1.9.3, 2.0.0, 2.1.0 and now 2.2.0 without checking correctness with RubySpec, nor have they created RubySpecs to define the Ruby behaviors they have implemented for these versions, to the detriment of all Ruby programmers and businesses who depend on Ruby.

As of today, I'm ending the RubySpec project. The personal cost of RubySpec over the past eight years, as well as the cost to Rubinius are greater than any benefit derived from the project.

Independent of RubySpec, the Ruby community needs to address this issue. The MRI developers [claim to own Ruby](https://bugs.ruby-lang.org/issues/7549#note-2), so it's their responsibility to clearly define the semantics of Ruby and ensure the correctness of their implementation.

Instead of contributing to RubySpec, the MRI developers write their own tests. That is their choice to make. As the issues here are complex, I want to provide some historical context and explain why the MRI tests are inadequate.


## Ruby Is What Ruby Does

When I first began contributing to Rubinius in late 2006, I knew two things: I wanted Rubinius to be successful, and I wanted it to accurately reflect MRI behavior so that developers could simply switch to Rubinius to run their Ruby programs.

At the time (around version 1.8.3-4), MRI had almost no tests. There were two previous projects to attempt to write tests for Ruby: RubyTest and BFTS. Both of these projects were also very limited and used _ad hoc_ facilities for handling platform differences.

My approach to the specs was simple: The way a Ruby program behaves is the definition of Ruby. So, we began to comprehensively write small snippets of Ruby code to show how Ruby behaved when run by MRI. Then we would match that behavior in Rubinius.

Writing tests for a language as complex as Ruby is tremendously difficult.  Even for a single implementation, there are platform issues like endianness and machine word size. Now we needed to also account for different implementations and, with the beginning of Ruby 1.9 development, completely different versions with syntax incompatibilities.

I started developing a consistent set of facilities to handle all these challenges as well as writing a separate spec runner that was compatible with RSpec 2.0 syntax so the specs could be run by RSpec. The custom spec runner used very simple Ruby features to enable nascent Ruby implementations (like Rubinius) to start running specs with very little preparation.


## The Birth of RubySpec

On May 10, 2008, just before RailsConf, I created the RubySpec project, by extracting the specs we had been writing for Rubinius, in the hope that MRI and other projects would contribute to it and use it. Some people had already started questioning why MRI did not use the specs.

Later that year, at RubyConf 2008, I gave a talk titled, [What Does My Ruby Do](http://confreaks.com/videos/269-rubyconf2008-what-does-my-ruby-do) about RubySpec. Matz and several other MRI developers attended. Immediately after my talk, contributors to Rubinius sat down with Matz and other MRI developers to discuss their effort to create an ISO specification for Ruby. We asked whether RubySpec could be part of the specification but were told that it was not appropriate to include it.

Fast forward to the pending release of MRI 1.9.2. The transition from 1.8.7 to 1.9 had been torturous. There were a number of behaviors introduced in 1.9.0 and 1.9.1 that were reverted in 1.9.2. The Ruby community was very reluctant to adopt 1.9 because of the confusion about 1.9 behavior, instability of 1.9.0 and 1.9.1, and the cost of migrating code. The MRI developers had started writing more tests, but there was still almost no participation in RubySpec.

That changed suddenly when Yuki Sonoda ([@yugui](https://twitter.com/yugui)), as the 1.9.2 release maintainer, stated that she would not release 1.9.2 until it passed RubySpec. There was a flurry of activity that all but ceased when 1.9.2 was released.

No release maintainer since then has asserted that requirement. MRI developers have written many MRI tests in the last several years. As described below, there are still Ruby features for which there are no MRI tests, but they are writing tests. However, MRI developers have essentially written no RubySpecs for the 2.0, 2.1, or 2.2 features they have implemented.


## The Problem With MRI Tests

Not too long ago, prominent Rubyist Erik Michaels-Ober asked me, "What's wrong with MRI's tests?" I was surprised by his question.

Since Erik, who is an experienced Ruby and Rails developer, asked this question, I imagine other people have wondered the same thing.

From the perspective of adequately defining Ruby semantics and providing a mechanism for other Ruby implementations to check correctness, here are the problems with MRI's tests:

1. **They include MRI implementation details.** One very difficult aspect of specifying Ruby involves the boundary between a Fixnum and a Bignum. This impacts almost every method in Array, String, Range, etc. Since MRI is written in C, the machine types supported by C leak into Ruby behavior. One place this happens is the accepted range of values that can be used to index an Array. Since MRI uses a C `long`, there are some values that are bigger than a Fixnum that can be used to index an Array. For an implementation that isn't dependent on C, these hidden semantics expressed in MRI tests [like these for Array](https://github.com/ruby/ruby/blob/47d66a8f2fed4a0901ece2d5a699d0c14f9341b8/test/ruby/test_array.rb#L1845-L1858) make the tests unsuitable.

    <script src="https://gist.github.com/brixen/3efb81d1e82a15d8555c.js"></script>

1. **They include MRI bug details.** Rather than improving the general quality and coverage of the tests, MRI adds [specific bug conditions](https://github.com/ruby/ruby/blob/47d66a8f2fed4a0901ece2d5a699d0c14f9341b8/test/ruby/test_lazy_enumerator.rb#L249-L269) to the tests. These bugs are irrelevant to other implementations. Moreover, littering the tests with specific bug cases instead of improving the overall quality makes the test suite harder and harder to maintain over time.

    <script src="https://gist.github.com/brixen/8a2801587e69adf21844.js"></script>

1. **They have no facility for distinguishing versions.** The tests are specific to whatever version the MRI code supports. However, it's not that simple. MRI develops features by pushing new commits directly into their Subversion repository trunk (which is somewhat like the master branch under Git), and then periodically pulling certain specific commits into a release branch. This makes it extremely difficult to track the MRI features that are intended for an upcoming version.

1. **They have no facility for distinguishing non-MRI platform issues.** As stated in the first point above, there are MRI-specific semantics that are not shared by other implementations. The MRI tests assume these semantics and this makes the tests much more difficult to use.

1. **They include behavior that Matz has explicitly said is undefined.** Matz has explicitly said that [modifying a collection while iterating is undefined behavior](http://blade.nagaokaut.ac.jp/cgi-bin/scat.rb/ruby/ruby-core/23633).  Yet, [the MRI tests include this behavior](https://github.com/ruby/ruby/blob/47d66a8f2fed4a0901ece2d5a699d0c14f9341b8/test/ruby/test_array.rb#L2297-L2319). Another Ruby implementation must exclude these tests if, for example, they result in an infinite loop. But since MRI can change the tests at any time and add more of these types of tests, this requires constantly checking which tests exhibit undefined behavior and omitting them.

    <script src="https://gist.github.com/brixen/e731f0a836302e7bc3a5.js"></script>

1. **They are not discoverable.** The tests are roughly divided into `test/` and `test/ruby`, but locating the tests for a specific class and method is not possible through any standard convention. One must constantly grep the code and attempt to determine if the test is specifically for that method, or if the use of the method is coincidental.

1. **They combine too much behavior into a single test.** Ruby includes very complex behaviors and there are many different aspects of those behaviors that may depend, for instance, on which parameters are passed or the kind of parameters passed. This [example of `Range#bsearch` tests with Float values](https://github.com/ruby/ruby/blob/47d66a8f2fed4a0901ece2d5a699d0c14f9341b8/test/ruby/test_range.rb#L414-L463) spans 49 lines and is hardly the most confusing test in MRI. Or this test, titled `test_misc_0`, that [asserts something about Arrays](https://github.com/ruby/ruby/blob/47d66a8f2fed4a0901ece2d5a699d0c14f9341b8/test/ruby/test_array.rb#L99-L137). It is extremely difficult to identify the key boundaries of Ruby behavior and implement them correctly.

    <script src="https://gist.github.com/brixen/d20fba48b7c93afc78d8.js"></script>

    Another example is `Process.spawn`, which has at least 43 distinct forms of passing parameters. It is very difficult to understand that complexity [from these MRI tests](https://github.com/ruby/ruby/blob/47d66a8f2fed4a0901ece2d5a699d0c14f9341b8/test/ruby/test_process.rb).

1. **They use _ad hoc_ facilities for setting up state and sub-processes.** There are three basic kinds of specs one can write for Ruby: 1) a value computed with no side effects, 2) state change (side effects) internal to the Ruby process, and 3) state change (side effects) external to the Ruby process. The first kind of specs are the simplest and easiest to write. The latter two often require starting a new Ruby process to execute the test.  The MRI tests use various different ways to do this, including `IO.popen`, `Process.spawn`, `system`, etc. There are also many different ways for setting up the state for the test.

1. **They are often incomprehensible.** This issue of the difficultly of understanding the tests is part of several of the previous points, but it stands on its own as well. It is often not clear at all what behavior the test attempts to show. This makes implementing Ruby much more difficult. For example, [what exactly is the behavior here](https://github.com/ruby/ruby/blob/47d66a8f2fed4a0901ece2d5a699d0c14f9341b8/test/ruby/test_process.rb#L257-L315)?

     <script src="https://gist.github.com/brixen/874a76d4ea12d9adc838.js"></script>

1. **They are incomplete.** Ruby is complex, so it's not completely surprising that testing Ruby is hard. However, this is MRI. They are defining the behavior of Ruby. Complexity is no excuse whatsoever for incomplete tests.  The consequence of incomplete tests is serious. Recently, Rubinius implemented `Thread::Backtrace::Location` from the documentation in MRI.  There are no MRI tests for `Thread::Backtrace::Location#path`, a method that Rails 4.1 started depending on to initialize Rails. The MRI implementation appears [to have several bugs in it](https://bugs.ruby-lang.org/issues/10561). This resulted in a [Rails issue](https://github.com/rails/rails/pull/17782), a [Rubinius issue](https://github.com/rubinius/rubinius/issues/3223), a bunch of time wasted and all our Rails 4.1 applications being broken and requiring a monkey patch while waiting almost a month for Rails 4.1.9 to be released, which still hasn't happened. It is unreasonable that the developers defining Ruby features are not writing tests for them.

The facilities and structure I built for RubySpec address all of these significant problems with MRI tests, except for completeness, which is impossible for RubySpec to solve while MRI continues to change the definition of Ruby without adequately specifying the behavior.

This issue of completeness is extremely difficult given the complex behaviors of Ruby in areas like processing keyword arguments. When calling a method, the object passed as a parameter that is considered to be a candidate for keyword arguments is determined by hidden semantics. A single object will sometimes be split into two separate Hashes, at other times an exception will be raised. The semantics are extremely complex. While writing numerous RubySpecs for this, I encountered several bugs and filed issues with MRI. Some of those bugs persist in Ruby 2.0 and 2.1.

The existing MRI tests are simply inadequate to define the semantics of Ruby and require significantly greater cost than necessary for other implementations to use to implement Ruby behavior.


## Developers And Businesses Suffer From Poor Ruby Quality

I think it's unreasonable for the MRI developers not to use and contribute to RubySpec. I don't think it's acceptable to release a stable version that segfaults when running an easily accessible and valuable test suite. Businesses who depend on Ruby and employ developers to write Ruby deserve a more mature, more rigorous design and quality assurance processes.

I have personally discussed RubySpec with Matz on multiple occasions. I have sat with Matz and other MRI developers to discuss RubySpec. I have advocated for using RubySpec in my talk, [Toward a Design for Ruby](http://www.confreaks.com/videos/1278-rubyconf2012-toward-a-design-for-ruby), in the feature request [A Ruby Design Process](https://bugs.ruby-lang.org/issues/7549) on MRI's feature / bug tracker, in my blog posts explaining my proposed design process ([A Ruby Design Process](http://brixen.io/2012/12/11/a-ruby-design-process/), [A Ruby Design Process - Talking Points](http://brixen.io/2012/12/30/a-ruby-design-process-talking-points/)), and in [my petition to adopt the proposed design process](http://rubyspec.org/design/).

RubySpec has existed for almost eight years, but has clearly failed to suit the needs of MRI developers. That's disappointing to me, but I no longer view it as a problem I can help solve.


## The Future Is Bright

Ultimately, I've made the decision to end RubySpec for the benefit of Rubinius as a project and to support current and future contributors.

There is a significant opportunity cost for Rubinius in supporting RubySpec. We have the ability to experiment with even better approaches to specifying Rubinius features. For example, a literate programming style that combines Rubinius documentation and code examples to serve as tests. Or a custom language that makes the specs easier to write and understand.

Attempting either of these approaches is untenable if broad compatibility across implementations is a requirement. Moreover, Rubinius needs to be free to prioritize efforts toward things that benefit Rubinius, just like all the other Ruby implementations have done.

I am more excited about Rubinius than I have been since early 2007. We continue to write very high quality specifications of the Ruby behavior we support, and all the new features that are being created. RubySpec was born in Rubinius out of a desire for the best Ruby support we could create. Whatever approach we take, that goal is deeply embedded in Rubinius.

Rubinius is getting better every day. It's the easiest way to transition your applications from MRI to a system with a just-in-time machine code compiler, accurate generational garbage collector, an evolving set of comprehensive code analysis tools, and concurrency support for all those CPU cores we have these days. If those things are important to you, we'd love to hear from you.

## Acknowledgments

<em>While the decision to end RubySpec was mine alone and not everyone fully agrees with it, I received tremendously helpful and generous feedback from many people. Special thanks to Chad Slaughter, Tom Mornini, Sophia Shao, Jesse Cooke, Yorick Peterse, Gerlando Piro, Giles Bowkett, Kurt Sussman and Joe Mastey.</em>
