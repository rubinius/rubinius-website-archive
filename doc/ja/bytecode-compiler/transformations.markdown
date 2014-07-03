---
layout: doc_ja
title: Transformations
previous: Writer Stage
previous_url: bytecode-compiler/writer
next: Customizing the Pipeline
next_url: bytecode-compiler/customization
---

バイトコードコンパイラはASTを変形するためのシンプルな仕組みを備えていて、
特定の形のASTを認識して置換します。
置換されたASTは元のASTとは違うbytecodeを出力します。
この変形のソースコードは `lib/compiler/ast/transforms.rb` にあります。

TODO: コンパイラプラグインのアーキテクチャについてドキュメントを書く。


### 安全な計算のためのコンパイルによる変換(Safe Math Compiler Transform)

コアライブラリは他のRubyコードと同じブロックで構成されています。
Rubyは動的な言語で、クラスのメソッド定義等をあとから変更することができ(open class)、
実行時の定義が使用されるので(latebinding)、Fixnumなどの基礎となるクラスに
他のクラスが期待するセマンティクスを破壊するような変更を加えることが可能です。
例えば、以下の様な場合を考えます：

  class Fixnum
    def +(other)
      (self + other) % 5
    end
  end

固定小数点数の足し算を５を法とする剰余計算(modulo 5)に定義しなおすことは確かに
可能ですが、このようなことをすると、
例えば、Array等が必要な時に正しい長さを計算することができなくなるでしょう。
Rubyの動的さはとても慈しまれている性質ですが、
まさに諸刃の剣となる場合もあります。

標準ライブラリの 'mathn' は Fixnum#/ を不適合で安全でない振る舞いに再定義しています。
このライブラリは Fixnum#/ を Fixnum#quo のエイリアスとしていて、
これはデフォルトでは Floatオブジェクトを返します。

このため、特別なコンパイラプラグインを用意して、#/メソッドがあったら
違うメソッド名が出力されるようになっています。
コンパイラは #/ でなく #divide を出力します。
数値のクラス Fixnum, Bignum, Float, Numeric はすべてこのメソッドを定義しています。

この安全な計算のための変換は、コアライブラリのコンパイル中は有効になっています。
通常のユーザコードのコンパイルでは、このプラグインは有効になっていません。
この仕組みにより、コアライブラリを壊したり不便な手順を強制したりせずに、
mathnライブラリをサポートすることができます。
