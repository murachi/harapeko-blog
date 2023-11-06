[author: murachi]
# Boost ライブラリ - バージョン間の差意について

ここでは、 Boost ライブラリのバージョン間の差意について、 otoco の開発に関係がありそうな範囲で pick up する。

## 正規表現関連

* 1.38.0 にて、以下の修正が行われている。
  * パターンが空文字列の場合、「常にマッチする」ように修正された。[これ以前のバージョンでは、例外が発生していた](https:://svn.boost.org/trac/boost/ticket/1081)。
  * 正規表現内で記述できるキャプチャ変数 $1, $2, ... について、 ${1}, ${2}, ... といった記法が許容されるようになった。
  * Added support for accessing the location of sub-expressions within the regular expression string (issue \#2269).
    * ごめんちょっとよく分からなかった。[不具合報告? の内容...](https:://svn.boost.org/trac/boost/ticket/2269)
  * コンパイラ互換性に関する修正。
