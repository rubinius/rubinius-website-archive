---
layout: post
title: "nil Is Not NULL, and Other Tales"
author: Brian Shirai
twitter: brixen
---

With the 2.3 release, Ruby has introduced a new operator. Designated the "lonely operator", this new Ruby syntax (`&.`) adds unnecessary complexity, inconsistency, and additional confusion for developers.

**Edit:** _Here is the [Ruby 2.3 News file](https://github.com/ruby/ruby/blob/trunk/doc/NEWS-2.3.0#L28-L41) describing the "lonely operator" (aka the "safe navigation operator"). And this is the [Ruby Redmine feature ticket](https://bugs.ruby-lang.org/issues/11537)._

Ruby is often criticized for code that developers cannot easily understand or reason about. This new operator creates a second way to call a method that doesn't improve code, nor does it improve the ability of a developer to reason about the code.

This post dissects some of the confusion about nil and explains the significant downsides of this new operator.

Fundamentally, there are two problems to solve: **1. Developers want their programs to be reasonably deterministic, and random runtime exception from an errant nil value are one of the most obvious things that interferes with that determinism; 2. Understanding the behavior of a program written with a dynamically-typed language requires specific tools.**

The "lonely operator" partially and inconsistently addresses the first problem, but with significant and unnecessary complexity that does not pay for itself. It does nothing to address the second problem.

## nil is an Object, NULL is a memory pointer

Sometimes developers from Ruby visit a mysterious and magical land, called Static Typing, and return with deliriously happy tales of programs that always work, are easy and inexpensive to write, and run for decades. Sometimes developers from that land visit Ruby and laugh. This makes some Ruby developers sad and they start thinking that Maybe if Ruby had Option "types", they could easily write perfect code that runs for decades, too.

Sadly, when this happens, Ruby developers are confusing a simple little Ruby _object_ for something that's usually radically different in "blub" language. Often, this other thing is a memory pointer, sometimes called NULL, which traditionally has the value 0. When 0 is used as a memory pointer, most computers very sternly complain, abort your program, and send you packing to the principal's office.

But in Ruby, nil is an object. You can use it as a value, you can call methods on it, you can define methods for it. It's _not_ NULL and it doesn't make your programs vulnerable to things that NULL makes your program vulnerable to.

Even when Ruby developers understand that nil is not NULL, they still often perceive it as some evil thing to be destroyed. This perspective usually comes from the fact that calling a method on nil that it doesn't implement raises a runtime exception, which causes your program to abort.

To be absolutely clear, a _segmentation fault_ and a _runtime exception_ are radically different things. But the consequence, your program aborts, makes them seem quite similar.

In a statically-typed language, even if the implementation doesn't treat its "nil-like" value as NULL, it still must resolve, _at compile time_, that a particular function or method will be called, and if its "nil" doesn't support that, something must be done. This characteristic of a statically-typed language makes nil _seem_ dangerous.

In Ruby, the fact that calling a method on nil that it doesn't understand results in a runtime exception is not a fundamental aspect of the language. Instead, it's a simple decision that was made, and it's possible to make a different decision, and you can do this in your own program at any time. We'll explore that aspect of nil later.


## nil is mathematically realistic

In mathematics, and specifically with functions and sets, the idea of nil is both necessary and useful (just as it is in Ruby, but that's discussed later).

A [_partial function_](https://en.wikipedia.org/wiki/Partial_function) is a function where every input may not map to an output. A value like nil signals that "no mapping is available for that input". It's a very useful concept.

With sets, we also know how to compute with such a concept. For example, consider set intersection, the operation that produces from two input sets, the set of members that both have in common. For example, { 1, 2, 3 } `intersect` { 3, 9, 12 } would yield the set { 3 }. In the case of two sets that don't share any members, the result of the intersection operation is the null set, often denoted by { }.

For some operation like, `A intersect B union C`, an exception is not raised if the result of `A intersect B` is the null set. There's nothing particulary odd or special about this; it's entirely natural.

So why is nil such a big deal in Ruby? Ah, now that's a good question.


## nil is mistreated in Ruby

There are two problems with the way nil is treated in Ruby. One problem, listed above, is that nil is misunderstood and considered an unwanted nuisance. The other, bigger problem, is that nil is ill-defined in Ruby. It's handicapped for no good reason.

In the first case, developers tend to see nil as an _error_, and hence, it is quite unwelcomed. "Oh look, that method returned a nil, something must be _wrong_!"

No, not at all! Nothing need be wrong. The method returned a nil to say, "no value here!"

Related to the problem of seeing nil as an error condition, some Ruby APIs treat nil as a marker (or sentinel) value. In Rubinius, this is such a problem that we had to introduce a special value we called "undefined" to be able to implement the Ruby core library in Ruby. The problem is that some methods take nil as a value, but also have a default value. So, we were unable to distinguish between `some_method()` and `some_method(nil)`.

Finally, because nil is ill-defined, _and_ because developers are inclined to see nil as an error condition, when a method is called on nil that it doesn't understand and a runtime exception is raised, it reinforces this dysfunctional relationship with nil.

This brings us to the ill-conceived "lonely operator".


## The "lonely operator" is an unnecessary mistake

The "lonely operator" supposedly solves the problem of calling a method on nil that then raises a runtime exception causing the program to abort.

Unfortunately, it only partially solves this problem, further requiring the newly introduced `#dig` methods (also unnecessary). It isn't necessary, and doesn't even solve the problem. What a mess.

Essentially, the "lonely operator" adds unavoidable complexity to syntax and programs, adds cognitive load to developers, and is an incomplete solution to the problem. Let's look at each of these in turn.

1. **It increases system and code complexity:** At every place a method is called, there must now be a decision about whether to use `.` or `&.`. The lonely operator _doubles the complexity_ of making a method call and due to interaction with other aspects of coding, significantly more than doubles overall complexity.
2. **It makes communication about code difficult:** How do we communicate to other developers when to use `.` and when to use `&.`? Why is it acceptable to ignore exceptions in one area of the code, but not in another? What happens when that decision is distant from the code you are looking at? What happens when those assumptions change?
3. **It doesn't solve the underlying problem:** The real, and legitimately painful, problem that developers need help with is understanding what their programs are actually doing when they run them. We'll look at this problem below.


## nil is A Good Thing™

The object nil in Ruby is neither a dangerous thing, nor a bad thing. It's actually a good thing!

If we focus on behavior, and objects inter-operating based on the behaviors they support, nil is a useful concept. It corresponds to "nothing", "no behavior here". It doesn't need to interfere with code functioning, and is only relevant when delivering a result to the user. We need to be able to say, "Hello there, that thing you requested doesn't actually have any representation". _If that even matters_. Sometimes it doesn't matter at all, and nil is just a blank.


## The simple alternative to the "lonely operator"

The only thing that the "lonely operator", and the new `#dig` methods, provide is the ability to ignore runtime exceptions from calling a method on nil.

A very simple alternative with no special syntax has existed in Ruby forever. Let's see how that works.

First, we recall that nil is a singleton value of NilClass. Nothing special, just an object that already responds to a few methods, like `#to_s`, `#to_h`, `#to_c`, `#to_a`.

In Rubinius, you can find where a method in the core library is defined by calling `#inspect` on the Method object:

```
irb(main):001:0> nil.method(:method_missing)
=> #<Method: NilClass#method_missing (defined in Kernel at kernel/delta/kernel.rb:46)>
```

We can see that NilClass has inherited `#method_missing` from Kernel. Simple, we'll just open NilClass, define our own `#method_missing`, and see how that works.

{% highlight ruby linenos %}
class NilClass
  def method_missing(*)
    self
  end
end
{% endhighlight %}


Here's the way to look at it: _nil is the value (or object) that turns every method into the identity method_.

The concept of an identity function is fundamental in math. A function `f(x) = x` for all `x` is the identity function; it returns its input unchanged.

We could reverse this perspective and talk about the value instead of the function. We could say, NIL is the value that when passed to any function, the result is NIL: `f(NIL) = NIL` for all functions `f`.

In Ruby, we can do this with nil: `nil.m => nil` for (almost) any method `m`.

So, with the very simple addition above, let's compare some code, first on Ruby 2.3.0 and then on Rubinius 3.5:

```
$ ruby -v
ruby 2.3.0p0 (2015-12-25 revision 53290) [x86_64-darwin15]
$ irb
irb(main):001:0> a = nil
=> nil
irb(main):002:0> a&.+2 * 3 + 5
=> nil
irb(main):003:0> h = {a: 1}
=> {:a=>1}
irb(main):004:0> h[:b][:c][1]
NoMethodError: undefined method `[]' for nil:NilClass
	from (irb):4
	from /Users/brianshirai/.rubies/ruby-2.3.0/bin/irb:11:in `<main>'
irb(main):005:0> h.dig(:b, :c, 1)
=> nil
```

Now, for Rubinius:

```
$ ruby -v
rubinius 3.5 (2.2.0 1453a0a5 2016-01-10 3.5.1 JI) [x86_64-darwin15.2.0]
$ irb
irb(main):001:0> class NilClass
irb(main):002:1> def method_missing(*)
irb(main):003:2> self
irb(main):004:2> end
irb(main):005:1> end
=> :method_missing
irb(main):006:0> a = nil
=> nil
irb(main):007:0> a + 2 * 3 + 5
=> nil
irb(main):008:0> h = {a: 1}
=> {:a=>1}
irb(main):009:0> h[:b][:c][1]
=> nil
```

We can see that we need to use undeniably more complex syntax (`a&.2`), and _inconsistent_ syntax (`#dig`), to achieve the same thing on Ruby 2.3.0. In contrast, with _no syntax changes_ on Rubinius 3.5, by merely using fundamental Ruby features (ie defining a method), we have consistent syntax and the same result.

Avoiding runtime exceptions when calling methods on nil is both easy and natural in Ruby. No special syntax and no extra confusion required. But solving _this_ part of the problem isn't that important. It's the second part of the problem that is way more important to solve for developers.


## Ruby developers need to see where nil is

The real problem that developers need Ruby to solve is the ability to know where nil values come from when they are _not_ desired. As demonstrated above, the simple solution to _computing with nil_ without causing runtime exceptions already exists in Ruby and has since forever.

This problem of knowing where a value comes from is much bigger than nil. It is a result of the fundamental tradeoff that a late-bound language (usually called a _dynamically-typed_ language) makes relative to an eagerly-bound language (usually called a _statically-typed_ language).

The tradeoff has a massive benefit that is under-appreciated. Late binding provides a malleable system that easily manages the complexity of high-uncertainty contexts. Objects can interact with other objects that provide certain behaviors. They _do not_ have to be specific _kinds_ of objects. The lessening of the constraints that _the developer's assumptions_ impose on the system can increase the utility and resilience of the system.

Unfortunately, as is often the case, we are seeing the world in black and white, picking one side, and missing half the picture. Late bound languages make simple code able to manage a lot of runtime complexity, but also potentially add a lot of confusion for the developer trying to understand what the runtime behavior actually is in a particular case.

The solution to the problem of understanding runtime behavior is a system that provides rich analysis features for the developer. Rubinius is building a system like this. Having a general solution is great, but we can get a lot of benefit from focusing specifically on nil.


## Traceable nils in Rubinius

In Rubinius, there are two types of "objects". There are objects like an Array, Object, or Hash instance, or an instance of some class in your code. These objects have _two parts_: 1. the object's data, which lives somewhere in memory, and 2. the object's _reference_, or pointer to the location of the object's data.

There's another kind of object in Rubinius. These are called _immediate values_ because their data and their "reference" are the same thing. These are also called _tagged pointers_ because the value is essentially a memory pointer where we've set one or more "tag" bits.

Values like `1`, `0xcafe`, `true`, `false`, and `nil` are immediate values, or tagged pointers, in Rubinius. This is what nil looks like as a (binary formatted) pointer value `0b11010`. If the least significant five bits of a pointer match that value, the value is considered by Rubinus to be nil.

In the past, we have only ever used that precise value. In other words, all the other bits are zero. But nothing requires this, and those other bits don't need to be wasted. On 64bit architectures, this gives Rubinius approximately 2^59 values of "nil". That's more than enough for a typical Rails app, I'm sure.

So, how can we use this abundance of nil values? Easy! When a method returns nil and that nil is not a value that was propagated from a value passed to the method, we can return a nil that is tagged for that specific method. The value behaves exactly as nil, when nil was a singleton value. But now we can find the source of the nil when we encounter it later. We can trace various paths of specific nils through code and help the developer understand why a particular value is nil.

_This is the problem that developers need Ruby to solve_. It's already possible with zero extra complexity of syntax, communication, and cognitive load to avoid runtime exceptions when calling methods on nil. But _the system_ needs to help developers understand how their code is functioning, and why it is functioning that way.

There's only one other feature we need to add to Rubinius, return value type caching. We already cache the type of values at method call sites, and those caches help the JIT generate more efficient machine code. With return value type caching in place, the JIT will improve and Rubinius runtime analysis tools will be even more powerful.


## A system must support _writing_ AND _running_ code

Everything that we've been looking at in this post points to a fundamental concern with programming: Our systems have been woefully incomplete. There are extremely few problems that we know everything about up front. Most "interesting" problems have significant novelty or they would already have simple solutions. To support writing programs for these problems, we need to help the author write code _and_ we need to help the author understand the code as it runs.

This is the fundamental problem that Rubinius is focused on. If that interests you, come hang out with us and talk about it [in our Gitter chat](https://gitter.im/rubinius/rubinius).
