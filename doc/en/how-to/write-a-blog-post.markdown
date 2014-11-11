---
layout: doc_en
title: How-To - Write a Blog Post
previous: Write Benchmarks
previous_url: how-to/write-benchmarks
next: Write Documentation
next_url: how-to/write-documentation
---

The [Rubinius blog][1] uses [Jekyll][2] and is integrated with the website and
documentation. We encourage and appreciate guest blog posts about your
experiences using or developing Rubinius.

The preferred format for blog posts is Markdown. However, if you have special
formatting needs, the post can be written in HTML directly.

Clone Rubinius repository as that is where the website and posts are stored:

    git clone https://github.com/rubinius/rubinius.github.io.git

To get started, ensure you have the `github-pages` gem installed. Use
`bundler` inside your local clone directory:

    rbx -S bundle install

Now go to the posts directory:

    cd _posts

Create a file in the console using the filename format `YYYY-MM-DD-perma-link.markdown`.

    touch "%(date +"%Y-%m-%d")-perma-link-title.markdown"

Write your brilliant post:

    cat /dev/random > <<the file post>> # :-p

Run jekyll to serve the website locally, including your post:

    rbx -S jekyll serve --watch

Once you are happy with the results create a commit of your post:

    git add _posts/
    git commit -m "Wrote a blog post on ....."

Submit a patch, pull request, or if you have commit rights, push the commit to
the master branch.

Tell us that there is a new blog post. We may have some feedback for you before
publishing.

[1]: /blog "Rubinius' Blog"
[2]: https://github.com/mojombo/jekyll "Mojombo's Jekyll"
