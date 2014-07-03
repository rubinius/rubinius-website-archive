---
layout: doc_ja
title: Packager Stage
previous: Encoder Stage
previous_url: bytecode-compiler/encoder
next: Writer Stage
next_url: bytecode-compiler/writer
---

EncoderステージでGeneratorオブジェクトが適切にエンコードされたら、
Rubiniusは新しいCompiledCodeオブジェクトを作り、そのたくさんのプロパティを設定して、
バイトコードを CompiledCode オブジェクトとしてパッケージします。

以下のプロパティは全てのCompiledCodeオブジェクトにあります。
`executable`メソッドを呼ぶと、
RubyのメソッドオブジェクトからCompiledCodeオブジェクトを取り出すことができます。

* *iseq*: 生の命令列が入っているTupleオブジェクト
* *literals*: このメソッドで使われている全てのLiteralオブジェクトが入っている
  Tupleオブジェクト。
  リテラルオブジェクトはRubiniusが内部で文字列の値などに使用していて、
  `push_literal` 命令と `set_literal` 命令に使われます。
* *lines*: バイトコードで表現される各行について、最初の命令へのポインタが入っている配列。
* *required_args*: メソッドが必要とする引数の数。
* *total_args*: 省略可能な引数を含む、全ての引数の数。
  ただし、`*args`は含みません。
* *splat*: splat引数があれば、その位置。
* *local_count*: ローカル変数の数。引数も含みます。
* *local_names*: 全てのローカル変数名が入っているTuple.
  必須な引数、省略可能な引数、splat引数、ブロック引数の順になります。
* *file*: ファイル名。スタックトレースやその他のデバッグ情報に用いられます。
* *name*: メソッド名
* *primitive*: プリミティブの指定があればその名前
* metadata: コンパイルされたメソッドに追加で構造的なメタデータを持たせることができます。
  元のジェネレータがブロックのために作られた場合には、
  コンパイルされたメソッドは `for_block` というメタデータを持ち、
  その値は `true` です。

Packagerステージでは、全ての子ジェネレータ（メソッドやブロックなど）もまた
CompiledCode化されます。
これらの子CompiedCodeは親CompiledCodeのliterals内に入ります。

Generatorが自身をCompiledCodeとしてパッケージし終えたら、
それを入力値としてWriterステージに移行します。

## 参照されているファイル

* *kernel/bootstrap/compiled_code.rb*: CompiledCodeの基本実装です。
  主に、primitivesの配線をしています。
* *kernel/common/compiled_code.rb*: CompiledCodeのもっと堅牢な実装です。
  primitiveなメソッドと、Rubyで書かれたメソッドが組み合わされています。
* *vm/builtin/compiledcode.cpp*: CompiledCodeのprimitivesが、C++で実装されています。
* *lib/compiler/generator.rb*: `Generator`オブジェクトから情報を取り出して
  `CompiledCode`オブジェクトに入れる、`package`メソッドの実装です。

## カスタマイズする

`package` メソッドは、一般的には、CompiledCodeオブジェクトの変数郡を充填させ
るために設計されていますが、同じインターフェースを使ってPackagerを
他のオブジェクトに情報を入れるために使うこともできます。
しかし、追加のカスタマイズをさらに行わないと、そのままではあまり便利ではないかもしれません。
