[author: murachi]
# swfmill を用いた Flash 開発チュートリアル

## swfmill とは?

Flash の SWF ファイルと、データ記述言語 XML との相互変換を行うフリーソフトウェアのツールです。

* [swfmill swf2xml and xml2swf](http:://swfmill.org/)

## インストール

上記リンク先から自身のマシン環境にあった書庫ファイルをダウンロードして展開し、中の swfmill(.exe) ファイルを、環境変数 PATH に設定されているディレクトリへコピーします。

Linux 等の一部のディストリビューションにおいては、ディストリビューションが提供するセントラルリポジトリからインストールすることも出来るかもしれません。

## 使ってみよう

ここでは基本的に、 swfmill と MTASC を組み合わせて利用する方法について説明します。

### 画像ファイルを取り込む

swfmill では、 JPEG/PNG/SVG 画像ファイルを画像リソースとして取り込んだ SWF ファイルを作成することができます。まずは、以下の XML ファイルを作成し、 test.xml ファイルとして保存してください。

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<movie width="320" height="320" framerate="15">
  <background color="#000000"/>
  <frame>
    <library>
      <clip id="fig_kuma" import="kuma.png" />
    </library>
  </frame>
</movie>
```

画像ファイルは以下の画像をダウンロードして使いましょう。

![image](kuma.png)

test.xml ファイルと kuma.png ファイルを同一ディレクトリに保存したら、ターミナル (またはコマンドプロンプト) を起動し、 cd コマンドを使ってこれらのファイルがあるディレクトリに移動してから、以下のコマンドを入力してください。

```
$ swfmill simple test.xml test.swf
```

これで、画像ファイル kuma.png の内容が画像リソースとして取り込まれた test.swf ファイルが生成されます。

しかし、これだけでは test.swf ファイルを開いても画像は表示されません。画像を表示するには、実際に画像を表示するスクリプトを書いて、 MTASC でコンパイルし直してあげる必要があります。そこで、今度は以下の ActionScript プログラムを記述し、 test.as ファイルとして保存してください。

```js
class Test {
    static var app:Test;

    function Test(mc:MovieClip) {
        // 画像リソースを割り当てたムービークリップを生成し、表示する
        mc.attachMovie("fig_kuma", "image_mc", 1);
    }

    static function main(mc:MovieClip) {
        app = new Test(mc);
    }
}
```

そうしたら、次に、ターミナル上で以下のコマンドを入力します。

```
$ mtasc -cp . -swf test.swf -main test.as
```

これで、 test.swf ファイルが更新され、取り込んでいた画像が表示されるようになりました。 test.swf ファイルを Web ブラウザなどで開いてみましょう。

それでは順を追って説明しましょう。まずは XML ファイルから。

```
<movie width="320" height="320" framerate="15">
```

<movie> タグの width 属性と height 属性は、それぞれ Flash ムービーの横幅と高さを指定します。この大きさで表示するとちょうどきれいに表示される、ぐらいの意味合いで捉えましょう。実際にブラウザ等で表示した場合、引き延ばされたりすると思いますが、その場合でも座標値 (320, 320) が画面の一番右下になる、という意味です。

framerate 属性は、その名の通り Flash ムービーのフレームレートを指定します。これらの指定は、 MTASC コマンドで -header オプションを使用する際に指定する 3つの値と同義になります。

```
  <background color="#000000"/>
```

<background> タグを用いることで、 Flash ムービーの背景を設定することができます。ここでは color 属性を使用し、黒を表す RGB 値 "#000000" を指定しています。

```
  <frame>
    <library>
      <clip id="fig_kuma" import="kuma.png" />
    </library>
  </frame>
```

<frame> タグは、 Flash ムービーのフレーム 1つ分を表す要素です。 <library> タグは、<frame> タグの中に記述することができ、画像などの情報をライブラリリソースとして登録するのに用いる要素です。 <clip> タグは、 <library> タグの中で、実際に取り込む画像などの情報を指示するのに用いるタグです。

<clip> タグの import 属性では、取り込む画像などの情報をファイルからインポートする場合に、そのファイル名を指定します。 id 属性は、取り込んだ画像などの情報に割り当てる ID 名です。 ActionScript プログラムからは、この ID 名を使用して画像などの情報を特定します。
