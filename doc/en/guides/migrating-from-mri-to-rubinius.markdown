---
layout: doc_en
title: Guides - Migrating from MRI to Rubinius
previous: Guides
previous_url: guides
next: How-To
next_url: how-to
---

This guide will assist you to migrate your Ruby or Ruby on Rails application
from MRI (Matz's Ruby Implementation) to Rubinius. This guide assumes that you
are familiar with Ruby or Ruby on Rails, RubyGems, and Bundler. It also assumes
that you can use the command line shell.

## 1. Installing Rubinius

The first step in migrating to Rubinius is to install Rubinius. Since you will
likely need to switch back and forth between MRI and Rubinius while migrating,
the easiest approach is to use a Ruby switcher utility.

The [chruby](https://github.com/postmodern/chruby) command line tool for
switching between Ruby implementations is recommended for use with Rubinius.
All examples in the guides will use `chruby`.

### Installing chruby and ruby-install

If you are using OS X, the easiest way to install `chruby` and `ruby-install`
is to use the [Homebrew](https://github.com/Homebrew/homebrew) package manager
as follows:

    $ brew update
    $ brew install chruby ruby-install

If you are not using Homebrew or are using a different operating system than OS
X, please see the [chruby install
instructions](https://github.com/postmodern/chruby#install) and the
[ruby-install install
instructions](https://github.com/postmodern/ruby-install#install).

To configure `chruby`, add the following to the ~/.bashrc or ~/.zshrc file:

    source /usr/local/opt/chruby/share/chruby/chruby.sh

See the documentation for more [configuration
instructions](https://github.com/postmodern/chruby#configuration).

### Using ruby-install

Once you have installed the `ruby-install` utility, installing Rubinius is
straightforward:

    $ ruby-install rbx 2.2.5

For instructions for installing other Ruby implementations, please see the
`ruby-install` [installation
synopsis](https://github.com/postmodern/ruby-install#synopsis)

### Using chruby

Now that Rubinius is installed, you can make it active with the following
command:

    $ chruby rbx

## 2. Gems

Most gems that run on MRI should run on Rubinius, with the exceptions noted
below. Additionally, Rubinius makes many components of the system available as
gems. These include the Rubinius tools for parsing and compiling Ruby code, the
Ruby debugger and profiler, and the Ruby standard library. All these gems are
pre-installed when you install Rubinius.

### C-extension Gems

Many gems that use C-extensions run fine on Rubinius. The exceptions are ones
that rely on MRI internal data structures. These gems cannot be supported on
Rubinius and include gems like `ruby-debug` and `ruby-prof`. Rubinius provides
its own Ruby source debugger and profiler, as well as other tools. See the
[tools documentation](http://rubini.us/doc/en/tools/) for more details.

## 3. Gemfiles

Your Gemfile should work fine with Rubinius but you should run `bundle update` to force the gem dependencies to be recomputed for Rubinius. Also, if you are using gems that are incompatible with Rubinius, you can put them in a `platforms` block until completing the migration to Rubinius as follows:

    # Example platforms block for MRI-specific gems
    platforms :mri do
      gem 'ruby-prof'
      gem 'ruby-debug'
    end

## 4. Compatibility Issues

Rubinius created the [RubySpec](http://rubyspec.org) project, and continually
enhances it, to describe the behavior of Ruby and monitor compatibility with
MRI.

With few exceptions, Rubinius is expected to be compatible with MRI 2.1.  Some
features, like keyword arguments, are not yet implemented. Other features may
not be implemented because they are unknown and no RubySpecs exist for the
feature yet. Several standard library components, including Continuation,
Ripper, TracePoint, and Tracer, are not yet implemented but may be in the
future.

If you encounter an incompatible behavior in Rubinius compared to MRI, it's
most likely a bug. Please [open an
issue](https://github.com/rubinius/rubinius/issues) for it.
