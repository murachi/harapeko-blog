[author: murachi]
# Motion-Twin ActionScript 2 Compiler (MTASC) を用いた Flash 開発チュートリアル

## MTASC とは?

フランス [Motion-Twin 社](http:://www.motion-twin.com/japanese)が開発したフリーの ActionScript 2 コンパイラです。 C コンパイラなどと同様、コマンドラインツールとして実行し、スクリプトファイルや画像ファイルなどから Flash 形式のファイル (.swf ファイル) を生成することができます。

* [MTASC 公式サイト](http:://www.mtasc.org/)

## MTASC のインストール

### Windows の場合

基本的に、書庫ファイルを展開して、PATH を通せば ok 。

1. [公式サイト](http:://www.mtasc.org/#download)から、「Windows precompiled binary, zipped」と題された zip ファイルをダウンロードする。
1. ダウンロードした zip ファイルを適当な場所 (C:\ など) に展開する。
  * 「mtasc-1.14」といった名前のディレクトリが展開される。以下、展開されたディレクトリパスを「C:\mtasc-1.14」として説明。
1. 展開されたディレクトリパスを環境変数 PATH に追加する。
  1. コントロールパネルの「システム」をダブルクリック (またはマイコンピュータを右クリックしてコンテキストメニューより「プロパティ」を選択)。
  1. 「システムのプロパティ」ダイアログにて「詳細設定」タブを選択し、「環境変数」ボタンをクリックする。
  1. 「環境変数」ダイアログの「システム環境変数」リストから「Path」変数を選んでダブルクリック。
  1. 変数値の末尾に、「;C:\mtasc-1.14」を追加し、「OK」をクリックする。「環境変数」ダイアログも「OK」をクリックする。

### Mac OS X の場合

こちらも基本的には同様ですが、 Unix 的な流儀に則っておいた方が良いでしょう。

1. [公式サイト](http:://www.mtasc.org/#download)から、「Mac OSX package」と題された zip ファイルをダウンロードする。
1. ダウンロードした zip ファイルを適当な場所に展開する。
  * とりあえずホームディレクトリの直下にでも展開してください。
1. root 権限で、展開されたディレクトリの中身全てを /usr/local/bin 以下にコピーする。
   ```
   $ sudo cp -R ~/mtasc-1.12-osx/* /usr/local/bin
   ```
  * /usr/local/bin ディレクトリが存在しなかった場合は、コピーする前にディレクトリを作成してしまえばよい。 PATH は通っているハズ…。
    ```
    $ sudo mkdir -p /usr/local/bin
    ```

### Linux の場合

各ディストリビューションのパッケージ管理システムに則った方法でインストールしてください。

例えば、 Ubuntu であれば Ubuntu ソフトウェアセンターを起動し、右上の検索欄にて「MTASC」と入力すれば見つけられるはず。

## MTASC を使ってみる

### Hello World プログラム

UTF-8 に対応したテキストエディタにて、以下の内容のスクリプトファイルを作成しよう。ファイル名は、とりあえず test.as とでもしておく。

```js
class Test {

    static var app :Test;

    function Test() {
        _root.createTextField("tf", 0, 0, 0, 240, 240);
        _root.tf.text = "Hello World!";
    }

    static function main(mc) {
        app = new Test();
    }
}
```

### コンパイル

ではコンパイルしてみよう。コンパイルコマンドは、 Windows だろうと Mac だろうと Linux だろうと、以下の通りで ok 。

```
$ mtasc -swf test.swf -main -header 240:240:10 test.as
```

**-swf オプション** は、生成される .swf ファイルのファイル名を指定する。 **-main オプション**は、 main() という名前の関数をエントリポイントとして使用するよという意味。この関数は static 関数である必要がある。 **-header オプション**では 3つの数値を指定しているが、それぞれ、横幅、高さ、FPS (フレーム/秒) だ。横幅と高さはピクセル単位だが、実際にはいくらでも引き延ばして表示できるので、プログラム内で使用する座標との帳尻あわせと、縦横比率の目安の為に指定する、と考えればいいと思う。

さて、これによって test.swf ファイルが生成されたはずなので、早速ブラウザ等で表示してみよう。画面の上の方に「Hello World\」と表示されれば ok 。

### 簡単なアニメーションプログラム

さて、 ActionScript 2 および Flash Light 2.x のリファレンスについては、以下のサイトを辞書的に参照するとよい。

* [Flash CS3 ドキュメンテーション](http:://livedocs.adobe.com/flash/9.0_jp/main/wwhelp/wwhimpl/js/html/wwhelp.htm)

HTML+JavaScript の世界が HTML 文書を中心に回っているならば、 Flash+ActionScript の世界は Flash **ムービー** を中心に回っている。 Flash ムービーは複数の**ムービークリップ**を重ね合わせるような形で構成されており、レイヤーを分けて同時に複数のアニメーションを走らせることができるようになっている。

MTASC では、 -header オプションを用いて固定のフレームレートを指定することができるが、このフレームが切り替わるタイミングで呼ばれるメッセージハンドラメソッドが存在する。それが、 MovieClip.onEnterFrame() メソッドだ。基本的には、空っぽのムービークリップオブジェクトを作成し、そのオブジェクトのメンバフィールド onEnterFrame に、 JavaScript でやるのと同じ要領で、実際に描画を行う処理が記述された関数オブジェクトを代入しておけばよい。

以下は、画面内でランダムな 3頂点による三角形を絶え間なく表示し続けるプログラムのサンプルである。

```js
class Tuto {
    // 画面の大きさ
    static var MAX_HEIGHT = 240;
    static var MAX_WIDTH = 240;

    static var app:Tuto;

    var mc:MovieClip;

    function Tuto(parent_mc) {
        // 空のムービークリップを生成
        this.mc = parent_mc.createEmptyMovieClip("tuto_mc", 1);
        // フレームが切り替わるタイミングで呼ばれるメッセージハンドラを設定
        this.mc.onEnterFrame = function() {
            // ** この関数内の this は、関数の外側における this.mc と同じ意味になる。要注意! **
            // 画面をクリアする
            this.clear();
            // 塗りつぶしの開始
            this.beginFill(0xff4499);
            // 線のスタイルを設定
            this.lineStyle(2, 0x444444);
            // 線を描画
            this.moveTo(Math.floor(Math.random() * Tuto.MAX_WIDTH), Math.floor(Math.random() * Tuto.MAX_HEIGHT));
            this.lineTo(Math.floor(Math.random() * Tuto.MAX_WIDTH), Math.floor(Math.random() * Tuto.MAX_HEIGHT));
            this.lineTo(Math.floor(Math.random() * Tuto.MAX_WIDTH), Math.floor(Math.random() * Tuto.MAX_HEIGHT));
            // 塗りつぶしを終了; ここまでで描画した線の内部が塗りつぶされる
            this.endFill();
        };
    }

    // プログラムの開始点
    static function main(mc) {
        app = new Tuto(mc);
    }
}
```

このプログラムをファイル tuto.as として保存し、以下のコマンドでコンパイルすると、

```
$ mtasc -swf tuto.swf -main -header 240:240:10 tuto.as
```

this.mc.onEnterFrame に代入している関数が、毎秒 10 回呼び出される。その中での処理内容として、既に描画されている三角形を消し、塗りつぶしの色と線のスタイルを設定し、ランダムな位置と形で三角形を描画する、ということをやっている。

## デバッグする

MTASC に対応したデバッガは存在しないが、デバッグ版の Flash を使用することでログを出力することができる。また、このログを Firefox 上でリアルタイムにモニタするアドオンも存在する。

### デバッグ版の Flash をインストールする

デバッグ版の Flash プレイヤーは以下の場所にて配布されている。

* [Adobe Flash Player - Downloads](http:://www.adobe.com/support/flashplayer/downloads.html)

Firefox で使用するには、自身が用いている OS の「Plugin content debugger」と題されたリンクをクリックしてダウンロードし、インストールを行う。

### ログファイルのパス

ログファイルのパスは固定であり、変更できない。 OS によって出力先が異なり、以下のようになっている。

* Windows XP の場合: %HOMEDRIVE%%HOMEPATH%\Application Data\Macromedia\Flash Player\Logs\flashlog.txt
* Mac OS X の場合: /Users/(ユーザー名)/Library/Preferences/Macromedia/Flash Player/Logs/flashlog.txt
* Linux の場合: $HOME/.macromedia/Flash_Player/Logs/flashlog.txt

### Flash Tracer をインストールする

Firefox 上でログをリアルタイムに表示するアドオン Flash Tracer をインストールする。

* [Firefox extensions - Flash Tracer (2.3.1) - sephiroth.it](http:://www.sephiroth.it/firefox/flashtracer/)

リンク先ページの一番下にある「Install now」をクリックすると、インストールできずに黄色い帯の警告が表示されるので、「許可」ボタンをクリックしてインストールを強行する。

その後、 Firefox を再起動し、「ツール」メニューの「Flash Tracer」を選択すると、画面左に Flash Tracer ペインが出現する。その横幅をドラッグ操作で広げてあげると、右下隅にスパナーの形をしたアイコンが表示されるので、それをクリックすると設定画面が表示される。その中の「Select output file」と書かれた入力欄に、上述したログファイルのパスを入力し、「Default charset」に「UTF-8」を選択して「OK」ボタンをクリックすれば、準備完了だ。

### トレースログを出力してみる

以下のプログラムを作成し、コンパイルする。

```js
class Test {
    static var app:Test;

    function Test() {
        trace("Hello debug log!!");
    }

    static function Test(mc) {
        app = new Test();
    }
}
```

生成された swf ファイルを Firefox で読み込み、「ツール」メニューの「Flash Tracer」を選択して Flash Tracer を起動すると、以下のようなメッセージが表示されているはず。

```
Hello debug log!!
```

このように、**trace() 関数**を用いることによって、任意の文字列をログに出力することができる。
