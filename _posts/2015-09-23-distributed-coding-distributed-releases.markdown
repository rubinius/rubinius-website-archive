---
layout: post
title: "Distributed Coding, Distributed Releases"
author: Brian Shirai
twitter: brixen
---

In [the last post](http://rubini.us/2015/09/22/major-minor-maximize-delivering-features-minimize-trouble/), I talked about the new Rubinius versioning scheme. A version doesn't mean much to you if there's no release that goes with it. In this post, I'll describe the new release process we've been using.

What is a release, fundamentally? **For Rubinius, it's a function from a version number to a commit SHA.** A release is a git tag on _master_, and from this we can automatically derive the version number, the date of the release, and the commit SHA.

What a release isn't is a tarball or binary or any other artifact. As such, I make a distinction between _releasing_ and _a release artifact_ (or _deploying_, to borrow a word we use typically with SaaS). First we release, then we build a release artifact. Leveraging this two step process gives us the ability to correct errors early. We can easily delete a git tag and push a new one. Once a release is done, we build one or more artifacts and, in a sense, freeze the release in time.

This process solves a major coordination issue that we had with our previous process. Previously, we had to actually commit code to the repository to make a release. This meant that someone else could push code that changed what would be included in the release by racing on committing, or could push code that would conflict with the code in your release commit. With a git tag, synchronization is automatic, and it's a very light weight operation to delete a tag and push a new one when necessary.

This process also gives anyone with commit rights to the Rubinius repository the ability to make a release. Making a release requires a human; we are not going to automate it. The second part, building the release artifacts, needs to be automated and we are working on that. For example, we'll build the binary version that you use on Travis to test your code with Rubinius as part of our Travis CI job. We'll also upload the release source tarball directly from Travis.

The new release process and automation of the release artifacts, in concert with the new version scheme, should help us accelerate getting Rubinius enhancements and features to you with a minimum amount of work. Additionally, by making releasing something that any contributor can do, it will improve the scope of participation in the project. We're really excited about these developments. If you have questions, let us know <community@rubini.us>.
