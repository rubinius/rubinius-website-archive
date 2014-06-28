---
layout: doc_ja
title: Bytecode Compiler
previous: Virtual Machine - Custom Dispatch Logic
previous_url: virtual-machine/custom-dispatch-logic
next: Parser
next_url: bytecode-compiler/parser
review: true
---

RubiniusのバイトコードコンパイラはRubyのソースコードをバーチャルマシンによって
実行されるバイトコードに変換します。
コンパイル処理は一続きの複数のステージで構成されていて、
入力値をバーチャルマシンが理解できる形式に変換していきます。

各ステージは自身の以降の処理からは切り離されていて、
特定の形式の入力のみを期待し、次のステージに結果を渡します。
このため、コンパイル処理を柔軟に設定できますし(configurable)、
各ステージにおいて簡単に情報を得ることができます。

各ステージは入力を受け取り、担当する処理を行い、結果を次のステージに渡します。
下の図は、デフォルトで用意されているステージとその各ステージの入力と結果です。

<div style="text-align: center; width: 100%">
  <img src="/images/compilation_process.png" alt="Compilation process" />
</div>

1. [Parser Stage](/doc/ja/bytecode-compiler/parser/)
1. [AST](/doc/ja/bytecode-compiler/ast/)
1. [Generator Stage](/doc/ja/bytecode-compiler/generator/)
1. [Encoder Stage](/doc/ja/bytecode-compiler/encoder/)
1. [Packager Stage](/doc/ja/bytecode-compiler/packager/)
1. [Writer Stage](/doc/ja/bytecode-compiler/writer/)
1. Printers
1. [Transformations](/doc/ja/bytecode-compiler/transformations/)
1. [パイプラインのカスタマイズ](/doc/ja/bytecode-compiler/customization/)
