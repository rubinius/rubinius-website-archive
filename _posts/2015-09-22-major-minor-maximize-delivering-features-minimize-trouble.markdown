---
layout: post
title: "MAJOR.MINOR: Maximize Delivering Features, Minimize Trouble"
author: Brian Shirai
twitter: brixen
---

There are a lot of versioning schemes out there, from lax to strict to weird and everything in between. It's the stuff of endless debates and plenty of disappointment and unhappiness all around. That's a bit sad since a versioning scheme is about getting new stuff. Who doesn't want new stuff? Rubinius is switching to a MAJOR.MINOR version number scheme and I'd like to tell you how it works and why we're doing it.

At its core, a versioning scheme is part communication and part contract. The communication part is a signal from developers to users that an update is available. The contract part is an agreement about the impact of the update on the user.

Additionally, the versioning scheme exists in the context of two competing concerns: 1. the developer wants to deliver features and improvements; and 2. the user wants the highest stability for the least amount of effort on their part.

It seems to me that the conflict between these two is not well appreciated. Every user's needs are unique (even allowing for some equivalence classes), and trying to devise a scheme that meets the union of those is impossible. This is especially true as the scope of the software covered by a single version number increases (and hence why the _small modules_ approach can look very attractive). Consequently, there's a tension between batching things up in big enough chunks to accommodate the users who update slowly and providing new features quickly to those that update often.

To complicate both these aspects, the software development industry seems to operate as if it's still putting bits on physical media, putting those into boxes, into trucks, and onto store shelves. Very slowly, we are moving away from this model and to one where a mostly invisible stream of improvements find their way automatically to our devices and applications. Thank goodness.

So, what's in a Rubinius version number under this MAJOR.MINOR scheme? One part communication and one part contract:

1. Communication: The MAJOR number designates an _epoch_--a period of time in the project's history typically marked by notable events or particular characteristics (adapted from the Apple dictionary). The MINOR number is a monotonically increasing number. **If you are using Rubinius version A.N and there exists a version A.M, where M > N, you should upgrade.**
2. Contract: **If you are using Rubinius version A.N, version A.N+1 will only: 1. remove a previously deprecated feature; or 2. add a deprecation warning.** In other words, if you have no deprecation warnings, you can update to A.N+1 and expect no breaking changes.

This versioning scheme helps us deliver features faster, which is one of my main goals. And it explicitly decouples Rubinius changes from your decision about when to upgrade. I provide a clear signal and a clear expectation about the impact of a new version. If you decide to update every 15 or 20 versions, you can do so in whatever way works best for you. You can jump ahead to the newest version and if that doesn't work, bisect version numbers just like you might bisect git commits to find an issue.

Finally, this scheme ties in well with providing landmarks about Rubinius development. At any point in time, there are the things we have learned but haven't delivered features based on that learning yet, and things we are learning. I dislike roadmaps because they are mostly predictions, and we collectively are terrible at predictions. Focusing on landmarks gives the opportunity to discuss direction without necessarily deciding on the path there. As we learn, we deliver features and improvements and learn more. Of course, this approach is possible with other versioning schemes, but I think the one I'm describing here is especially good for this approach.

As for landmarks, here are a couple: the last version of Rubinius 2.x will be 2.71828182 and the last version of Rubinius 3.x will be 3.14159265. That should be plenty of numbers to do some interesting stuff.

The new versioning scheme will start with the next release, version 2.6. We'll be working the issues out of the deprecations process, so please be patient with that. As always, we love to hear about your experiences, so drop us a note <community@rubini.us>.
