---
layout: post
title: Testing Your Project with Rubinius on Travis
author: Brian Shirai
---

[Travis CI](http://travis-ci.org) has been a tremendously useful tool for
automating testing and has provided Ruby implementations with valuable
feedback. However, it can be a trial-and-error process to find the right
incantations for `.travis.yml` to get your preferred selection of Ruby
implementations running. This post explains how to test your project on
Rubinius.

With the Rubinius 2.0 release, there are no longer language modes. Rubinius is
presently working on compatibility with the upcoming MRI 2.1 release. There
are still compatibility issues to be fixed, primarily keyword syntax and some
core library API changes. What this means for your `.travis.yml` file is that
'rbx-18mode' and 'rbx-19mode' are no longer supported. We submitted a patch to
the [Travis lint tool](https://github.com/travis-ci/travis-lint) to check for
this, so please use the linter!

So, if those language modes are no longer available, what should you add to
your `.travis.yml` file? Remember that Rubinius is releasing new versions every
week or so. Together with recent changes to RVM, this gives four options for
how specific you want to be about the Rubinius version you test against. The
following list of Rubinius options is from least-to-most specific:

1. 'rbx' - This means the most recent Rubinius release. Every time your tests
   run, the most recently released binary will be used.
1. 'rbx-X' - This means the most recent Major release. For example, if you use
   'rbx-2' (which is the only one available right now), your tests will run on
   the most recent 2.Y.Z release, but would not run on 3.Y.Z.
1. 'rbx-X.Y' - This means the most recent Minor release. If you use 'rbx-2.2',
   your tests will run on the most recent 2.2.Z release, but will not use the
   2.3.Z or 2.1.Z release.
1. 'rbx-X.Y.Z' - This means precisely the specified release. So, 'rbx-2.2.1'
   will run only on the 2.1.1 releease and no other release.

This method of designating the version of Rubinius you wish to run against
should look similar to specifying versions of gems with the pessimistic
operator (~>). It is intended to give you the same flexibility while allowing
for stability constraints that you choose.

Since each of these designations is independent, you can mix and match them as
you wish. For example, if you know that Rubinius 2.2.1 is green for your
project, you can specify both 'rbx-2.2.1' and 'rbx'. Or you can always stay on
the cutting edge by just using 'rbx'.

If you need help getting your project set up or updated on Travis, please let
us know. I've created a simple project called
[travis-canary](https://travis-ci.org/rubinius/travis-canary) that you can
check to see if the ways of specifying Rubinius listed above are currently
working on Travis.

Happy testing!
