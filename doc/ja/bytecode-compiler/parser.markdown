---
layout: doc_ja
title: Ruby Parser
previous: Bytecode Compiler
previous_url: bytecode-compiler
next: AST
next_url: bytecode-compiler/ast
review: true
---

コンパイル処理のパイプラインの最初のステージはRubyパーサです。
Rubyパーサはコードの文字列またはファイルを入力として受け取り、
ASTを次のステージであるジェネレータに渡します。

パーサ（Melbourne）にはCで書かれたパートとRubyで書かれたパートがあり、
前者は基本的にはMRIのパーサで、後者はRubyのASTの生成を担当しています。
Cのパーサはパースツリーの各ノードについてメソッドを呼ぶことで
Rubyとコミュニケーションをとります。

各メソッドは処理対象となる部分木について、その特徴となる全ての情報を受け取ります。
例えば、対象のRubyコードに`if`文がある場合、
Cパーサは行番号、条件を表す値、if文の本体を表す値、もしあればelse節を表す値、
を引数として、`process_if`を呼びます。

    def process_if(line, cond, body, else_body)
      AST::If.new line, cond, body, else_body
    end

Rubiniusのソースコード内の`lib/melbourne/processor.rb`を見ると、
全ての起こりうる`process_`呼び出しが確認できます。

多くの場合、パーサは`process_`メソッドに
その一つ前に呼び出された`process_`メソッドの結果を渡します。
例えば `true if 1` というコードでは、パーサはまず `process_lit(line,1)` と
`process_true(line)` を実行します。
加えて、この場合元のパースツリーは`else`の本体として`nil`を持っているので、
 `process_nil(line)`も実行されます。
その後、`process_if`を、行番号、`process_lit`の結果、`process_true`の結果、
`process_nil`の結果を引数にして呼び出します。

このような処理が再帰的にツリー構造を構築し、
Rubiniusはそれを次の段階であるジェネレータステージに渡します。

## 参照されているファイル

* *lib/melbourne/processor.rb*: CパーサのRubyインターフェースです。
  このファイルには `process_` で始まる名前のメソッド郡が定義されていて、
  これらのメソッドはCパーサがのCのパースツリーの各ノードについて呼び出します。

* *lib/compiler/ast/\**: Melbourneが使う各ASTノードの定義があります。

## カスタマイズする

コンパイル処理のこのステージをカスタマイズするには２つの方法があります。
ASTの生成をカスタマイズする一番簡単な方法は、
[AST Transforms](/doc/en/bytecode-compiler/transformations/)を使う方法です。

Melbourneのサブクラスを定義し、`process_`メソッドを好きなように定義する方法も
あります。こちらは上級者向けのまだ十分に実証されていない方法です。
