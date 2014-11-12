---
layout: post
title: "Rubinius 3.0 - Part 3: The Instructions"
author: Brian Shirai
---

So far in this series, I've talked about the Rubinius Team and our approach to building Rubinius and delivering it to you. Today, I'll start talking about technical aspects of Rubinius 3.0, beginning with the new instruction set.

In this context, I want to reiterate what I wrote in the first post about over-emphasis of technology. In many projects, there appears to be an implicit assumption that those who code do the "technical" tasks, while beginners or those who can't code do the "non-technical" tasks, and the latter are inherently less valuable. This often goes unquestioned, but it's obvious that we spend time on what we consider valuable. If documentation is lagging, it's less important.

One manifestation of this that has always bothered me is tagging issues for "beginners", or suggesting that beginners start out with tasks like documentation, or other "sweeping the floor" tasks. If the technology tasks were the most important, we'd try to get everyone to work on them, even beginners. In reality, they are neither the most important, _nor the most difficult_, which is why it's easy for us writing code to go do them (and over-emphasize their importance).

We have done poorly in this regard with Rubinius. Which is why I'm highlighting it. These posts are in order of importance. By the last post, we'll see an interesting relationship between the parts covered in each post. On post three of five, I'm starting to talk about technical details. If we had done the first two parts better in Rubinius, you'd already be using the features I'm starting to talk about today.

## Compiling Ruby

In Rubinius, there is a parser that turns Ruby source code into a tree structure and a compiler that turns this tree into bytecode instructions. The "virtual machine" then interprets the instructions to run the Ruby program. So, Rubinius uses a bytecode interpreter to run Ruby.

In contrast, MRI 1.8 will parse Ruby source code into a tree and then walk the tree to execute the Ruby program. It does not convert the tree into bytecode instructions first. This illustrates that there is more than one way to execute code. The tree and the bytecode are called _intermediate representations_ because they come in between the source code and the execution of the program specified by the source code.

If you have Rubinius installed, you can see these intermediate representations. Let's look at the tree that results from a simple program:

    $ rbx compile -A -e 'puts "Hello" ", " "world"'
      Script
      @name: :__script__
      @pre_exe: []
      @file: "(snippet)"
      @body: \
        SendWithArguments
          @privately: true
          @vcall_style: false
          @line: 1
          @name: :puts
          @block: nil
          @arguments: \
            Arguments
              @line: 1
              @array: [
                StringLiteral [0]
                  @line: 1
                  @string: "Hello, world"
              ]
              @splat: nil
          @receiver: \
            Self
              @line: 1

If you look closely at the source code, you'll see that there are three Strings next to each other with no operator in between them. In the tree output, there is a single String. The Ruby parser concatenates two adjacent String literals. This knowledge may come in handy if you take a class from Sandi Metz.

The tree above is the abstract syntax tree (AST). It has all the details necessary to generate bytecode. Sometimes this representation is too verbose. There is another tree representation, called an _s-expression_, that is often used.

    $ rbx compile -S -e 'puts "Hello" ", " "world"'
    [:script, [:call, nil, :puts, [:arglist, [:str, "Hello, world"]]]]

If you squint at this and mentally replace square brackets with parentheses, colons and commas with nothing, you end up with something that may look familiar.

    (script (call nil :puts (arglist (str "Hello, world"))))

This looks almost exactly like Lisp and proves that Ruby really was based on Lisp like Matz has said. You might mention that to any Clojure programmers that are trying to convince you that their version of Lisp is better.

Joking aside, what it shows is that there is a deep structure shared by different programming languages. In Lisp, that structure is visible to begin with. In Ruby, we need tools to derive that representation for us. This is important to note because the next step we look at, bytecode, is merely another representation of this structure.

    $ rbx compile -B -e 'puts "Hello" ", " "world"'

    ============= :__script__ ==============
    Arguments:   0 required, 0 post, 0 total
    Arity:       0
    Locals:      0
    Stack size:  2
    Literals:    2: "Hello, world", :puts
    Lines to IP: 1: 0..10

    0000:  push_self
    0001:  push_literal               "Hello, world"
    0003:  string_dup
    0004:  allow_private
    0005:  send_stack                 :puts, 1
    0008:  pop
    0009:  push_true
    0010:  ret
    ----------------------------------------

These different representations are equivalent from the view of preserving the fundamental semantics, or _meaning_, of the program. So, why do we use these different forms?

The reason is, they provide different advantages. The AST is easy to understand and easy to process. Once the source code is parsed, the virtual machine can begin executing the program immediately. However, the AST takes up quite a bit of space and requires doing some things over and over. The effort to optimize the tree to remove unneeded steps can be complex. On the other hand, bytecode takes more time to generate but removes some redundant steps in the process. Further optimizing the bytecode can often be performed just by looking at a sequence of a few instructions, something called _peephole optimization_.

The main takeaway is that a Ruby program can have many representations. In the next post, we'll even look at a Ruby program represented by [LLVM](http://llvm.org) IR. Any one of these general classes of representations, like trees, may itself have multiple forms. For example, the object graph and the s-expression above are both forms of trees. In the same way, "bytecode" instructions can have different forms. But before we get to that, let's talk about the general design of an instruction set.

## The Instruction Set

There are two main problems that I've encountered in the design of an instruction set. I'll call them the _primitiveness fallacy_ and the _granularity fallacy_.

The primitiveness fallacy is the idea that the instructions must be primitive operations. If an instruction can be decomposed into other instructions, then it is not primitive enough. That seems to be the conventional wisdom. For example, addition of two machine integers is seen as a primitive operation, so there is an instruction for that. But searching for a particular byte in a sequence of bytes can be represented by more "primitive" operations like incrementing an index to reference the next character, and comparing the byte with the target the same way two integers are compared.

The primitiveness fallacy is really the manifestation of an arbitrary decision on what to include in an instruction set. The operations of the instruction set are not really primitive, or the instruction set would consist of just a [NAND gate](https://en.wikipedia.org/wiki/NAND_gate). Forcing the operations in an instruction set to be too simple carries the risk of losing important information about the program. In a _semantic_ sense, the information is not lost because the program executes correctly. However, there is another dimension to a program besides correctness, and that is performance. We'll return to that in a moment.

The second fallacy is the idea that all the instructions should have about the same granularity. If one instruction adds two integers and another instruction reads a file into a buffer, the "granularity" is significantly different. This may appear to be the primitive fallacy because this example, reading from a file, can be implemented with simpler instructions. However, reading from a file involves an operation that _cannot_ typically be performed by simpler instructions, namely, invoking a system call. System calls, in typical virtual machines, are handled separately from instructions, usually by a supporting library of functions. System calls are not the only illustration of the granularity fallacy, but they represent any need to access some functionality outside the program itself.

The problem with these fallacies is that they undermine the whole system. Primitive operations, all of about the same granularity, sound a lot like sand to me. However, the instruction set is the foundation of executing a program. Foundations are the most important part of a structure where the biggest stones are used, not sand. The same is true of intellectual systems like logic and math. If the foundation is insufficient or inconsistent, everything unravels.

A second problem is that they lead to something I call the _semantic cone_, an inverted cone that is larger at the top than at the bottom. At the top, there's the program source code. At the bottom, there are the virtual machine instructions. The shape of the code (larger at top than at bottom) represents the loss of information as the source code is processed into instructions.

There is a saying, "First make it correct, then make it fast". This is good advice. It highlights that a program can have many forms, but to be useful, they need to be correct. So we start with a correct form, then transform it to one that is still correct but performs better. It's this second step that suffers in the context of a semantic cone. It's possible to make the program correct, but the amount of information lost makes it very difficult to make the program fast.

To address this limitation, we can use something like a just-in-time (JIT) compiler. However, to make the JIT work well, we have to _invert the semantic cone_. That is, we have to recreate the information that we lost going through the cone so that we can use that information to make the program fast. That seems wasteful. What if we just _don't_ throw away the information to start with? Great question!

My answer to this is that we need to convert the semantic cone into a semantic column to the greatest extent possible. The way to do that is to have instructions that represent information that we normally throw away. This brings us to the new Rubinius instruction set.

## Kinds of Instructions

In Rubinius, every _executable context_, every script body, class or module body, block, or method, is represented by a CompiledCode instance. Every compiled code object has a separate executor, or "function" that executes that compiled code's "instructions". These instructions may be bytecode, or they may be machine code generated by the JIT compiler.

### Stack Instructions

Stack instructions are operations that remove one or more values from a special structure, the stack, then compute a new value and push it back onto the stack. Stack instructions are illustrated in the bytecode example above.

The advantage of stack instructions is that they are easy to generate and fairly easy to reason about. In fact, there are languages that are essentially just stack instructions, like [Forth](http://en.wikipedia.org/wiki/Forth_%28programming_language%29), [Factor](http://en.wikipedia.org/wiki/Factor_%28programming_language%29), and [PostScript](http://en.wikipedia.org/wiki/PostScript). If you have ever used an RPN calculator, you have written a stack program.

The downside to stack instructions is that they can be somewhat difficult to optimize. The stack itself obscures the relationship between operations.

The current Rubinius instruction set is stack-based. In Rubinius 3.0, we will retain and refine the stack instructions. This enables us to develop the new instructions incrementally while continuing to compile to the stack instructions.

### Register Instructions

When computers were first invented, there was a law passed that required every real or virtual machine to have only one kind of instruction. Except there really was no such law passed. We just act like there was.

Every major virtual machine that I know of basically chooses either stack-based or register-based instruction sets. You may have heard that Lua or mruby uses a "register-based virtual machine". Of course, whenever two things are different, our natural tendency is to rank them. If they are different, one must be better than the other. So, we may hear that register machines are "faster".

As noted above, there are advantages and disadvantages of stack-based instructions. There are different advantages and disadvantages of register-based instructions. In some cases, the advantages outweigh the disadvantages. Since Rubinius can use a different function to execute every compiled code object, we can use stack-based instructions for one and register-based for others. This enables us to benefit from the advantages while limiting the disadvantages.

Rubinius 3.0 adds register-based instructions to the bytecode. In fact, there's nothing preventing us from mixing register and stack instructions in the same method.

### Parsing Instructions

In my experience, 95% of the time that someone wants to write a C-extension and escape Ruby code, they are either parsing some text, generating some text, or both. Parsing is basically decomposing text into some structure, and templating is basically composing some structure into text.

Parsing is such a common and essential part of programs that we should have special support for it. Rubinius 3.0 adds support for parsing instructions modeled after [LPEG](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html), which is an implementation of a parsing machine in Lua for parsing-expression grammars (PEGs).

There are many nice properties to PEGs, perhaps the most interesting being composability. This enables building up more complex grammars from simpler pieces. For instance, what if parsing dates were part of the base language and you wanted to create a special-purpose language, perhaps for parsing a simple config file format? You could compose the base language's date parsing with other parts of your language.

Adding parsing instructions also enables Rubinius to read pre-compiled bytecode describing parsing operations and execute them. This is extremely useful when bootstrapping, where we need to parse Ruby code to execute it, but we need to execute Ruby code to be able to parse it.

### Assertion Instructions

Assertion instructions describe constraints for computed values without _changing the computation itself in any way_. However, the assertion instructions may change the way the program runs, depending on configuration. The assertion could raise an exception, which would abort the program if not handled. Or it could just log the location and values that failed the constraint validation.

Tests for code are important. But tests are usually only representative. If you have a test for how many cats can play in the same room, you don't typically write tests for 0, 1, 2, 3, ..., N cats. This is especially true if the range of values is huge.

Usually, it's possible to describe, as very simple predicates, the constraints on a value. It should be greater than zero. Or it should be between 200 and 5 million. Assertion instructions can _optionally_ be executed when running tests or in production to check that values conform to constraints. In this way, defects can more readily be pinpointed.

### Instrumentation Instructions

Instrumentation instructions enable analysis and monitoring of code. Like assertion instructions, they do not change the semantics of code in any way.

The design of the instrumentation instructions was influenced by a paper titled, "The JVM is Not Observable Enough (and What To Do About It)". [1] The authors had implemented a framework for instrumenting Java bytecode. This is a common approach for program analysis. They discovered serious problems with this approach, including causing the Java virtual machine to deadlock or even segfault.

An even more influential paper for me was, "Hidden in Plain Sight". [2] The authors of DTrace described the constraints they were working under. They wanted DTrace to be used in production, so it was essential that it had no performance impact if not in use, it had to be absolutely safe for production, and it needed to collect only the relevant data so that it would operate well.

The instrumentation instructions enable Rubinius to build powerful tools that can analyze and monitor production code with the guarantee that the semantics do not change. This is especially important in a regulatory environment where code changes must be strictly controlled.

## Other Machines

That was a fast and high-level introduction to the new Rubinius 3.0 instruction set. I'll be writing in much greater detail about all parts of this in the coming weeks.

One important aspect of the new instruction set is the attempt to completely describe the language semantics at the instruction set itself. This means *not* using supporting functions in a separate place. This enables us to use all the existing tools for compiling Ruby, and then run the resulting program on anything that provides these instructions.

For example, some people are very interested in Rust and have asked if we're going to rewrite the virtual machine in Rust. We have no plans to do so, but if someone is interested, all they would need to do is implement the instruction set. The same goes for Haskell or Go or even [asm.js](http://asmjs.org   /).

The possibilities here are pretty exciting. The Ruby language itself is not the most important piece, as we'll see in the last post of this series.

## Acknowledgments

<em>I want to thank the following people: Chad Slaughter for entertaining endless conversations about how we build software and challenging my ideas.  Yehuda Katz for bringing us many nice ideas. Joe Mastey and Gerlando Piro for review and feedback, some of it on these topics going back more than a year.  The Rubinius Team, Sophia, Jesse, Valerie, Stacy, and Yorick, for putting up with my last-minute requests.</em>

[1]: http://www.cl.cam.ac.uk/~srk31/research/papers/kell12jvm-preprint.pdf "The JVM is Not Observable Enough (and What To Do About It)"
[2]: http://queue.acm.org/detail.cfm?id=1117401 "Hidden in Plain Sight"
