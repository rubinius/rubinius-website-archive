---
layout: post
title: "Code Climate vs Rubinius"
author: Brian Shirai
twitter: brixen
---

It is difficult to understand the behavior of a program written in a dynamic language, like Ruby, without running the program. While static analysis, like [Code Climate](https://codeclimate.com), can tell us a fair amount about the code, there's still a lot more it can't tell us.

Wouldn't it be nice if the system running our program could tell us about what the code is doing while it's running? Rubinius can do this.

While a program is running, there are two graphs interacting. The first is the graph of functions (or methods) as they call one another. The second is the graph of data objects that the functions create or operate on.

In Rubinius, these two graphs intersect at the _inline cache objects_. Basically, wherever you have a method call in your Ruby program, when Rubinius runs your program, it creates a special object that records the _type of object_ on which the method is called and _what method_ is called. These simple Ruby objects record the graph of methods called in your program. From this graph, we can analyze all kinds of _actual behavior of your code_.

This sounds awesome, doesn't it? When was the last time you wondered how a bit of Ruby code in your program was interacting with other parts of the code? If you're like me, that's _every time I'm writing Ruby code_. So, how can Rubinius help? That's the problem, we don't have the tools that you need right now.

But I want to fix that. The question is, how? I need your help to decide.

Recently, Code Climate announced they were [releasing their platform as open source](http://blog.codeclimate.com/blog/2015/06/19/code-climate-platform/), "the first open and extensible platform for static analysis". One possibility to leverage the ability that Rubinius has to help you understand your code is to integrate with the Code Climate platform.

Another possibility is to create a stand-alone Rubinius service that would start with some simple, runtime analysis of your code, but could expand in many different dimensions, showing data-flow, security analysis, performance, and many facets of application analysis beyond what is possible with simple static analysis.

I want to emphasize that these two options are only superficially similar. The facilities that we have in Rubinius, and continue to expand and improve, can provide far greater depth of analysis than that possible with static analysis. So, the question is really, where do we start?

We'd love to have your input. Please take this short survey and let us know what you think.

Survey: [Code Climate vs Rubinius](https://goo.gl/forms/MWLEcLwFAS)

If you have more you'd like to share, write us <community@rubinius.com>.
