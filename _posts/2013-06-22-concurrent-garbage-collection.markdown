---
layout: post
title: Concurrent garbage collection
author: Dirkjan Bussink
---

Just a few days ago my work on making the Rubinius garbage collector more
concurrent has landed in Rubinius master. In this post I'll describe how
this work was done and what the ideas behind it and pitfalls encountered
are. Hopefully after reading this post, you'll better understand what
concurrent garbage collection means for your Ruby programs and how they
operate. Note that I have explicitly chosen to keep things like benchmark
and performance numbers out of this post - it is long enough as is.

### In the beginning there were long pauses

Garbage Collection pauses: anyone with a somewhat complex app usually
knows about them. They often result in wildly varying performance of
your web requests and cause performance issues in unexpected
situations. There has been a lot of effort put towards working around
these issues, such as out of band garbage collection done with servers
such as Unicorn and Passenger. Honestly, I think these techniques are
very useful but in essence still workarounds because of a deeper
problem in MRI.

With this work on concurrent garbage collection, long pauses are a
thing of the past in Rubinius. You can run a single process application
server and see stable and fast performance in highly concurrent scenarios.

So for this post I'll first introduce some fundamental concepts that we
needed to make explicit in Rubinius to support concurrent garbage
collection. After that I'll discuss some of the issues encountered when
stress testing and putting Rubinius under load with applications such as
Rails and Sidekiq. Often there will be a reference to a commit, this is
because commits won't change, but code can. This makes sure this is still
correct and relevant even when things are changing in the future.

### The tri-color invariant

For people having some knowledge about garbage collection theory, the
term tri-color invariant probably sounds familiar. It is a term that
describes a property of the system that is important in being able to
garbage collect properly without for example cleaning up used data.

The tri-color invariant defines three different states for an object:
white, grey and black. Each of these states describes the state of an
object during a garbage collection phase.

**White objects** are objects that the garbage collector hasn't seen yet
and doesn't know about yet. It might be found in the future, or it might
not if it's actually garbage and not a reachable object.

**Grey objects** are objects that the garbage
collector has seen, but hasn't completely handled yet. By handling I
mean that this object has not been completely scanned for references to
other objects. This means for example that we haven't visited the class
or instance variables table in Rubinius.

**Black objects** have been seen by the garbage collector
and also has been scanned. This means that this object is handled and
doesn't need to be revisited again during the current phase.

Garbage collection works in different phases. The first phase is to
start and mark objects known as roots. Roots are objects that we define
as always reachable and who should never be cleaned up. One example
of these in Rubinius are the built-in classes. We never want to garbage collect
Module, Class or Object. Another group of root objects are the objects
currently on the stack when we garbage collect. These might be used still
after the garbage collection finishes and the application continues to run.

When we start the garbage collection cycle, we make the roots grey. This
is done in Rubinius by marking them and adding them to a list of objects
that are going to be scanned in the future. This allows us to do garbage
collection without having to use recursion here, which could lead to very
deep call stacks and potential stack overflows.

When we have done this, we start handling the so called mark stack. We
pop off an object and scan it. This makes the object implicitly black,
because it is marked and no longer in the mark stack. It is important to
realize that the invariant colors are not always explicit states, but
sometimes implied by the total system state. They are a tool for
reasoning about garbage collection, not a design for how you must write
an algorithm.

During the scanning of an object we might encounter new ("white") objects.
We mark them and add them to the mark stack as well, thus making them grey.
This process of handling the mark stack continues until the entire mark
stack is empty. At that point we know that all the reachable objects are
now black and the remaining objects can be cleaned up.

From these steps you can see the following invariant:

> "A black object never points to a white object, but always only to grey or other black objects."

This invariant is important because if it were to be violated, we would clean
up a white object if it would be never marked. But that would mean the
black object would refer to garbage instead of a valid object.

### Tri-color invariant and concurrency

Maintaining the tri-color invariant is important for correctness. If you
look at the reasoning in the previous section, you might already realize
where violation of the invariant might happen in a concurrent scenario.

The simplest example is the following. When we start to handle the mark
stack, we scan objects and make them implicitly black. Now imagine the
case where our code (that still runs during GC) writes an unseen white
object into that already scanned black object. In this case we can't
guarantee the tri-color invariant because our application might change
things behind our back without the garbage collector knowing about it.

So the question is, what would be a solution for this problem? Well, the
obvious thing would be to make sure we run some additional checks when
we encounter the scenario of a white object being writing into a black
object. This means, however, that we have to make sure we can catch all
these cases where this happens. What if somewhere in the virtual machine
we would assign a variable and not run this code? It would mean breaking
the invariant and that leads to memory corruption down the road.

This situation represents a triumph in the history of Rubinius engineering -
because the VM already had a Generational GC and no global interpreter
lock, the work that went toward making the GC concurrent was much simpler.
Let's try to understand why.

### The write barrier

In generational collection, we have a problem that is somewhat similar
to the tri-color invariant problem here under concurrent garbage
collection. Generational garbage collection stems from the "weak generational
hypothesis" which states that "objects tend to die young." One
obvious example of this can be found in Rails. The objects loaded in
your Rails app often consist of two classes of objects.

The first class of objects are all the objects that define your app. For
example the class definitions of your controllers and models, but also
templates for your views. These objects stick around for the entire
lifetime of your application. The second class of objects are the ones
allocated when you handle a request. They only live for the lifetime
of the request, so if you would only garbage collect those objects after
a number of requests, you would already prevent any memory growth.

This is a very simple example of the generational hypothesis and why
generational garbage collection works well on workloads like Rails. But
I said that this suffers from a similar problem as the tri-color
invariant violation, so what is that problem?

The problem occurs if you cross the barrier between young and old
generations. For example you store a new object into a long existing
array. That array is part of the mature generation. What you don't want
to happen is that when we clean up the young generation, we miss this.
If we would, the array would suddenly contain garbage and no longer a
valid object. This means that also for this problem we need to run an
additional check when writing an object into another object. 

The code running these checks is called a write barrier, because it's a
piece of code that is being run on each write of object into another
object. What exactly this code is depends on the use
case. We have already described two of them now, one for generational
and one for concurrent garbage collection.

### The remember set

So we've just discussed the write barrier. It also stated that we
already have a write barrier for generational collection. For some of
the issues mentioned later on, the generational concerns are also of
significance, so it's useful to also explain the concept of the remember
set.

The remember set is a set of objects that is used to scan for young
objects during a generational collection. This is needed for the cases
where we store a reference to a young object into a mature object. What
we do is store the mature object in the remember set, so we can scan it
when we do a young collection. This makes sure we follow all the paths
which through a young object is reachable.

### Changing the write barrier for concurrent GC

So we need to be able to run some code when we are writing a white
object into a black one during generational garbage collection. Because
we already have a write barrier, we should have a single place where we
can change this.

Well, that was not entirely true. As part of implementing the concurrent
garbage collection solution, I also cleaned up some of the internals to
unify for example our usage of write barriers. It wasn't a terribly
daunting task, since the logic was already there, just that we had the
logic in more than one place. If you want to take a look at what this
work was, I'd suggest taking a look at the diff here:

[Unify write barrier access](https://github.com/rubinius/rubinius/commit/c426e7bdfea54a742fc598d104a5fd756db27a6a "Unify write barrier access")

If you read through the commit, you can see it actually removes more
code than it adds, because it cleans up and unifies the usage of the
write barrier.

So this brings us to the next point of how to change the write barrier.
What we need to do is to make sure we guard the case where we store a
white object into a black object. If we detect this scenario, there are
basically two possible strategies. The first one is the make the white
object grey by marking it. This means we will scan it in the future. The
other option is to make the black object grey again, so we scan that
again in the future.

The first solution has the advantage of moving the collection forward,
not backward like the second solution. It does however have the downside
that it could keep objects alive longer than the second solution would.
In the implementation for Rubinius we've chosen the first option,
mainly because it proved to be the simplest to implement and it moves
the so-called wavefront forward.

The other thing we need for this check is to know whether an object is white,
grey or black. As you might remember, I've said before that these states
might not be explicitly modeled. This was actually the case in
Rubinius. There was a way to determine that an object was black, namely
when it was marked but not in the mark stack anymore. The first check is
easy and cheap, but the second check isn't. We would have to scan the
entire mark stack for an object if we want to write another one into it.
This would quickly become really expensive and isn't a good solution.

### Introducing a new garbage collection state

The only way to really tackle this problem is to introduce a new state
for an object, so we can easily see that it's already scanned, not only
just marked.

For this I also have to take a sidestep and explain another concept,
namely the rotating mark concept. If you look at how to track the state
during the mark phase, there are a few solutions. The easiest one is to
have a single bit to identify whether we've marked an object or not.
This comes with a downside though, that we have to make sure we set all
the marks back to 0 before we start a garbage collection, or at the end
of a garbage collection so it's 0 for the next cycle.

To prevent this overhead, there is a concept called a rotating mark.
It's actually very simple, instead of having a mark of 0 or 1, we have
0, 1 and 2. When an object is new, it still has a mark of 0. The first
time when we garbage collect, we use 1 as the mark. This means we can
clean up all the objects afterwards which don't have mark 1.

The second time we swap the mark to 2 and do the same thing. We can
remove anything with a mark that is not 2 after the collection cycle.
This is a concept that the garbage collection in Rubinius already used
before these changes. So what we did was extend the mark bits to also
include the scanned state. Rubinius uses the object header to store
this information inside the object and we had a few more bits of room
to store additional information.

We could have solved this basically in two ways, one would be to just
use a single bit as the scanned state. This would work fine, but it
would make updating the header with this bit more expensive in certain
concurrent scenarios, where we would end up doing a compare and swap
operation twice instead of once.

Therefore we opted for merging it with the mark bits. This means that
instead of 2 bits to store 0, 1 or 2, we now use 3 bits to store 0, 2,
3, 4 or 5. Why is 1 not used you might ask? Well, that is a side effect
of the combination of a rotating mark and the scanned state.

The scanned state is actually represented in the last bit. So a value of
1 would mean the object is scanned, but not marked. Since we always
first mark an object before scanning, this scenario can't happen. The
values 2 and 4 mean the object is only marked, the values 3 and 5 mean
the object is marked and scanned. These values also make the operations
for checking if a mark is set and setting the scan state simple.

You can find the code that introduces this new state in the following
commit:

[Introduce explicit scanned state for mature GC](https://github.com/rubinius/rubinius/commit/d1d99d6048a01f1687080f98baa904d4562ed920 "Introduce explicit scanned state for mature GC")

### A new version of the write barrier

So with this new addition, we actually have the tools at hand to change
the write barrier so we can handle this new case for the concurrent
garbage collection. So besides checking the generations of the objects,
we now also have to check the newly introduced scanned state, which
represents the black state in the tri-color invariant. If the object we
write into is already black and the object to be written is white, we
store this object in a separate set.

This set is then in the finalization phase of the concurrent garbage
collector used. How this works exactly will be explained later, for now
it's important to remember that objects in this set will be marked in
the future.

The new version of the write barrier can be found here in this commit:

[Update write barrier with checking for scanned objects](https://github.com/rubinius/rubinius/commit/bcecb3103d450502bd43a3411461381f5452a7e0 "Update write barrier with checking for scanned objects")

As you see, it also incorporates more changes, mainly the addition of
the before mentioned set and changes to the JIT. The changes to the JIT
are necessary because the JIT basically emits the assembly code for the
write barrier directly, so it also needs to emit the new version of the
code.

There is one thing here that might strike some as surprising. That is
the fact that we actually set the scanned state before scanning the
object. This is not a bug, but deliberate, since otherwise there is a
race condition possible. The race condition would be that a scan of on
object is in progress while another thread runs the write barrier for
that same object. In that case it could see the object as not scanned,
and not store the new reference. The other thread would then mark the
object as scanned, but it would still have a white object stored. This
would violate the tri-color invariant and cause corruption.

By setting the scanned state as the first operation, the only risk is
perhaps adding an element unnecessary due to a race condition, but this
isn't problematic but just a minor increment in the amount of work.
Since this only happens with this race condition, this case is so small
it doesn't cause any performance issues.

### The actual concurrent garbage collection

So with these pieces put into place, I was kind of surprised. It only
had taken very little work to add these new concepts, a testament I
think to the design that Evan and Brian originally started. Most of you
probably know the feeling where you start to implement a seemingly
daunting feature, but after starting it everything falls into place so
easily you feel something must be wrong. Of course at this point it did
take me longer to finish the work, but luckily not because of
fundamental problems with the approach, but just with bugs for certain
edge cases.

So with this initial surprise out of the way, let's see how the basic
algorithm works. The first thing that is important to realize, is that
this is a mostly concurrent garbage collection, so it's not 100%
concurrent. There are still stop the world pauses, but they are much
smaller than with a complete stop the world collection.

These stop the world pauses are needed to get a consistent view of the
roots to start collection from and to finish it all up to get another
consistent snapshot of the system. So what we do is trade one big stop
the world pause for two much smaller ones.

During the first stop the world, we do the same thing we did before to
setup the initial state to start collection with. After this is done, we
signal all the threads that they can continue to do their work, while a
separate thread will start the mark phase. This mark phase will happen
concurrently with the other threads running. When the mark thread has
marked everything in the mark stack, we request the second stop the
world phase. In this second phase we rescan all the roots, because they
might have changed. This means also rescanning the thread state so we
can see what is on the stack at that point.

After we do this, we also schedule the special set that the write
barrier has tracked during execution. We add that as well to the mark
stack. We then scan the remaining mark stack during this stop the world
phase. This is in general a much smaller stack, because we already
scanned the majority of the system.

After we've finished scanning the mark stack, we do the same
finalization as we did before in the stop the world collector. We have
to handle things like finalizers, C-API handles etc. and also sweep up
the garbage. If this final phase has finished, we can continue running
everything and garbage collection for this iteration is done.

The commit introducing this new thread can be found in:

[Introducing concurrent mature mark phase](https://github.com/rubinius/rubinius/commit/22c8c9c72e35be71d4258f00e57d8e9bb1f91a2f "Introducing concurrent mature mark phase")

Most of the changes are for updating the gathering of statistics. We of
course want to know how much time is spent in the stop the world pauses
and the concurrent mark phase, so we can easily see the benefits of this
strategy. Another large amount of the code is actually dedicated to
supporting another feature, which is described in the next section.

### Running young collections while concurrently marking

One thing that is of concern when doing a concurrent garbage
collection of the whole program is what happens to the young
generation. As explained earlier, Rubinius has a generational garbage
collector, so we run different styles of collection at different times.
Only when a young collection can't satisfy memory release, or we have
allocated a lot in mature space, we trigger a full concurrent
collection.

The easiest way to prevent this issue is of course preventing a young
collection from happening while we're concurrently marking. This however,
introduces a significant problem. The problem is that when this happens,
any allocation that would normally create a young object can't succeed.
This means those objects would be allocated as mature objects,
increasing the memory pressure for the total system.

In a system that for example runs Rails with reasonably complex pages,
it's not that strange a situation that we would want to run multiple
young collections during the runtime of the concurrent mark phase. If we
don't solve this problem, this would mean megabytes more of mature
objects being allocated.

The young collection still happens in a stop the world phase. This stop
is not that problematic, since it only takes a short time to collect the
young space. This is because it only copies over live objects and it's
limited in size. This means that collection times are usually in the
order of a few milliseconds. 

So we can use the fact that we stop the world in this cases to make sure
we update all the structures at that point for the concurrent mark
phase. The biggest thing here is that we need to update the mark stack
and other similar structures, such as the set of weak ref objects and
the finalizers.

Since the concurrent mark phase marks all objects, it also marks young
objects. It needs to see the copied objects if they are still alive.
This also applies to the set that is tracked by the write barrier.

### Inflated headers and code resources

Another solution to the problem of writing white objects into black ones
is to always make all objects grey if they are allocated during
a concurrent garbage collection cycle. This approach is similar to how
we solved problems with other managed resources that aren't Ruby
objects.

There are two types of those objects worth mentioning, inflated headers
and code resources. The first one is a place where we store information
if it doesn't fit in a normal object header, or when it requires non
moving memory. This last required applies for example to using an object
as a Mutex, which is possible with Rubinius much like how the JVM also
works. We can't just move memory if an object is locked, so we keep that
information separate.

The inflated headers also keep a mark. This is done so we can clean them
up after a garbage collection cycle and deallocate space for objects
that are not longer alive. This means we now have two marks which
implies a possible race condition. What if an object is marked by one
thread and concurrently inflated by another? We would need to make sure
we don't lose information in that case about the inflated header mark.

Since we don't inflate objects often, we choose a very conservative
approach here. What that means is that if an inflated header is
allocated, it always gets the current mark set as it's mark. It means
that the inflated will possibly not be reclaimed during the first
garbage collection if the object is already out of reach. We don't view
this as a problem, since inflated headers often are allocated for
objects that will stick around longer, for example because they are used
as a lock.

The second case is code resources. Code resources are things like code
compiled into the virtual machine representations of bytecode, or native
code created for jitted functions.

Here we use the same approach: always allocate them with the
current mark. Code resources are also very likely to stay around, since
often code gets executed more than once. This approach here is also the
simplest for fixing issues related to this problem.

These changes where made in the following two commits:

[We always create a code resource as marked initially](https://github.com/rubinius/rubinius/commit/c41e3a66ff476864e5996ca0a536e1644e745c66 "We always create a code resource as marked initially")

[Initialize inflated header marks with current mark](https://github.com/rubinius/rubinius/commit/76039182e837d2e1460e008f2d11515d27e499cc "Initialize inflated header marks with current mark")

### Fibers and how they affect stack scanning

Fibers are somewhat special. They have a part that works like a normal
Ruby object, for example where we store the return value of a fiber. The
other parts of a fiber is the actual stack that is executing.

In the past we tracked these two things separately, but that wasn't
actually needed. The problem is that with concurrent collection, we have
to separate them again. This is needed because we can't concurrently
scan the stack parts of a fiber, since those stacks could be executing
and become invalid at any moment.

This means we have to separate the handling of the stacks from the
normal object. We can then scan all the stacks in the final collection
phase for all the fibers that are marked and thus reachable at that
point. If we don't do this, applications like Sidekiq that heavily use
Fibers will crash almost instantly on the first garbage collection
cycle.

This change was made in commit:

[Return of tracking the fiber data explicitly](https://github.com/rubinius/rubinius/commit/b3579cfd46fb773b563e0f66154187dc32f31909 "Return of tracking the fiber data explicitly")

### The first more subtle bug

The additional of the concurrent mark thread is also where the first
subtle bug was introduced and subsequently fixed. As in the previous
section described, we have to update the mark stack during a young
collection cycle.

During a young collection cycle, objects can be promoted. This happens
when objects are alive for a certain number of young collections and are
moved to the mature space. During this time, the allocation mechanism
also changes and the object gets allocated in the area where we sweep
after a concurrent mark phase is complete.

The bug only happened when a young object was promoted that was already
marked in the concurrent mark phase. This means the object header
already had the mark set. The tricky thing here is that the mature
memory space also as a separate mark table. This is part of the
Immix algorithm used.

What would happen if a marked object was promoted is that the mark was
retained. This means however, that the underlying Immix memory space was
not marked. This means that during the sweep phase the memory was
incorrectly seen as not in use and reclaimed. This leads to memory
corruption because now a piece of memory is used for two different
objects.

The solution to this problem is actually very simple. The only thing we
have to do is when an object is promoted, is to remove the mark. This
makes sure that when this object is stored into some other object, it
would go through the write barrier and will be scheduled for proper
marking in the future. This will result in the underlying memory to be
also marked and the problem is gone.

[Introducing concurrent mature mark phase](https://github.com/rubinius/rubinius/commit/22c8c9c72e35be71d4258f00e57d8e9bb1f91a2f#L14R488 "Introducing concurrent mature mark phase")

### Concurrency, forking and deadlock hell

One of the most difficult features to get working correctly in a
concurrent virtual machine like Rubinius is forking. I can perfectly
understand the reason for the JVM not supporting this, besides it not
being cross platform. It requires careful orchestration of all the threads
to prevent the child process from ending up in a problematic state.

Of course, since we added a new thread for concurrent marking, this also
introduced a deadlock that could be triggered by forking easily. This
was evident in a Sidekiq app that relied on forking to spawn subprocesses
with the fork and exec pattern.

The problem here was that we still had the lock that is used to
safeguard the internals of the marker thread, while also waiting for a
stop the world pause. This stop the world pause would then be triggered
by a `fork()`, which then also requests all the auxiliary threads in the
system to pause at a safe point.

To get to this safe point, the forking thread would try to grab the lock
of the Immix thread, which it couldn't because the thread itself still
had the lock. The fix is similar to how other auxiliary threads such as
the signal thread and finalizer thread work, by releasing their own lock
just before they mark themselves as being dependent on the garbage
collector again.

This problem was fixed in commit:

[Fix deadlock when forking when immix thread is active](https://github.com/rubinius/rubinius/commit/e74d1fa0756e3f52f3535196a8106e3d29e0c34a "Fix deadlock when forking when immix thread is active")

### Finalizers are tricky to get right

This last found and fixed bug was the most elusive one. It was really
hard to trigger and only happened rarely, but luckily often enough so
that finding it wasn't completely impossible. As always with these bugs,
reproducing the bug is really 50% of all the work. 49% is finding what
causes it and 1% of the time is actually spent fixing it. Remember
though, that 80% of all statistics are made up on the spot.

Here again the intricate play between different threads and the timing
dependency of execution caused the issue to only appear rarely. The
easiest way to reproduce it, was to run a Sidekiq test application that
cpuguy83 graciously provided for finding another bug.

After running at full speed for maybe somewhere in between 5 and 10
minutes it would crash. Such long feedback cycles can be very
frustrating and make digging into it slow. However, the rewarding
feeling and actually providing people with a stable Ruby runtime is
definitely worth the hassle.

So what was at play here? First of all, it is important to know about
finalizers. Finalizers are functions you can register that run when the
garbage collector determines an object has gone out of scope. Inside
Rubinius we use this for a number of classes, such as for IO to close it
if it hasn't been and for Fiber's to clean up the stack space they might
have allocated.

This last category of object is actually what triggered the problem in
Rubinius. It looked like a fiber during the finalization had references
to no longer valid young objects. These young objects would for example
be the value that a fiber stores as the return value. So the question is
what was causing these invalid objects? It looked like the fiber was no
longer properly scanned during young collections and didn't see any
updated value.

That last sentence might seem easy and straightforward, but it took a
few hours to actually consciously realize the implications of it. I
sometimes call it "A-ha driven development." The point where you just poke
at code, trying to see through it in a concurrent scenario, until the
case where it can go wrong pops into your head. I haven't been able to
really identify the though processes going into it, but it includes a
lot of poking around and pondering. 

Actually often in cases like that I just go and take a walk to ponder
things, or if it's late just go to bed. I've literally had eureka
moments the next morning in the shower, realizing what the problem was.
If anyone has insights or ideas on how to improve this process, I really
welcome them. I'd love to get a better handle on it to see if it can be
made more consistent in some way.

So let's go back to the original sentence that is much more important
than it seems. It stated that the problem occurred if a mature fiber
object would not have it's young variables updated. This should makes us
remember the concept of the write barrier. As explained we use that to
make sure that if we add a young object to a mature object, we register
that. What happens in that case is that the mature object is stored in a
remember set, that is always scanned during a young collection. This
ensures these values get updated properly.

So what was happening here? After adding some debug logic (yes, just
lots and lots of printf statements), I could see that the Fiber was no longer in
the remember set. But it should be, since it had young objects
referenced to it!

So the question is why is it removed from the remember set. Normally if
a young object is collected and stored back into the mature object, the
write barrier gets executed and the object is added to the remember set
that is used in the next cycle. Why doesn't this happen in this case? So
after looking for where we active remove objects from the remember set,
I found this actually happens during finalization. After an object is
finalized, we removed it from the remember set since it couldn't be
referenced anywhere anymore.

This reasoning was perfectly sound until we introduced concurrent
garbage collection. Because that suddenly introduces a new place that
the object might still be referenced from, namely the current mark
stack! This means that this goes wrong in the following scenario. The
fiber is in the concurrent mark stack, which could happen because for
example it was still active when concurrent collection started. Then the
fiber goes out of scope and during a young collection it is scheduled
for finalization. Because finalization has to keep objects alive, it has
to keep the fiber live. This would then cause the fiber to be promoted
and to be a mature object. The concurrent mark stack would be updated
with the reference to this now mature object. This would also result in
the now mature fiber to be added to the remember set.

So this sets the stage for the bug, what would happen next is that the
finalizer would run and after it completed, it would removed the mature
fiber from the remember set. Meanwhile, before we actually reach the
fiber in the concurrent mark phase, another young collection runs. This
young collection then moves the object inside the fiber, for example the
return value. Now finally the concurrent mark stack reaches the fiber
and crashes, because it sees an invalid object as the fiber's return value.

So how do we fix this? Well, the fix is actually simple, we just don't
remove a finalized object from the remember set. This means it still is
scanned during young collections. The downside is that more objects
might end up being promoted than necessary, but that's better than
completely crashing and breaking the guarantees a garbage collector
should give.

There is room to improve here for the future though, we could check if a
mature collection is in progress and only then not remove the object
from the remember set. Since starting a mature mark phase always
happens in a stop the world phase, this check shouldn't be problematic
due to race conditions.

You can find the actual fix for this bug in this commit:

[Don't remove finalized objects from remember set](https://github.com/rubinius/rubinius/commit/b1c65d7e8d90e5ad3d1f63c8dc2c2f1f3e0508d6 "Don't remove finalized objects from remember set")

### Unknown unknowns

For now, these are the issues and problems encountered during the
implementation of the concurrent garbage collector. I'm sure there will
be more garbage collection related bugs in the future, although I think
they will often be bugs that also are present for our non-concurrent
collection. Of course there were also issues like just wrong syntax that would cause
C++ compile errors etc, but those are of course not that interesting
to talk about and part of the normal development progress.

I've stress tested the concurrent collector under different load
patterns, like running a concurrent Rails application and Sidekiq test
applications. Those systems are often pretty good in finding concurrency
bugs and race conditions and since those are stable, I'm pretty
confident that this is ready for more broad usage. This is of course
also why it was merged into master.

I've tried to highlight the tricky and more complex issues here, so you
hopefully have a better insight into what implementing all this means.

### Resources

If you're interested in more background on the things discussed here,
these are some pointers to more resources. As a general reference work,
there is the Garbage Collection Handbook. So far the best book on
garbage collection I've seen with a lot of clearly explained content.
It provides a really good starting point to learn about garbage collection
and contains a lot of references to papers that can be read for even
digging deeper into the subject matter at hand.

[The Garbage Collection Handbook website](http://gchandbook.org)

[The Garbage Collection Handbook on Amazon](http://www.amazon.com/gp/product/1420082795)

The original paper in the Immix garbage collector, which Rubinius uses.
Note that our version of Immix isn't compacting, something that made concurrent
collection possible. In the future this is something we want to revisit
and improve upon.

[Immix: A Mark-Region Garbage Collector with Space Efficiency, Fast Collection, and Mutator Performance](http://users.cecs.anu.edu.au/~steveb/pubs/papers/immix-pldi-2008.pdf)

For finding more resources on garbage collection, you can also check out
this bibliography of garbage collection related papers:

[the Garbage Collection Bibliography](http://www.cs.kent.ac.uk/people/staff/rej/gcbib/)

I would also like to thank Michael R. Bernstein for reviewing this post.
He's been writing interesting blog posts on garbage collection and gave
a presentation at GoRuCo 2013. These articles can be really useful if
you want to have a starting point for the basic garbage collection
concepts that this article assumes you are somewhat familiar with.

[Adventures in Garbage Collection Pedagogy and an Introduction to Racket](http://michaelrbernste.in/2013/05/20/adventures-in-GC-pedagogy.html)

[To Know A Garbage Collector: GoRuCo 2013](http://michaelrbernste.in/2013/06/10/to-know-a-garbage-collector-goruco-2013.html)
