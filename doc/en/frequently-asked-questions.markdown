---
layout: doc_en
title: Frequently Asked Questions
previous: Index of Terms
previous_url: index-of-terms
---

## Can I hide source code using Rubinius' byte code format?

No. Rubinius' byte code format provides no mechanisms to obfuscate the
contents of it or the original source code. Source code obfuscation will never
be added to the byte code format.

## Does Rubinius support ARM?

Not entirely. Currently the JIT does not support ARM and as such you'll have to
disable it if you want to run Rubinius on ARM. Even then we can't guarantee
that Rubinius will work reliably on this architecture. We hope to support ARM
in the future.

For more information, see the following issues:

* <https://github.com/rubinius/rubinius/issues/2867>
* <https://github.com/rubinius/rubinius/issues/2331>

## Does Rubinius support Windows?

No, not at the moment. While we would love to support Windows we sadly don't
have anybody with Windows experience on the team. A corresponding issue for
this can be found at <https://github.com/rubinius/rubinius/issues/2882>.

If you would like to help with getting Rubinius running on Windows please get
in touch.

## Why is the README of Rubinius in plain text?

Various developers have opened pull requests over time to change the plain text
README to a Markdown formatted README. Due to the simplicity of the README we
have decided to keep it plain text only.

For more information, see the following:

* <https://github.com/rubinius/rubinius/pull/1007>
* <https://github.com/rubinius/rubinius/pull/2903>
* <https://github.com/rubinius/rubinius/pull/1603>

## What are all these rubysl Gems doing in my Rubinius installation?

Rubinius uses Gems for its standard library. For example, `rubysl-date` is a
Gem containing the `Date` library used by Rubinius. These components are
installed, updated and otherwise maintained as ordinary Gems.

Although you can technically remove then it is not recommended to do so. Due to
these components being used throughout the Rubinius & Ruby ecosystem you might
end up with a broken installation if you remove them.

## How to install rubinius on OSX 10.10 "Yosemite"?

Warning: don't try to install rubinius using system ruby. Use `rbenv`, `chruby` or `rvm` to build latest stable MRI and then run ``ruby-install`` after selecting freshly built ruby.

1. Get the homewbrew if you don't have it: ``ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)"``
2. Get llvm: ``brew install llvm``
3. Get latest ruby-install: ``brew install ruby-install --HEAD``
4. Install with homebrews llvm: ``ruby-install rbx -- --llvm-path="/usr/local/Cellar/llvm/3.2.4"``


