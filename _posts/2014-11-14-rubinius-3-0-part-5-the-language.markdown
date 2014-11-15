---
layout: post
title: "Rubinius 3.0 - Part 5: The Language"
author: Brian Shirai
---

This is the last post in the Rubinius 3.0 series. We started by talking about the Rubinius Team. In part 2, we looked at the development process and how we more effectively deliver features to you. Following that, we explored the new instruction set, which is the foundation for running Ruby code. And yesterday, we looked at the Rubinius system and integrated tools. Today, I'll talk about changes that I'm introducing to the language.

I mentioned that these posts were in order of importance. If we arrange the posts as in the figure below, we see that the Team and community form the foundation on which the development and delivery process is built. This gives us a basis for making good technical decisions, like the new Rubinius instruction set. In turn, that enables us to build more powerful tools for developers.

Finally, the language comes at the top. It's the least important piece, really representing the icing on the cake. The language is still important. After all, a cake without icing is a poor cake. However, the language needs to be seen in context and in proper relation to the rest of the system.

              +----------------------+
             /        Language        \
            +--------------------------+
           /       System & Tools       \
          +------------------------------+
         /        Instruction Set         \
        +----------------------------------+
       /   Development & Delivery Process   \
      +--------------------------------------+
     /            Team & Community            \
    +------------------------------------------+

Now that we see where language fits in, we can investigate it further. Is this chocolate icing or vanilla icing?

## Everything Is An Object

There's no gentle way to say this, you've been mislead about Ruby.

> Everything is _not_ an object, and objects are not everything.

I admit that I suffered this delusion that everything is an object for a long time as well, and I earnestly tried to convince others that this was true. This falsehood is causing us a lot of problems. Even worse, it's preventing us from fully benefiting from objects.

There's an important reason we use objects, and _that's_ the reason objects are useful. That may sound circular, but it's not. Objects are useful because of the problems they help us solve. They are not abstractly useful independent of any context. In fact, when we _misuse_ objects, they aren't very helpful.

In [The Power of Interoperability: Why Objects Are Inevitable](http://www.cs.cmu.edu/~aldrich/papers/objects-essay.pdf), the author suggests the following reason why objects are useful when writing programs. He actually goes further than _useful_ and suggests that either objects or something that simulates objects are _inevitable_.

> Object-oriented programming is successful in part because its key technical characteristic, dynamic dispatch, is essential to fulfilling the requirement for interoperable extension that is at the core of many high-value modern software frameworks and ecosystems.

Objects are useful because they allow pieces of a system to inter-operate _while_ they evolve under different rates of change by encapsulating information such that coupling (ie dependencies, brittleness) is reduced to a minimum.

The idea of _interoperability_ includes the ideas of interface, boundary, and integration. Objects inter-operate at their boundaries, which define the interface with other objects. To integrate well, those interfaces must match up well enough to do useful work. At the same time, where they do *not* match up must not interfere with doing useful work.

It's important to understand that "interoperability" is merely a fancy way of saying, _to share the work_. Everything here is about sharing the work. If A sends a message to B, A is relying on B to do the work specified by the message. A could just as well do all the work itself, but that would be wasteful if B already does exactly what A needs.

With objects, we have two ways of sharing work. When we inherit from a class in Ruby, or include a module, we are sharing work by being a _kind-of_ the thing that we inherit from. When we delegate work to another object that we reference, we are using composition, or _has-a_ relationship to share work.

This leads us to a new definition of "Object":

> an Object is something you can send a _message_, not a _thing_ you reference (i.e. hold onto in a variable or data structure).

This definition gives a simple, unambiguous way to identify objects: "Can I send this a message?". If the answer is No, it's not an object. The focus of message is on communication and behavior, not _thingness_. This is even more important when we consider proxies. The actual thing I send the message to is unimportant. Insisting that it _be_ a particular thing causes endless pain in programs. The proxy may handle the message or delegate it, and this decoupling and encapsulation of information is essential for interoperability.

Inheritance and composition give us what I call _the family and friends_ model of sharing work. But there's an important dimension missing from this model.

## Everything Is Not An Object

We have seen that objects provide two things: a way to share work, and a means of inter-operating by encapsulating information. There is more than one way to share work. We aren't all friends and family.

At this moment, writing this post, I'm sitting at my desk in my apartment in Chicago in Illinois in the USA on earth, and so on. This _boxes in boxes_ containment relationship is essentially about context. In the context of my kitchen, I may cook or clean dishes. I do not typically clean dishes in my bedroom. _I'm the same person_ in each of these contexts, but my behavior may be substantially different. Of course, some behaviors may be the same. Whether I'm cleaning dishes or sleeping, I certainly hope I'm still breathing.

We are familiar with this containment relationship in Ruby. In the following code, the method `name` returns the value of the constant `X`. Ruby finds the constant by looking up a chain of boxes that in Rubinius are represented by the ConstantScope objects.

```ruby
class A
  X = "Ruby"

  def name
    X
  end
end
```

There is a need in Ruby to better express this sort of relationship. We need objects to be able to share work without relying solely on friends and family. It turns out, there's a simple idea that provides this very ability: they're called _functions_. In Ruby, we've been so busy thinking that objects and functions are opposites that we didn't realize they are mostly complementary. I would say objects and functions are _orthogonal_, serving different and independent purposes.

### Functions

As we see with the constant search example above, containing lexical scopes exist in Ruby. In Rubinius, they are objects you can reference and send messages to. The lexical scopes provide a mechanism to relate objects and functions.

It turns out that Ruby's syntax is just flexible enough to permit us to use a syntax for functions that is reasonably consistent with the syntax for methods (except for the ugly `do` on the end):

```ruby
class A
  fun polynomial(a, b, c) do
    a + b * c
  end

  def compute
    polynomial 2, 4, 9
  end
end

A.new.compute
```

Just like the constant search for X above, the `compute` method can refer to the `polynomial` function because it exists in the method's containing lexical scope.

This [Boundaries](https://www.destroyallsoftware.com/talks/boundaries) talk by Gary Bernhardt is the best illustration of these ideas that I know of right now. I highly recommend watching it. I'm not going into depth about functions today, other than introducing them. They are a very well-understood area of computation and they are extremely useful. In the coming weeks, I'll write more about how we are using them to rewrite the Ruby core library in Rubinius 3.0

### Gradual Types For Functions

Related to functions are the concept of _types_. Types are a mechanism to ensure that for any "well-typed" expression, we are guaranteed that the result of evaluating the expression will be well-typed, and that evaluation will succeed. This idea is referred to as _progress and preservation_. Types are an extremely powerful tool, when properly applied.

Ruby's syntax is also flexible enough to permit adding type annotations like the following:

```ruby
class A
  fun polynomial(a: int64, b: int64, c: int64) do
    a + b * c
  end
end
```

Again, I'm not going into detail about types in this post. However, Rubinius 3.0 will include gradual typing for functions. The field of gradual typing is experiencing growing interest, as illustrated by this recent [talk by Philip Wadler](http://galois.com/blog/2014/10/tech-talk-by-philip-wadler/) at [Galois](http://galois.com/). We will apply the best current research on gradual typing in Rubinius 3.0.

There's one aspect of gradual typing that I do want to make clear: _Objects are the absolute worst place to put types because types conflict with the reason objects are useful_.

Objects need to provide the _minimum_ interface to inter-operate. In other words, objects need to be as isolated as possible. Objects also need to have the ability to be incomplete. This incompleteness, or _partial_ completeness, is not just defined by _something missing_. The partial-ness provides a space for behavior to evolve in a way that integrates with the already existing behavior.

In Rubinius, we have no intention to add typing to objects. Down that road awaits infinite pain and suffering.

### Multiple Dispatch

There's one final idea I want to present today: the idea of _multiple dispatch_ for functions and methods. For methods, dispatch (or sending a message), is now done only by considering the kind of object that the message receiver represents. Unfortunately, this forces a single method body to include logic for any number and kinds of objects that can be passed as parameters.

For example, `Array#[]` or element reference, can take different numbers and kinds of arguments. It might receive a single Fixnum, two Fixnums, a Range, an Object that responds to `#to_int`. I'd have to go look at the RubySpecs to know if I've covered the cases. This method is not unusual in the complexity of its interface. There are worse.

IO.popen is an egregious example. It has _at least_ 43 possible combination of arguments. Some of those arguments can partially overlap and the semantics when they do are essentially undefined. The APIs in the Ruby core library are embarrassingly messy. It's obvious that we need additional support in the language to handle the complexity without a mound of the proverbial balls of mud.

In multiple dispatch, the receiver, number of arguments, and kinds of objects passed as parameters are all considered when finding the correct method to handle the message that was sent.

By using multiple dispatch, we can write each method to handle the specific work that it needs to perform based on the kinds of objects it receives and correctly factor the shared work into a separate method. This improves our ability to comprehend the code while also improving the performance of the system as well.

In Rubinius 3.0, we are implementing multiple dispatch and using it to rewrite the Ruby core classes. Following the example above, we might define `Array#[]` as follows:

```ruby
class Array
  def [](index=Fixnum())
    # return element at index
  end

  def [](index=Fixnum(), num=Fixnum())
    # return num elements starting at index
  end

  def [](range=Range())
    # return elements from range.start to range.end
  end

  def [](index)
    # coerce index and dispatch
  end

  def [](index, num)
    # coerce index, num and dispatch
  end
```

The compiler that is used to compile the Rubinius 3.0 kernel will understand multiple dispatch, so successive method definitions add to, rather than overwrite, the set of methods that can handle a message.

A note about the syntax above: `def [](index=Fixnum)` defines a method that takes a single parameter that is a kind-of Fixnum. The "default argument" syntax in Ruby is the only thing that permits expressing this simply. To distinguish this _positional_ argument from a _default_ argument, note that `Fixnum()` has no value in parenthesis. In contrast: `def [](index=Fixnum(123))` defines a single _default_ argument with value 123. Passing a parameter that is a kind-of Fixnum will match, and if no parameter is passed, the value 123 will be used.

There's an additional aspect of the `Fixnum()` syntax that I want to highlight. It looks like a function or operation and that's important. These are *not* "types". They are match-syntax for a kind of object and also reflect an operation that would coerce an arbitrary Object instance into an object of the specified kind. In the case of Fixnum() or Integer(), it would be the #to_int method.

To summarize, we have these things in Rubinius 3.0: _functions_, _gradual types for functions_, and _multiple dispatch for methods and functions_.

## This Is Not Rubinius X

I want to emphasize that this is *not* [Rubinius X](http://x.rubini.us).

Rubinius X includes these ideas but has many additional features. My objective for introducing these features into Rubinius 3.0 is to massively reduce the complexity of the current implementation of Ruby, significantly improve the performance of Ruby, and build the foundation for Rubinius X (and other languages) to integrate with Ruby.

The ideas explained in the other posts about the new instruction set and the tools we are building are all focused on making it possible to transition existing applications to Rubinius X _without_ paying the cost of disruptive rewrites. With this in mind, here's one more thing.

## A New Machine

We are living at a time where active experimentation with languages are escaping academia and having a major commercial impact. There was a dreary day when it looked like Java, C#, and C++ would dominate programming. Thankfully, that's no longer the case. Very good new languages like Rust and Swift are commercially viable, and "experimental" languages like Haskell and [Idris](http://idris-lang.org) are making their way into industry. Very exciting!

While working on Rubinius, we have learned a lot about features that facilitate language development. However, underneath, we have been biased toward many features in Ruby. This has limited the utility of Rubinius in building languages with features that don't significantly overlap those in Ruby. However, as I've described in this post, Ruby's semantics are too limited to provide a language that is useful for many critical programming tasks today.

Accordingly, we are extracting a more useful language platform out of Rubinius as the [Titanius](http://titani.us) system. With the function support I'm adding in Rubinius 3.0, we will use dynamic (Ruby-like), static (C-like), and complex (Idris-like) semantics to refine our design and implementation. We want to ensure that the languages are able to maximally reuse existing components while still having the ability to express their own semantics in a fundamental way.

I hope you have enjoyed this series on Rubinius 3.0 and that it has given you a view into a much more useful and refined Ruby language.

There are so many hard problems that we need to solve. To be happy writing code, the language must solve the problems we have. Then we can help people using our products to be happy, too. Then businesses can be profitable by building those products that we are happy making. We can't avoid understanding this deeply and we _must_ take responsibility for it. I hope you'll join us on this journey.

<em>I want to thank the following people: Chad Slaughter for entertaining endless conversations about the purpose of programming languages and challenging my ideas. Yehuda Katz for planting the seed about functions. Brian T. Rice for trying to convince me that multiple dispatch was useful even if it took six years to see it. Joe Mastey and Gerlando Piro for review and feedback, some of it on these topics going back more than a year. The Rubinius Team, Sophia, Jesse, Valerie, Stacy, and Yorick, for reviewing and putting up with my last-minute requests.</em>

