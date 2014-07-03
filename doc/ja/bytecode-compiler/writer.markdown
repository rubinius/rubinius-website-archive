---
layout: doc_ja
title: Writer Stage
previous: Packager Stage
previous_url: bytecode-compiler/packager
next: Transformations
next_url: bytecode-compiler/transformations
---

PackagerがCompiledCodeを作成したら、
Rubiniusはあとで利用できるようにメソッドをファイルに書き出します。
例えば、最初にファイルがrequireされたあと、次のrequireからはRubyコードではなく
書き出されたファイルの方をロードし、パースしコンパイルします。

このステージは、非常にシンプルです。
もともとのファイル名を受け取って、`c`を最後につけます。
そして、Rubinius::CompiledFile.dumpを、前段階のCompiledCodeオブジェクトと
書き出し先のファイル名を引数として、実行します。

ファイルをディスクに書き出し終わったら、
渡された入力値（CompiledCodeオブジェクト）を返します。
これはコンパイル処理全体の返り値になります。

## 参照されているファイル

* *lib/compiler/compiled_file.rb*: CompiledFileの実装です。
  実際に書き出しを行うには、`CompiledFile.dump`を呼びます。 

## カスタマイズする

このステージは実際には任意で、コンパイル済みファイル作りたい時のみ使われます。
文字列をeval等でコンパイルしたい場合には、この処理はスキップされます。
その場合、コンパイラはPackagerステージで止まり、
そこで作られたCompiledCodeをコンパイラの返り値として返します。

Rubiniusコンパイラのアーキテクチャでは、
このステージのあとにさらにステージを追加することは簡単にできます。
追加されたステージが渡されたCompiledCodeオブジェクト
（もしくは違うCompiledCodeオブジェクト）を返すようになっていれば、
全て期待通りに動くでしょう。

さらに情報を得るには、
[コンパイラパイプラインのカスタマイズ](/doc/ja/bytecode-compiler/customization/)
を読んでみて下さい。
