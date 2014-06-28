---
layout: doc_ja
title: Generator Stage
previous: AST
previous_url: bytecode-compiler/ast
next: Encoder Stage
next_url: bytecode-compiler/encoder
review: true
---

MelbourneはASTを作成すると、
それを入力値として次のステージ（Generator）を実行します。

ジェネレータステージでは`Rubinius::Generator`のインスタンスが作られ、ASTの
ルートにこの `Generator` オブジェクト上でそのバイトコード表現を生成してもらい
ます。

`Generator`はRubiniusのバイトコードを生成するための純粋にRubyなDSLを提供しま
す。その中心は、[Rubinius Instructions](/doc/ja/virtual-machine/instructions/)
に直接マップされるメソッド郡です。例えば、スタックにnilを積む命令は、
`Generator`インスタンスの`push_nil`メソッドを呼ぶことで作成できます。

`Generator`クラスには他にも多くの便利なメソッドが定義されていて、
よくあるパターンのバイトコードはRubiniusの命令セットの
特に非常にローレベルな部分を扱わなくてもに生成できるようになっています。

例えば、直接Rubiniusのバイトコードを使ってメソッドを呼ぶには、
まずメソッド名を表すリテラルを作る必要があります。
そして、メソッドを呼ぶために`send_stack`を呼びます。
さらに、もしプライベートメソッドを実行したい場合は、
先にプライベートメソッドをの実行を特別に許可するためのバイトコードを生成します。

プライベートメソッドの実行を許可した上で`puts`メソッドを引数無しで呼びたい場合、
以下のように書きます：

    # ここで、 g は Generator のインスタンス
    g.allow_private
    name = find_literal(:puts)
    g.send_stack name, 0

これは、`send`ヘルパーを使うともっと簡単に書くことができます：

    g.send :puts, 0, true

あるASTについてバイトコードを生成する際、
RubiniusはASTの各ノードについて`bytecode`メソッドを呼びます。
この時、引数として現在の`Generator`インスタンスを渡します。
これは、`if`ノードの bytecode メソッドです：

    def bytecode(g)
      pos(g)

      done = g.new_label
      else_label = g.new_label

      @condition.bytecode(g)
      g.gif else_label

      @body.bytecode(g)
      g.goto done

      else_label.set!
      @else.bytecode(g)

      done.set!
    end

まず、`pos`メソッドを呼びます。
これは`Node`クラスのメソッドで`g.set_line @line`を実行します。
これは、VMがデバッグ用の情報を実行時に提供するのに使われます。

次に、`Generator`の`label`ヘルパーを使っています。
純粋な Rubiniusバイトコードは、
少数のgoto命令(`goto`、`goto_if_true`、`goto_if_false`)を除き、
一切の制御構造を持ちません。
`goto_if_true`のかわりに`git`、`goto_if_false`のかわりに`gif`、
と短く書くことができます。
ここでは、２つの新しいラベルを作成しています。
１つは`if`条件の終わり、もう１つは`else`ブロックの開始地点を示しています。

２つのラベルが作成されると、`if`ノードがその子ノードである`@condition`について
現在の`Generator`オブジェクトを引数として`bytecode`メソッドを呼びます。
これにより、この条件式のバイトコードが現在のストリームに出力されます。

この処理はこの条件式の値をスタックに残すので、
`if`ノードは`else_label`にすぐ飛ぶための`goto_if_false`命令を出力します。
そして、上で見たのと同じやり方で`@body`のバイトコードを現在のストリームに出力したら、
条件式の最後に飛ぶための無条件の`goto`命令を出力します。

次に、`else_label`の場所をセットする必要があります。
ラベルの作成と使用が分離されているので、
ラベルオブジェクトの場所を指定する前に`goto`命令に渡すことができます。
この性質は、いろいろな制御構造を作る際に重要になります。

それから、`@else`ノードのバイトコードを出力し、`done`ラベルの場所を指定します。

この処理はルートノードから始まり再帰的にAST全体について実行され、
結果として`Generator`オブジェクトにASTのバイトコード表現が生成されます。

`lib/compiler/ast`ディレクトリ内に全てのASTノードとそのbytecodeメソッドが定義
されているので、参照すると役立つと思います。
`Generator API` の実用的な例としてもいいと思います。

`Generator`はASTのバイトコード表現を生成し終えたら、次のステージを実行します。
エンコーダステージです。

## 参照されているファイル

* *lib/compiler/generator_methods.rb*: Rubinius 命令セットのRubyによるラッパー
  が生成されています。
  これらのメソッドは各命令（[Rubinius Instructions](/doc/en/virtual-machine/instructions/)）
  に直接マップされています。
* *lib/compiler/generator.rb*: `Generator` オブジェクトの定義です。
  このクラスは純粋なジェネレータメソッドに加えて、
  よくあるバイトコードのパターンを生成する多数の抽象度の高いAPIを定義しています。
* *lib/compiler/ast*: パーサステージで生成される全てのASTノードの定義があります。

## カスタマイズする

ジェネレータステージをカスタマイズする一番簡単な方法は、
`Generator`にデフォルトで定義されている一般的なものに加えて、
抽象度の高いメソッドをさらに定義することです。

使われるジェネレータクラスを変更することもできます。
コンパイラの各ステージやパイプラインをカスタマイズする方法についてさらに知るには、
[コンパイラパイプラインのカスタマイズ](/doc/ja/bytecode-compiler/customization/)
を参照して下さい。
