---
layout: doc_ja
title: Troubleshooting
previous: Running Rubinius
previous_url: getting-started/running-rubinius
next: Contributing
next_url: contributing
translated: true
---

以下ではRubiniusのビルドやインストール、実行の際に直面する可能性のあるエラーについて、
推奨される解決策とともに説明しています。

どんな場合でも、最初にすべきことは最新でクリーンなRubiniusをチェックアウトしていることを
確認することです。先に進む前に、以下を実行してください。


    $ git checkout master
    $ git reset --hard
    $ git pull
    $ rake distclean
    $ rake

### Rubiniusが`runtime`ディレクトリを発見できない

  ビルドまたはインストールのあと、Rubiniusを実行しようとすると以下のエラーが起こる。

    ERROR: unable to find runtime directory

    Rubinius was configured to find the runtime directory at:

      /Users/brian/devel/rubinius/runtime

    but that directory does not exist.

    Set the environment variable RBX_RUNTIME to the location
    of the directory with the compiled Rubinius kernel files.

    You may have configured Rubinius for a different install
    directory but you have not run 'rake install' yet.

#### 解決策

  Rubiniusを`--prefix`オプション付きで設定したなら、`rake install`を実行します。

  Rubiniusを`--prefix`オプション付きで設定したあとでインストール先のディレクトリを
  インストール後にリネームしたなら、Rubiniusを再設定して再インストールします。

  Rubiniusのビルド後にソースディレクトリをリネームしたなら、再設定してリビルドします。

  一般的に、ビルド後またはインストール後にリネームを行わないでください。


### FreeBSDでビルドする際にセグメンテーション違反が起こる

  バージョン8.1stableを含めたFreeBSDで、execinfoがらみの問題が
  Rubiniusのロード時にセグメンテーション違反を引き起こす。

#### 解決策

  設定時にexecinfoを無効化します。

    ./configure --without-execinfo
    
### ruby-buildでのインストールが失敗する

  時折、[ruby-build](https://github.com/sstephenson/ruby-build)でRubiniusをインストールする際に
  問題が発生する。
  
#### 解決策

  依存関係を確認後、自身でRubiniusをインストールしてください。
  
    $ git clone https://github.com/rubinius/rubinius
    $ cd rubinius
    $ ./configure --prefix=/path/to/rubinius/location
    $ rake install
    
  `--prefix`オプションを指定する必要はありません。特定のディレクトリに
  インストールしたい場合のみ指定してください。
  RubiniusをPATHの通ったディレクトリに置くのがいいでしょう。
