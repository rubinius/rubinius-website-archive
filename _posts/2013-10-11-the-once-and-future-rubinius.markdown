---
layout: post
title: The Once and Future Rubinius
author: Brian Shirai
---
Engine Yard has [posted their
statement](https://blog.engineyard.com/2013/the-future-of-rubinius) about
ending sponsorship for Rubinius, which gives me the opportunity to clearly
address the future of Rubinius.

First of all, Engine Yard deserves great respect and admiration for their
contribution to Rubinius and the entire Ruby community. I had the pleasure of
interacting often with three of the Engine Yard founders: Tom Mornini, Lance
Walley, and Ezra Zygmuntowicz. I have rarely had the good fortune to work with
people as ethical, careful, and visionary as these folks. They endeavored to
build community _and_ business together, and they were highly influential in
both.

Engine Yard's sponsorship of Rubinius certainly accelerated development and
brought the project to the attention of many developers. Additionally, Engine
Yard's sponsorship contributed to the success of
[RubySpec](http://rubyspec.org) as an idea and tool for unifying Ruby
compatibility across more than a half-dozen significant implementations of Ruby
for the benefit of the Ruby community.

So, thank you very much, Engine Yard!

The simplest statement about the status of Rubinius is that there are now zero
people paid to work on the project. This fact has several implications, none of
which are inherently negative.

On the one hand, Rubinius is free to aggressively pursue  the goals of the
project in helping build the future of Ruby. On the other hand, I have
significantly less time to devote to the project. While unfortunate, I'm not
discouraged. I worked on Rubinius for over a year before Engine Yard hired me
and we accomplished a tremendous amount.

We still have numerous things yet to do. Over the past several weeks, I have
been working to simplify and focus the project so that all the time we can
invest pays significant rewards for developers and businesses. We'll continue
to streamline and accelerate delivering value to the people investing their
time to use Rubinius.

Rubinius has a broad and ambitious vision. Since Evan Phoenix created it,
Rubinius has been pushing the envelope. It was one of the first projects in the
Ruby community to use git. One of the first big projects on GitHub. One of the
first projects to use LLVM outside of the LLVM ecosystem. There have always
been skeptics voicing their opinions about Rubinius using Ruby, building
RubySpec, building our own virtual machine and garbage collector, removing the
global interpreter lock, using gems, about almost every aspect of the project.

Despite this, Rubinius keeps moving forward. People are experiencing the
tremendous value of running concurrent applications on modern hardware,
saturating the CPU cores instead of blowing out the memory. It's trivial to
migrate from MRI to Rubinius, continuing to use familiar platform tools and
running C-extensions. The terrific response to the 2.0 announcement has been
ample validation of our vision for Rubinius. We're just getting started.

Visit us in the #rubinius channel on Freenode and check out ways you can
[contribute](http://rubinius.com/doc/en/contributing/) to the project. The
simplest, and always the most fun, way to contribute is to use Rubinius to do
something you find interesting.

The future is, by definition, undefined. Let's define it.
