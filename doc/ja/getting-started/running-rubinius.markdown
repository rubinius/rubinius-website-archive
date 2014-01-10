---
layout: doc_ja
title: Running Rubinius
previous: Building
previous_url: getting-started/building
next: Troubleshooting
next_url: getting-started/troubleshooting
translated: true
---

Rubiniusをビルドする（そしておそらくインストールする）指示に従ったのであれば、
Rubiniusが動作していることを以下のコマンドで確かめられます。

    rbx -v

Rubiniusはコマンドライン上でRubyと同じように動作します。例えば、

    rbx -e 'puts "Hello!"'

'code.rb'という名前のRubyファイルを実行したい場合、

    rbx code.rb

IRBを実行したい場合、

    rbx

RubiniusのbinディレクトリをPATHに追加していれば、Rubiniusは
MRIから期待されるのと同様に振る舞うはずです。
`ruby`, `rake`, `gem`, `irb`, `ri`, そして `rdoc`のためのコマンドがあります。

Rubiniusを利用したい場合にのみ、binディレクトリをPATHに追加してください。
そうすれば、Rubiniusを利用したくないときに通常のRubyの邪魔をせずに済みます。
