---
layout: doc_ja
title: Building Rubinius
previous: Requirements
previous_url: getting-started/requirements
next: Running Rubinius
next_url: getting-started/running-rubinius
---

あなたはソースディレクトリからRubiniusをビルドし、実行することができます。
インストールは不要です。以下ではRubiniusのインストールとソースディレクトリ
からの実行の両方について詳しく説明します。

RubiniusはJITコンパイラにLLVMを使用し、LLVMのバージョン3に依存しています。
`configure`スクリプトはLLVMの存在を自動でチェックし、なければダウンロードします。
もしインストール済みのLLVMとのリンクに失敗する場合、`--skip-system`を
`configure`スクリプトに渡してください。

### ソースコードの入手

Rubiniusのソースコードはtarballとして、またGithubのプロジェクトとして入手可能です。
tarballは[こちらからダウンロード](https://github.com/rubinius/rubinius/tarball/master)することができます。

Gitを使う場合、

  1. 開発用のディレクトリに移動し、
  2. `git clone git://github.com/rubinius/rubinius.git`


### Rubiniusをインストールする

もしあなたが自身のアプリケーションをRubinius上で実行することを計画しているのであれば、
インストールは良い選択です。しかし、ソースディレクトリからもRubiniusを実行することができます。
詳細は次のセクションを確認してください。

Rubiniusを`sudo`や管理者特権が必要ない場所にインストールすることをお勧めします。Rubiniusをインストールするには、

  1. `bundle install`
  2. `./configure --prefix=/path/to/install/dir`
  3. `rake install`
  4. Follow the directions to add the Rubinius _bin_ directory to your PATH


### ソースディレクトリからの実行

Rubinius自身を開発したいなら、こちらを選ぶ必要があります。

  1. `./configure`
  2. `rake`

ただRubiniusを試してみたいだけであれば、_bin_ディレクトリをPATHに追加する指示に従ってください。

ただし、Rubiniusを開発しているのであれば、_bin_ディレクトリをPATHに追加するべきではありません。
RubiniusのビルドシステムがRubiniusの`ruby`や`rake`を使おうとしてしまいます。
Rubiniusはビルドの際にブートストラップするために、別のRubyの実行可能ファイルを必要とします。

### Development Mode for Debugging

もしあなたがVMをデバッグしていてGDBのようなデバッガをアタッチしたいなら、
Rubiniusを最適化なしでコンパイルしたいと思うでしょう。
そのためには、'DEV'環境変数をセットしてからビルドする必要があります。

例えば: `DEV=1 rake build`
