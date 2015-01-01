---
layout: doc_en
title: How-To - Write a Ruby Spec
previous: Write a Ticket
previous_url: how-to/write-a-ticket
next: Fix a Failing Spec
next_url: how-to/fix-a-failing-spec
---

Make sure you have read:

  *  [Getting Started](/doc/en/getting-started/)
  *  [Specs](/doc/en/specs/)

Then, follow these steps to write a spec for a Ruby method:

  1. Edit a file under `spec/ruby/...`
  1. Run `bin/mspec -tr spec/ruby/some/spec_file.rb`
  1. Repeat until the spec passes on MatzRuby
  1. Run `bin/mspec -tx spec/ruby/some/spec_file.rb` to test with Rubinius
  1. Commit your changes
  1. Open a pull request.
     If you are also implemented the method, feel free to do that
     together in a single pull request but make sure you have separate
     commits for the specs and code.
