---
layout: post
title: "Rubinius :heart: Gitter IM"
author: Brian Shirai
twitter: brixen
---

_Full disclosure: Gitter neither solicited nor reviewed this post but they do donate the Rubinius organization Gitter account at no cost so we can have full access to private chat and history. We thank them for this sponsorship._

[Gitter IM](https://gitter.im) is the sweet spot of open source, [GitHub](https://github.com)-centered, near-realtime team communication. We've been using it for months on the Rubinius project and I've been meaning to write this post for months because it was immediately obvious how good it is.  However, just before we started using it, I _didn't_ get it. I'm hoping this post will help you see why it is such a great tool.


## The Need To Communicate

One of the central, critical aspects of an open source project is communication.  We ask a question, share an idea, talk over some code, answer a question, coordinate work, and share general life experiences. Since we are distributed in time zones across the world, we need a balance point between communication that is synchronous (I ask a question and you immediately respond) and asynchronous (I ask a question and at some, undetermined, future time you respond).

Historically, [IRC](https://en.wikipedia.org/wiki/Internet_Relay_Chat) has been the medium for this sort of communication. Many people defend its use and some people are adamant that it's the only valid option for open source projects. Unfortunately, it has significant drawbacks for everyone involved in a project, experienced and beginner alike.

I'm comfortable in IRC and have been for years. But I pay for a VPS to maintain my IRC bouncer so I have history for all the channels I'm in even when I'm not physically online. I also need to maintain that server with security patches and upgrades.

There are alternatives to rolling your own IRC bouncer but the point is that there's a significant cost here. And most importantly, as a newcomer, you have to _know_ something about IRC before you can ask your first question. Unless you are contributing to IRC software, I'm certain your question has nothing to do with IRC.

And that's why the limitations of IRC affect everyone on the project. Numerous times I've logged in to find a question from someone and when I go to reply, I notice they are no longer in the channel. Missed connection, missed opportunity. Bummer. That doesn't only happen with newcomers either. The number of experienced developers that I've interacted with for _years_ on IRC who don't use a bouncer or service is high.

So, we have a vital need to communicate but just getting to the point of asking a question is a big hurdle. However, that's not where the difficulty ends. Even when someone has gotten on IRC, there are more challenges waiting.  This is where Gitter really shines. It eliminates common barriers to communication and improves the context for the communication.


## Barriers To Communication

On Gitter, your nick or handle is your GitHub user name. So simple; so useful.  It may not seem like a big thing: you have a GitHub user name and you have an IRC user name and you created them at different times so they are often different. Big deal; who can't remember two names?

Well, me for one. I need to keep track of a lot of people and not having to even wonder what a person's GitHub user is when I'm chatting with them on Gitter is a huge help. It's also a huge help for people who are new to the project and just getting oriented.

Think about this: when you go from one thing to two things, you've increased complexity by 100%. But as programmers, with our fancy loop constructs, we typically don't think twice (no pun intended) about multiplying by two. If you think that's not a huge deal, perhaps try asking for double your salary. In the "real world", multiplying by two is a big deal.

With Gitter, everyone is who they are on GitHub. And since we're working with code hosted on GitHub, that's an obvious, useful, simplication.


## Communicating In Context

Communication happens in a context. It's inseparable from that context. This is the key thing that Gitter gets right, and the thing that I didn't get until I created an account and started poking around.

Your GitHub organization and repositories have corresponding Gitter rooms.  Boom! Did you see that? Where do you go to talk about Rubinius?  [gitter.im/rubinius/rubinius](https://gitter.im/rubinius/rubinius). Simple, direct, and (in retrospect) obvious.

These are some of the contexts we are already using:

1. The Rubinius organization room ([gitter.im/rubinius](https://gitter.im/rubinius)): This room is public/private. It's available for everyone who is a member of Rubinius (over 100 people), but it's not visible to people outside the organization.  This is a perfect place to hold organization-member wide discussions.
2. The Rubinius team room: This is a private channel accessible only to [the Rubinius team members](http://rubinius.com/2014/11/10/rubinius-3-0-part-1-the-rubinius-team/) where we can have a safe space to discuss and share.
3. The main Rubinius room ([gitter.im/rubinius/rubinius](https://gitter.im/rubinius/rubinius)): This is a public room where anyone with a GitHub account can stop by to ask questions, leave comments, or interact. We have integrations enabled so we can see when issues are opened, comments are made on issues, code is updated, and Travis CI results are posted.
4. Individual private discussions: These are rooms where two people can hold a conversation in private.
5. Other project rooms: These are public rooms corresponding to other Rubinius repositories. Rubinius is a complex project with many parts. Having a dedicated room to discuss a specific project (for instance, the Rubinius bytecode compiler), provides a good way to put communication in context and cut down on the cross-communication that happens when a project has a single IRC channel. (Most projects I've been involved with had at most two IRC channels.)
6. Other private group rooms: These can be created as-needed by people leading specific parts of the Rubinius project. This is not being used extensively yet, but with the [work we're doing to define project roles](http://rubinius.com/2015/07/31/the-next-10-million-programmers/), this will be a very useful tool.

Of course, all of this is possible with IRC. But it requires specific, extra work. It's built-in with Gitter, and that is the best part of the service. It's not _all_ of why it's good, but it's an essential aspect.

Another aspect of context is the consistency of the experience. The Gitter app or the website function equivalently (I'm explicitly excluding the Gitter IRC bridge from this). With IRC, there are such a variety of clients that you don't necessarily know what the other person is seeing. With newcomers, this can impact the ability to communicate well. With Gitter, I can better evaluate if how I'm communicating is helpful or not because I can be reasonably sure they are seeing what I'm seeing.


## Communicating About Code

Finally, since we're working on an open source project, a lot of the communication is about code. With GitHub flavo(u)red Markdown, communicating about code _is really nice_.

It seems like a simple thing, but it's not. For example, HipChat gets this horribly wrong. Communicating about code in HipChat is like using Notepad.exe (sorry, Notepad diehards) to program. It's possible, but only by stretching the meaning of the word beyond recognition.

We often make the assumption that code is concrete, but it's usually quite complex and being able to communicate about it well and visually is tremendously useful. And even when not communicating about code, the formatting available with Markdown makes communicating better and more enjoyable.

So, those are the reasons I think Gitter is awesome. Have you used it? Do you like or dislike it? What do you use instead and why do you like that? We'd love to hear from you: <community@rubinius.com>.
