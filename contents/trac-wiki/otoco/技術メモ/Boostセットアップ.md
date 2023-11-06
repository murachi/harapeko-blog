[author: murachi]
# Boost セットアップ

## ダウンロード

* [Boost C++ Libraries (公式)](http:://www.boost.org/)
* [Boost C++ Libraries (日本向けコミュニティ)](http:://boost.cppll.jp/HEAD/)

上記いずれかのサイトよりダウンロードする。公式の場合は「DOWNLOADS」から、日本向けサイトの場合は「Getting Started」から。

バージョンは、**1.40.0** を使用する<small> (開発途中で変更する場合有り。開発メンバーには都度アナウンスする。) </small>。

## インストール

### Windows の場合

あらかじめ、 Visual C++ 2008 [Express Edition](http:://www.microsoft.com/japan/msdn/vstudio/express/) がインストールされているものとして説明する。

1. 公式サイトより、任意の形式の書庫ファイルをダウンロードし、解凍する。
1. スタートメニューより、「すべてのプログラム」→「Microsoft C++ 2008 Express Edition」→「Visual Studio Tools」→「Visual Studio 2008 コマンド プロンプト」の順に辿り、選択する。
1. コマンドプロンプトが表示されるので、この中で、 1 で書庫を解凍してできたディレクトリへ移動し、 **bootstrap.bat** を実行する。
   ```
   C:\Program Files\Microsoft Visual Studio 9.0\VC>cd C:\download\dev-tool\boost\bo
   ost_1_39_0
   
   C:\download\dev-tool\boost\boost_1_39_0>bootstrap.bat
   ```
  * 1.34.1 では、 bootstrap.bat はまだ存在しない。その場合、以下の操作を行うこと。
    1. tools\jam\src ディレクトリへ移動し、 build.bat を実行する。
       ```
       C:\download\dev-tool\boost\boost_1_34_1>cd tools\jam\src
       
       C:\download\dev-tool\boost\boost_1_34_1\tools\jam\src>build.bat
       ```
    1. 生成される bin.ntx86 ディレクトリ下の bjam.exe を、 a で移動する前のディレクトリにコピーし、元のディレクトリへ移動する。
       ```
       C:\download\dev-tool\boost\boost_1_34_1\tools\jam\src>copy bin.ntx86\bjam.exe ..\..\..
       
       C:\download\dev-tool\boost\boost_1_34_1\tools\jam\src>cd ..\..\..
       ```
1. bjam.exe というファイルが生成されるので、それをさらに実行する。
   ```
   C:\download\dev-tool\boost\boost_1_39_0>bjam
   ```

### Ubuntu Linux の場合

aptitude を用いてインストールする。

```
$ sudo aptitude install libboost-dev
```

バージョン番号付きのパッケージも有るが (しかももっと新しい…)、当面はこのパッケージを指定する。また、バージョンもそのバージョンで統一する。

## 利用

### Visual C++ からの利用

Visual Studio の IDE から利用する場合は、あらかじめオプション設定でインクルードディレクトリとライブラリディレクトリを指定しておく。手順は以下の通り。

1. Visual C++ 2008 を起動する。
1. 「ツール」→「オプション」メニューを選択し、オプションダイアログを表示する。
1. 設定ツリーの「プロジェクトおよびソリューション」→「VC++ ディレクトリ」を選択する。
1. 「ディレクトリを表示するプロジェクト」コンボボックスから「インクルード ファイル」を選択する。
1. リストの空行を 2回クリックし、 boost をインストールしたディレクトリの絶対パス (ex: C:\libs\boost_1_39_0) を入力する。
1. 「ディレクトリを表示するプロジェクト」コンボボックスから「ライブラリ ファイル」を選択する。
1. リストの空行を 2回クリックし、 boost をインストールしたディレクトリ下の stage\lib ディレクトリへの絶対パス (ex: C:\libs\boost_1_39_0\stage\lib) を入力する。
1. ダイアログの「OK」ボタンをクリックして、ダイアログを閉じる。

これで、プロジェクト側に特別な設定の追加は必要なくなる。

nmake を使用する場合など、 cl.exe コマンドを直接呼んで利用する場合は、環境変数を事前に設定しておくか、またはコマンドラインオプションにインクルードパスやライブラリパスを指定する。

環境変数を利用する場合は、 INCLUDE に boost インストールディレクトリの絶対パス、 LIB に stage\lib ディレクトリへの絶対パスをそれぞれ追加する。

```
C:\Program Files\Microsoft Visual Studio 9.0\VC>cd C:\Developer\original\test

C:\Developer\original\test>set INCLUDE=C:\download\dev-tool\boost\boost_1_39_0;%
INCLUDE%

C:\Developer\original\test>set LIB=C:\download\dev-tool\boost\boost_1_39_0\stage
\lib;%LIB%

C:\Developer\original\test>cl /EHsc /MD test.cpp
```

コマンドラインオプションのみで解決する場合は、以下のような書式になる。

```
cl.exe /EHsc /MD /I(インクルードディレクトリへのパス) (ソースファイル) /link /LIBPATH:(ライブラリディレクトリへのパス)
```

サンプル:

```
C:\Developer\original\test>cl /EHsc /MD /IC:\download\dev-tool\boost\boost_1_39_
0 test.cpp /link /LIBPATH:C:\download\dev-tool\boost\boost_1_39_0\stage\lib
```

### Ubuntu Linux における gcc からの利用

g++ コマンドにてコンパイルする。その際、使用するライブラリの名称を明示する必要がある。例えば Boost.Regex を利用している場合、以下のように -l オプションを指定する。

```
g++ test.cpp -lboost_regex
```

このとき -l オプションに指定する名称は、実際にインストールされているライブラリファイル (Ubuntu の場合、通常は /usr/lib にインストールされる) のファイル名が libhogefuga.a ならば、前置詞 lib と拡張子 .a を取り除いた「hogefuga」が指定すべき名称ということになる。



[[FootNote]]
