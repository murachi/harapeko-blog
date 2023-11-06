[author: murachi]
# MinGW セットアップ

## ダウンロード

* [Browse MinGW - Minimalist GNU for Windows Files on SourceForge.net](http:://sourceforge.net/projects/mingw/files/)

上記サイトよりダウンロードする。

バージョンは **5.1.6** を使用する<small> (開発途中で変更する場合有り。基本的に最新版を用いるものとする。開発メンバーには都度アナウンスする。) </small>。

### 最低限ダウンロードするもの

* MinGW インストーラ (「Download Now\」でダウンロードできるもの)
* MSYS インストーラ (MSYS Base System/msys-1.0.11/MSYS-1.0.11.exe)
  * インストーラが存在する最も最新のバージョンを選択すること。

## インストール

### MinGW

1. インストーラ (MinGW-x.y.z.exe) を起動する。
1. "Download and Install" -> "Current" -> "Full Install" を選択する。
1. あとはよしなに。

### MSYS

1. インストーラ (MSYS-1.0.11.exe) を起動する。
1. ダイアログのウィザードではよしなに。
1. コマンドプロンプトが表示され、いくつか質問されるので、"y" -> "y" -> MinGW インストールディレクトリ (通常は "c:/MinGW") と入力する。

### GCC 4.4.0

そのままでは GCC のバージョンが 3.4.5 と古いので、より新しい GCC 4.4.0 を上書きインストールする。手順は以下の通り。

1. ダウンロードサイトにて、「MinGW」→「BaseSystem」→「GCC」→「Version4」→「Previous Release gcc-4.4.0」の順に開く。
1. 「gcc-full-4.4.0-mingw32-bin-2.tar.lzma」をクリックして、ダウンロードする。
1. MSYS を起動し、手順2 でダウンロードしたファイルがあるディレクトリに移動する。
  * ダウンロードフォルダのパスが "C:\download\mingw\gcc" の場合、
    ```
    cd /c/download/mingw/gcc
    ```
    で移動できる。
1. 以下のコマンドをタイプし、書庫ファイルを展開する。 bin や lib などのディレクトリがその場にごろっと展開される。
   ```
   tar -xvf gcc-full-4.4.0-mingw32-bin-2.tar.lzma --lzma
   ```
1. 展開されたディレクトリを、mingw のインストールディレクトリ下 (通常は C:\MinGW) に全てコピーする。

## 追加インストール

他に必要なコマンドやライブラリ (ex: liviconv) があれば、上記 URI のサイトの下の方から必要なパッケージを探してダウンロードし、インストールすることができる。その手順は概ね以下の通りとなる。

1. インストールしたいパッケージのバイナリが入った lzma 圧縮書庫ファイル (ファイル名が "-bin.tar.lzma" で終わっているもの) を、任意のディレクトリにダウンロードする。
1. MSYS を起動する ("スタート" -> "MinGW" -> "MSYS" -> "MSYS")。
1. 1 でダウンロードしたディレクトリに移動する。例えば C:\download\MinGW\hoge の場合、以下の通り。
   ```
   $ cd c:/download/MinGW/hoge
   ```
1. 1 でダウンロードしたファイルを展開する。ファイル名が hoge-1.0-1-mingw32-bin.tar.lzma の場合、以下の通り。
   ```
   $ tar -xvf hoge-1.0-1-mingw32-bin.tar.lzma --lzma
   ```
1. 展開されたファイルを、然るべき場所に移動する。概ね以下のようにすればいいはず。
   ```
   $ mv bin/* /bin/
   $ mv share/* /share/
   ```

## 参考リンク

* [MinGWによるWindowsにおけるインストール方法 - JUnNetHack project Wiki - SourceForge.JP](http:://sourceforge.jp/projects/junnethack/wiki/MinGW%E3%81%AB%E3%82%88%E3%82%8BWindows%E3%81%AB%E3%81%8A%E3%81%91%E3%82%8B%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E6%96%B9%E6%B3%95)
* [LZMA圧縮されたファイルの扱いについて - 試験運用中なLinux備忘録](http:://d.hatena.ne.jp/kakurasan/20080511/p1)
