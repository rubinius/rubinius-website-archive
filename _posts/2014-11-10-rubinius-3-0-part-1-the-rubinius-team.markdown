---
layout: post
title: "Rubinius 3.0 - Part 1: The Rubinius Team"
author: Brian Shirai
---

Today, I'm introducing some big changes for Rubinius. As the title says, this
post is only part one. There are five parts and I'll publish one each day this
week. I'll be covering these topics: _the Rubinius Team, the development and
release process, the Rubinius instruction set, the Rubinius system and tools,
and one more thing._

Also, as the title says, this is **Rubinius 3.0**. The past year has been
incredibly influential in helping me understand the many facets of Rubinius as
a project and Ruby as a language and community. The other posts will dive into
more detail, but I want to highlight that all of this is Rubinius 3.0.

## Introducing the Rubinius Team

Sometimes we save the best for last, but this is not one of those times. I'm
tremendously honored and excited to introduce you to the **Rubinius Team**.
They've all volunteered to contribute their time, experience, and passion to
improving Rubinius and its impact to make the world better.

Here we are, in no particular order:

**Sophia Shao**: As a recent graduate of Carnegie Mellon University's
Electrical & Computer Engineering department, Sophia is currently tackling a
massive application migration from MRI to Rubinius. She's also been improving
Rubinius every day. Hit her up for tips about debugging machine code.

**Jesse Cooke**: As co-founder of [Watsi](https://watsi.org/), a venture to
fund healthcare for people around the world, Jesse was part of YCombinator's
first ever non-profit. Jesse has been contributing in any way he can to
Rubinius for a long time. If you visit Portland, OR, you may see him riding
this weird bike with a belt instead of a chain.

**Valerie Concepcion**: If you're interested in getting things like Raspberry
PI's, Legos, and Wii Remotes to play well together, Valerie can help. Drawn to
the Maker movement and inspired by her friends who work in non-profits, she is
interested in applying technology for social good.

**Stacy Mullins**: At one point, Stacy would have gladly chosen a typewriter
over a computer. But at school for graphic design, she became fascinated by
technologies like HTML and CSS and the ability to create something from
scratch. Now she's learning about crafting code and communicating well with
other developers.

**Yorick Peterse**: When not breaking code, Yorick is fixing it and asking
questions. Either way, there is a lot of code happening. He's drawn to the
deep technical details of systems like just-in-time compilers and concurrency.
He may or may not be a Dr. Evil character hatching plans for world domination.

**Brian Shirai**: Having once passed over Ruby for being too much like Perl,
Brian rediscovered Ruby over ten years ago and has been working on Rubinius
for the past eight. Inadvertently, he's also learned Perl.

### Why?

After all these years, why do I want to form a Rubinius Team? And what is it?
Is it like the "core team" we see in Rails or other projects?

I'm so glad you're wondering about that!

Early in the Rubinius project, Evan Phoenix started a policy we called "the
open commit bit": if we accept your patch, you get permission to commit
changes to the source code repository.

This contrasted with many open source projects that had a small number of
people who could make changes to the code. Usually, this group was called a
"core team". Limiting permission to change the code was seen as an essential
part of maintaining code quality. If anyone could commit, people would just
make a big mess.

This conventional wisdom turned out to be false. We let anyone who made one
good patch have access to commit any changes they wanted. In practice, almost
everyone was extremely careful. We rarely had to revert changes, and when we
did, it was not usually a question of quality. Hundreds of people committed
changes and Rubinius benefited a great deal.

For this reason, whenever the topic of a "core team" for Rubinius came up,
Evan opposed it. There was no real value in trying to be gate keepers. Giving
people the opportunity to contribute and welcoming them to do so had a
positive impact and showed appreciation for their efforts.

**The Rubinius Team is *not* about creating a different class of contributor,
exclusiveness, gate-keepers, or overseers.**

Another characteristic of the typical open source project "core team" is that
the members are usually the most technically skilled and have the greatest
number of commits. This automatically creates an imbalance of emphasis on only
technical issues and technical expertise, despite the fact that the vast
majority of people using, contributing to, or impacted by a project will not
be "top technical contributors".

**The Rubinius Team is *not* focused exclusively, or even primarily, on the
technical aspects of the project.**

A third characteristic of typical "core teams" is the implicit privilege of
the members and the resulting economic, gender and diversity imbalance.
Someone struggling with two jobs won't have time to be a top committer, no
matter how capable they are. Likewise for someone caring for kids at home, a
responsibility that disproportionately rests with women. All of these problems
stem from the dangerous fallacy that open source software is a "meritocracy".

### So, what *is* the Rubinius Team?

**The Rubinius Team is a group of people who work together, influenced by our
values, to accomplish things that fulfill the Rubinius vision and mission.**

Our vision is a world where Ruby is the most useful programming language for
building things that improve people's well-being and quality of life.

"Most useful" means the most benefit for the least amount of effort for the
greatest number of people. There will always be incredibly smart people who do
very difficult things. For the rest of us, to steal a quote by Moshe
Feldenkrais, we want to "make the impossible possible, the hard easy, and the
easy elegant".

Our mission is to build the best Ruby implementation and the best programming
tools that benefit the greatest number of people, prioritizing our efforts to
improve access for people who have been marginalized and excluded.

We value impact, quality, inclusiveness, diversity and balance, and we
actively promote them. We celebrate our differences and appreciate them as a
source of strength. We prioritize improving access and championing the needs
of people who have traditionally been excluded. We get things done, lead by
example and we constantly strive to improve. We realize that we enjoy a lot of
privilege and we work hard to empower others rather than advancing our own
interests.

We welcome anyone who shares our vision, mission and values to be a part of
the Rubinius Team. And one of our objectives will be growing the team. There
are many roles to play. From outreach to industry, academia, and communities
like [Women Who Code](https://www.womenwhocode.com/) and [Black Girls
Code](http://www.blackgirlscode.com/) to marketing, budgeting, and planning.
From documentation to organizing meetups. There are many ideas we don't even
know about yet, and are waiting for you to create.

### It's about quality

I want to talk more about the over-emphasis of technology in open source
projects because I don't hear this discussed often.

The source code written is a small part of a much bigger picture. The purpose
of design is to create something that is useful for humans. Better
understanding leads to better design. Better design leads to a more effective
tool. A more effective tool leads to better engagement. Better engagement
leads to greater understanding. There is no hierarchy here; there is no
ranking. They form a circle of interaction. Each of these is important, and
any one of them is only as good as all the others. 

We strive to ensure that _we are reaching the people we want to help, and that
we are helping the people we want to reach_. We do this by seeking global
understanding of the problems our community needs to solve. Too narrow a focus
on the local technology problems will mislead us.

Pondering these matters leads us to consider the Rubinius community.

## The Rubinius Community

I have a very broad view of the Rubinius community. It includes developers and
people learning to write Ruby. It also involves people who are not primarily
involved in programming but may need to understand or even write some Ruby
code. For example, a database developer working with a team of Ruby
programmers on an application. The community also includes the people who use
the software written in Ruby. And it includes the businesses who employ people
to write in Ruby.

The Rubinius Team is also a part of the Rubinius community. The relationship
between the Rubinius Team and the Rubinius community is important. The Team's
purpose is to help the community. And here, "help" means to serve.

In business the people we serve are our customers, but the concept of a
customer is not common to open source projects. Since people do not usually
pay for open source software, the idea of a customer does not seem to make
sense. However, envisioning the user of Rubinius as a customer has many
benefits. To develop an effective product, we must deeply understand the needs
of a customer.

The customer relationship provides important benefits to both sides. On one
hand it clarifies who we, the Team, are trying to help and to whom we are
responsible. On the other, it makes clear the customer's responsibility to
engage and communicate clearly, and to provide feedback to help us improve.
Both sides must be vested in the relationship.

This is where the analogy of a typical business relationship begins to break
down when applied to open source. We provide a thing of value: Rubinius. What
thing of value does the customer provide in return? One thing is the person's
time. Taking the time to try Rubinius, open an issue, or share their
experience with someone else is a thing of value they are giving Rubinius.
However, there is not yet a thing that has the same tangible value as money.
When we are asked to pay money for something, it increases the stakes for us.

We want the Rubinius community to be healthy, inclusive, safe, and helpful. We
want people to learn and grow and build awesome things. So we are adopting a
Code of Conduct for the community based on the [Citizen Code of
Conduct](http://citizencodeofconduct.org/) by the excellent [Stumptown
Syndicate](http://stumptownsyndicate.org/). We know this will be an important
aspect of creating an environment of respect and support as we continue to
explore how to improve the relationship between Rubinius as a product and
project and those who use Rubinius.

I'm excited to share more about the path of Rubinius 3.0 in the other posts
this week. We'd love to hear from you. Please send your comments to
[community@rubini.us](mailto:community@rubini.us).

## Acknowledgments

<em>I want to thank the following people:  Ashe Dryden and James Coglan, who
have forced me to question many things about open source projects. Evan
Phoenix for starting and leading Rubinius. Chad Slaughter for taking a risk
and being a stellar mentor. The Rubinius Team, Sophia, Jesse, Valerie, Stacy,
and Yorick, for their generosity and feedback on the post. Joe Mastey and
Gerlando Piro for their review and many fruitful conversations.
[Enova](http://enova.com/), for giving me hard problems to solve. And thanks
to you, the Rubinius community, for making it worthwhile.</em>
