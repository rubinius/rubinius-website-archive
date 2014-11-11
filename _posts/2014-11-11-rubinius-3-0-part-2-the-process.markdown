---
layout: post
title: "Rubinius 3.0 - Part 2: The Process"
author: Brian Shirai
---

In this post, I'll talk about the release process for Rubinius 3.0. We want you to be using the new Rubinius 3.0 features as soon as possible. To explain our approach, I'll first talk about releasing software. I'm spending 20% of this week's posts on the release process because it's one of the hardest things we've struggled with.

Over the past about 12 months, we released Rubinius 15 times. It was less than the one release per week that I was aiming for. However, in the two years previous to that, we did not release Rubinius a single time. It's not that we didn't do tons of work; we did! In fact, the number of commits per year hasn't varied that much. But if we don't release, it's as if the features don't exist. Thus, how we release has a big impact.

## Conventional Wisdom

Why did we wait so long to release Rubinius 2.0? It's simple, we put too much into it! The reason we did that was concern for quality, not to delay release. However, our decision when to release was flawed. We were aiming for broad feature coverage at a point in time instead of optimizing for how quickly we could roll out features to you. To understand why, we need to look at what our priority should have been and why it wasn't that.

Our priority is to have an _impact_, providing value to people using Rubinius. We did not have our priorities straight, and this is why: we did not prioritize for people using Rubinius. Instead, we were persuaded by the following convention wisdom that was often offered as advice and is completely wrong:

> If it doesn't work when someone tries it, they may wait a long time before trying it again, if ever.

There are several problems with this fallacy. It confuses local and global effects. For every one person for whom a feature did not work, there are an unknown number of people for whom some feature did work. The latter group may have a much bigger global effect than the one person. It also does not account for the cost of people waiting for features. Finally, it's rooted in fear. The best antidote to fear is facts. If engagement is important, we should measure engagement and take steps to improve it.

Now that we know what to *not* do, what should we do? To understand that, we need to question how we release software.

## Software Release

How we release software is heavily influenced by how we build software, including our _definition of working software_. It is also influenced by the _relative cost to release it_. And finally, by _social factors_ related to adoption of changes in general.

I see three fallacies in how we release software: we model software as being mechanical, we act like releasing it costs a lot, and we think individual pieces of software should be reliable instead of building systems to be reliable.

### Software Is Mechanical

It seems that since software is technology, then it must be mechanical. Of course, this is not a new realization. We spend a lot of time arguing whether we should apply manufacturing or construction analogies and methods to creating software. However, software is not primarily mechanical.

In fact, software is much more like biological systems than mechanical systems. We needlessly impose mechanical limitations on software. In software, we can build something figuratively in mid-air, with neither support nor suspension. It's a choice we made to model software on mechanical processes. We can choose differently.

The fact that features are constantly being developed does not mean that a system must be chaotic. Children do not wait till they are full-grown to chew on things. And while they are chewing on things with baby teeth, their adult teeth are growing. Some transitions (like loosing teeth) can be disruptive, but for the most part, they get along fine even as their abilities are constantly developing. Why don't we choose biology as a model for creating software?

As opposed to mechanical systems, biological systems are *very well adapted* to functioning as a whole while changing or growing. Consider the difference between a plant growing from a seed and building a car. It's only at the very end of the manufacturing process that a car is able to function as a whole. Meanwhile, the fundamental metabolic processes in a plant function from the very beginning. The plant changes as it grows, so not every part is functional at the start. The key is to decide which parts should function first.

Another important aspect of biological systems is where the boundaries lie. There are cellular boundaries, system boundaries, and the organism as a whole, which has a boundary between itself and its environment. These boundaries serve the dual purpose of keeping things separate but in contact. Along with these boundaries, different parts of a biological system have different degrees of resilience. For example, a skeleton versus soft tissue. These two concepts&#151;boundaries and joining different types of resilience&#151;can be useful in understanding software releases. I'll return to them later.

### Releasing Costs A Lot

There is a significant, often unmeasured, cost of features sitting in the deployment pipeline or queue. That cost used to be balanced by another cost, the cost of delivery. Driving down one meant driving up the other.

In the era of shrink-wrapped software, the cost of each release was huge. Physical diskettes or CDs had to be created, put in boxes, wrapped in plastic, put on trucks, driven to stores or mailed to businesses. All that costs time and money. If a release had *N* features, then the total cost of release *C* would give *C/N* as the release cost of each feature. So the total cost of a feature would be the cost to develop it, *D*, plus the cost of release, or *D + C/N*. In other words, the cost of releasing software was like a tax on every feature.

Releases are now practically free. The better we make the release process and the better your continuous delivery process, the cost of delivering you features approaches zero. That's called eliminating waste, which are resources spent on activities that do not provide value. Furthermore, the value of a feature is not that it exists, but in using it. The sooner you have it, the sooner you can leverage its value. As the cost of releasing software approaches zero, the relative cost of waiting for features gets higher.

### Programs Should Be Perfect

We assume that programs (individual pieces of software) should be perfect, or bug-free. The problem is, simply, that it's impossible. Instead, we should _expect_ individual pieces of software to have defects, and build systems to be robust. It's possible to create a system of components where the reliability of the system is greater than the reliability of any single component. In fact, this is how many parts of a computer are constructed.

Not only do we expect programs to be perfect, but we expend a lot of effort trying to maintain that illusion. We craft processes to check and double-check things, signing off on this or that check box. We engineer systems with the assumption that the parts _will work_ and consequently make the whole system brittle.

I'm not advocating that people be reckless or not care about failure. I'm advocating for the exact opposite. We can actually make systems better by expecting failure. Once you slay the lie that programs can be perfect, the oppressive fear of a program failing evaporates, replaced with a realistic effort to build resilient systems. [1]

To summarize, building software is more like a biological than mechanical process, we are approaching zero cost to deliver software, and we should focus on resilient systems that tolerate individual programs failing.

## Changing The Release Process

To reiterate, our priority is for Rubinius to have an impact on its users. With Rubinius 3.0, we continue the approach we started with the 2.0 release, pushing more releases with a smaller number of changes in each release.

Our goal is that soon, _continuous delivery_ for you will include pushing a new Rubinius release straight to production. If that sounds crazy, consider how much has changed in the past five years. Today, pushing Rubinius to production can mean dropping a [Docker](https://www.docker.com/) container in and if something goes awry, drop the whole mess and start over. It's truly amazing.

In software release, as everywhere else, we must focus on managing complexity. As the release cost goes down, other costs rise in relative importance. Someone mentioned recently that newer developers mistake the complexity of *N* features as being *N*. It's actually exponential, or _2<sup>N</super>_. If you have 2 features, it's *not* 2, it's *4*. If you have 3 features, the complexity is *8*. That's because each feature may interact with one or more of the others. Ideally, each release would have *one* feature.

This was underscored for me in a training with Sandi Metz recently. She was teaching us refactoring and we used the mantra "one undo from green" to keep us on the golden path and out of the weeds. If we have made one change and the tests fail, we can go back and then go forward, rather than piling confusion on confusion. The reduction in cognitive cost is significant.

The same can be applied to changing software. If we release Rubinius with small enough changes and you are many releases behind the current one, you can jump all the way forward at once, or apply one change at a time, or bisect. Whichever way you choose, you can significantly simplify upgrading and more easily pinpoint any problem.

We aim for each release to be so small, it's hardly worth mentioning. The focus changes from a big release announcement to the point when a customer notices the value of a new feature. Think of it like a menu. Increasing the granularity makes it possible for the customer to combine things in ways that work for them, rather than packaging it up for them in only one way, and taking a long time to do so.

## Automatic Update

Besides changing the focus of the release process, we are adding a feature to Rubinius to support the process: _automatic update_.

The first time you start Rubinius after a fresh installation, if the setting does not already exist, you'll get a message like the following:

    Rubinius is able to automatically install new versions. Installing
    a new version will not overwrite your existing version.

    Enable this feature? ([Y]es, [n]o, [a]uto, ne[v]er, [h]elp):

When a new version is available, you'll get a message like the following:

    Version 3.0.5 is available. You are running version 2.8.12. Installing the
    new version will not overwrite this version, both versions will be available.

    Install the new version? ([Y]es, [n]o, [a]uto, ne[v]er, [h]elp):

We will install the new Rubinius from a binary build using the same packager that was used to install the existing one. Since [chruby](https://github.com/postmodern/chruby) already works exceedingly well with many existing system package managers, the new version of Rubinius should seamlessly fit into your workflow if you are using `chruby`.

## How You Can Help

There are a number of things you can do to help us be more effective in getting new Rubinius features into your hands.

1. Help us strip the ornamentation out of releases. Help us find the equivalents of the cardboard boxes, plastic wrap, CD cases, and related waste in our release process and eliminate them so we can deliver features more quickly.
2. Help us create binary packages. We're going to dedicate December to _Binary Packages for Rubinius_ month. We hope that if you have some free time over the holidays, you will experiment with building binary packages for Rubinius or helping existing maintainers update their packages
3. Communicate with us about what is working and what can improve. Help us understand the problem you have. Open an issue or write a post about how you're using Rubinius and link us.
4. Help other people get started with Rubinius. The [chruby](https://github.com/postmodern/chruby) utility is our favorite Ruby switcher. It's so simple and just works. It also can switch Rubies installed by many OS system packages, and other installers/switchers like [rbenv](https://github.com/sstephenson/rbenv). Even if you don't end up changing your current Ruby switcher, give chruby a shot. It has made life _so much better_ for us.

## Principles

Now that we've looked at the problems with releasing software and at what Rubinius is doing differently with 3.0, as well as what you can do to help, I want to tie everything together into one simple idea.

Recently, Tom Dale posted [The Road to Ember 2.0](https://github.com/emberjs/rfcs/pull/15). The Ember authors have many new features and better approaches they want to implement derived from ample contact with the struggles that developers using Ember face. However, they realize that Ember users _need things now_. To balance these needs, they have a mantra: _stability without stagnation_.

I've always been impressed with the Ember development effort and it's exciting to read about the work they're doing. It's also validating to hear them talk about tackling similar issues to the ones we've faced with Rubinius. I would phrase their idea it the opposite way. In Rubinius, we are aiming for **progress with purpose**.

The guiding principle is to iterate _**on**_ what we want, not _**toward**_ what we want. We want to *start* with a functioning kernel of a feature and grow it into a full-fledged, mature component. Going back to the discussion of biological versus mechanical models above, we are focused on getting just enough of the skeleton and boundaries in place to enable consistent, functional growth. Much of the design for Rubinius 3.0 has been done. The next three posts will get into technical details.

## Acknowledgments

<em>I want to thank the following people: Chad Slaughter for entertaining endless conversations about how we build software and challenging my ideas.  Yehuda Katz for bringing us many nice ideas. Joe Mastey and Gerlando Piro for review and feedback, some of it on these topics going back more than a year.  The Rubinius Team, Sophia, Jesse, Valerie, Stacy, and Yorick, for putting up with my last-minute requests.</em>

[1]: http://techblog.netflix.com/2012/07/chaos-monkey-released-into-wild.html "Chaos Monkey Released Into The Wild"
