---
layout: doc_ja
title: Abstract Syntax Trees
previous: Parser
previous_url: bytecode-compiler/parser
next: Generator Stage
next_url: bytecode-compiler/generator
review: true
---

パーサが`process_*`メソッドでの処理を終えると、抽象構文木が出来上がります。
この構文木は、パースされたコードの構文構造を抽象的に表現します。
各ノードがソースコード内の１つの構造を表しています。
このツリーは *lib/compiler/ast/* ディレクトリ内で定義されているクラス群の
インスタンスによって構成されています。
各クラスは`Rubinius::AST::Node`クラスを継承しています。
この`Node`クラスにはサブクラスから利用されるいくつかのメソッド
（例えば、デバッグ用に行番号を記録する pos(g)、など）
が定義されています。

ASTクラスには通常、少なくとも以下の３つのメソッドが定義されています：

* initialize(line, args) - 前のステージで、さまざまな `process_*` メソッドから呼ばれます。
* bytecode(g) - 次のステージのGeneratorが使います。
* to_sexp - ASTノードをシンボリックな表現(S式)にします。

作成されたASTを確認する一番簡単な方法は
`to_ast`メソッドをコードの文字列に対して呼ぶことです：

    irb(main):002:0> "a = 1".to_ast
    => #<Rubinius::AST::LocalVariableAssignment:0xf70
       @value=#<Rubinius::AST::FixnumLiteral:0xf74 @value=1 @line=1>
       @name=:a @variable=nil @line=1>

もしくは、コードをコンパイルするときに -A オプションを指定します：

    rbx compile -A -e "def hello;end"
    Script
      @name: __script__
      @file: "(snippet)"
      @body: \
      Define
        @name: hello
        @line: 1
        @arguments: \
        FormalArguments
          @defaults: nil
          @names: \
          @block_arg: nil
          @optional: \
          @splat: nil
          @line: 1
          @required: \
        @body: \
        Block
          @line: 1
          @array: \
            NilLiteral
              @line: 1

同様に、`to_sexp`メソッドで構文木のS式の表現を得ることもできます：

    irb(main):002:0> "a = 1".to_sexp
    => [:lasgn, :a, [:lit, 1]]

もしくはコンパイル時に -S オプションを使います：

    rbx compile -S -e "def hello;end"
    [:script, [:defn, :hello, [:args], [:scope, [:block, [:nil]]]]]

抽象構文木は、ノードが他のノードを持つ、ネストした構造です。
例えば、上の例にある `hello` メソッドの定義の場合、
その構文木は`Script`ノードで、`@body`に`Define`ノードを持ちます。
この`Define`ノードの`@arguments`には`FormalAguments`ノードが、
`@body`には`Block`ノードが入っています。
そしてこの `Block`ノードの`@array`には`NilLiteral`だけが入っています。
この`NilLiteral`ノードは葉（リーフノード）で、他のノードを持ちません。

以下の `if`式と：

    rbx compile -S -e ":foo if :bar"
    [:script, [:if, [:lit, :bar], [:lit, :foo], nil]]

違う方式で書かれている同等のif式が：

    rbx compile -S -e "if :bar then :foo; end"
    [:script, [:if, [:lit, :bar], [:lit, :foo], nil]]

全く同じ構文木になることに注意して下さい。
実際のシンタックスの全てを正確に表現するわけではないので、
「抽象」構文木と呼ばれています。


## 参照されているファイル

* *lib/compiler/ast/*: 全てのASTクラスはこのディレクトリ以下に定義されています。

## カスタマイズする

コンパイル処理のこの段階をカスタマイズするには２つの方法があります。
一番簡単な方法はASTの生成を
[AST Transforms](/doc/en/bytecode-compiler/transformations/)
を使ってカスタマイズすることです。

Melbourneパーサのサブクラスを定義し、
`process_`メソッドを再定義することもできます。
しかし、この方法は上級者向けで、まだ十分に実証されていません。
