[author: murachi]

# JavaScript チュートリアル

## JavaScript って何?

PC 上で動作する主要な Web ブラウザに必ずと言っていいほど搭載されているスクリプト言語です。

単体で動作する JavaScript インタプリタも存在しますが、主な用途は基本的には Web アプリケーションのクライアントサイドプログラムとなります。

### クライアントサイドプログラムとは?

CGI がブラウザ (クライアント) からの要求を受けてサーバーサイドでプログラムを動作させる仕組みであるのに対し、ブラウザ上、すなわちクライアントサイドでプログラムを動作させる仕組みのことをこう呼んでいます。

元々は、サーバーとの通信を必要としない動作 (画面表示の動的な変化、等) を記述する目的で使われていましたが、 Ajax と呼ばれる通信機能により、最近ではページ読み込みの待ち時間によるストレスを感じさせない UI 作りや、Google Map に代表されるようなリッチな UI の実装などに大きく貢献するようになってきています。

## とりあえず使ってみよう

### どうやって使うの?

JavaScript は HTML と一緒にブラウザに読み込ませて使います。従って、基本的には HTML ファイルの中に直接記述してしまいますが、別ファイルに JavaScript のスクリプトファイル単体として記述することも可能です。

まずは、最も簡単な例から見てみましょう。以下のサンプルプログラムは、ブラウザ上に「hoge」と書かれたボタンが表示され、それをクリックすると、メッセージボックスに「hoge」と表示する、というものです。

```text
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN">
<html>

<head>
  <meta http-equiv="content-type" content="text/html; charset=UTF-8">
  <title>hoge</title>
</head>

<body>

<form>
  <input type="button" value="hoge" onclick="alert('hoge');">
</form>

</body>

</html>
```

### 動かしてみる

上記のサンプルプログラムをテキストエディタに記述したら、拡張子 .html で保存し、そのファイルを Web ブラウザ (IE、Firefox、Safari、Opera、etc...) にドラッグ＆ドロップするなどして読み込ませれば動かせるはずです。

IE の場合、画面上部に警告メッセージの黄色い帯が表示されるかも知れませんが、その場合は帯の部分をクリックし、表示されるコンテキストメニューの「ブロックされているコンテンツを許可...」を選択してください。

### イベントハンドラ

上記のサンプルの中で、 JavaScript が用いられているのは、以下の部分です。

```text
  <input type="button" value="hoge" onclick="alert('hoge');">
```

<input> 要素に onclick という属性が存在し、その値として、 JavaScript によるプログラムが記述されています。

onclick 属性は、その要素がクリックされたときに、値に指定されたプログラムを実行せよ、という意味のものです。この手の属性は、他にも以下のようなものが存在します。(本当は他にももっといろいろ存在します…)

* onmousemove ... その要素が表示されている箇所でマウスカーソルが動いたら、プログラムを実行する。
* onmousedown ... その要素が表示されている箇所にマウスカーソルが存在するときに、マウスのボタンが押し下げられたら、プログラムを実行する。 (onclick が押して放さないと認識されないのに対し、こちらは押しただけで認識される)
* onmouseup ... その要素が表示されている箇所にマウスカーソルが存在するときに、マウスのボタンが放されたら、プログラムを実行する。
* onkeydown ... キーボードのキーが押されたら、プログラムを実行する。
* onkeyup ... キーボードのキーが放されたら、プログラムを実行する。
* onload ... HTML ファイルが読み込まれたら、プログラムを実行する。
* onfocus ... その要素にフォーカスが移ったら、プログラムを実行する。例えば、入力欄をクリックするなどしてカーソルが表示されるようになるタイミングで、その入力欄の <input> 要素に onfocus 属性があれば、その値が実行される。
* onchange ... コンボボックス (<select> 要素) の選択されている値が変化したら、プログラムを実行する。
* …などなど。

このような、「○○されたら××する」というプログラム上の仕組みのことを、**イベントハンドラ**と呼びます。多くの場合、 JavaScript はイベントハンドラから実行されるように記述します。

## <script> タグ

HTML 要素の属性値として JavaScript を記述することができるのは分かりましたが、これだけだと、より長くて複雑なプログラムを記述したい場合には不便です。

そこで、属性値の中ではなく、もうちょっと別の場所に、プログラムをまとめて記述する方法を以下に示します。以下のサンプルプログラムは、「hoge」ボタンを押すたびに、ブラウザ画面上の至る所に「hoge」という文字を表示する、というものです。

```text
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN">
<html>

<head>
  <meta http-equiv="content-type" content="text/html; charset=UTF-8">
  <meta http-equiv="content-style-type" content="text/css">
  <meta http-equiv="content-script-type" content="text/javascript">
  <title>hoge</title>
<script type="text/javascript" language="JavaScript"><!--
function putHoge() {
    var span = document.createElement("span");
    with (span.style) {
        position = "absolute";
        left = (Math.floor(Math.random() *
            (window.innerWidth ? window.innerWidth :
            document.documentElement.clientWidth ? document.documentElement.clientWidth :
            document.body.clientWidth))) + "px";
        top = (Math.floor(Math.random() *
            (window.innerHeight ? window.innerHeight :
            document.documentElement.clientHeight ? document.documentElement.clientHeight :
            document.body.clientHeight))) + "px";
    }
    span.appendChild(document.createTextNode("hoge"));
    document.getElementById("hoge_place").appendChild(span);
};
//--></script>
</head>

<body>

<form>
<input type="button" value="hoge" onclick="putHoge();">
</form>

<div style="position: absolute; left: 0; top: 0;" id="hoge_place"></div>

</body>

</html>
```

さて、先ほどのプログラムと比べると、今回のプログラムはかなり複雑です。今はプログラムの内容は気にせず、プログラムが記述されている場所と、その雰囲気に注目してください。

先ほどと同様、今回も <input> タグには onclick 属性が指定されています。

```text
  <input type="button" value="hoge" onclick="putHoge();">
```

しかし、そこに書かれているのは、 putHoge という名前の、恐らくは関数呼び出しに見える (後ろに丸かっこが付いているので) もののみが記述されています。

HTML ファイルの上の方に、なんだか見慣れない記述がありますね。以下の部分です。

```text
  <script type="text/javascript" language="JavaScript"><!--
function putHoge() {
    var span = document.createElement("span");
    with (span.style) {
        position = "absolute";
        left = (Math.floor(Math.random() *
            (window.innerWidth ? window.innerWidth :
            document.documentElement.clientWidth ? document.documentElement.clientWidth :
            document.body.clientWidth))) + "px";
        top = (Math.floor(Math.random() *
            (window.innerHeight ? window.innerHeight :
            document.documentElement.clientHeight ? document.documentElement.clientHeight :
            document.body.clientHeight))) + "px";
    }
    span.appendChild(document.createTextNode("hoge"));
    document.getElementById("hoge_place").appendChild(span);
};
//--></script>
```

この、 <script> タグに囲まれた部分も、 JavaScript プログラムです。そしてこのプログラム、

```js
function putHoge() {
```

で始まっていることから分かるように、先ほど <input> 要素の onclick 属性で呼び出そうとしていた putHoge() 関数の定義であることが、何となくおわかりいただけるかと思います。

このように、通常は <script> タグを用いて複雑な処理を行う**関数**を記述し、その関数をイベントハンドラから呼び出す、というスタイルで記述されるのが一般的です。

なお、上記の例では HTML 4.01 で記述している為、<script> タグの内側は HTML のコメントタグ (<\-- ～ -->) で包んでいます。これには以下の 2つの理由があります。

* <script> タグに対応していない古いブラウザや未知のブラウザでも、画面上に JavaScript のプログラムを表示してしまわないようにする為。
* JavaScript 中に出現する不等号記号 (<, >) が、 HTML タグと混同されてしまうのを防ぐ為。

一方、 XHTML で記述する場合には、 <script> タグの内側に記述するプログラムを HTML のコメントタグで包むことは禁止されています。 XHTML であれば、 <script> タグには必ず対応しているので、画面上に JavaScript のプログラムを表示してしまう危険性はありませんが、不等号記号が HTML タグだと誤解されるが故の誤動作の危険性は残っています。それを防ぐ為、 XHTML の場合は以下のように、コメントタグの代わりに <\[CDATA[ ～ ]]> で囲うようにする必要があります。

```text
  <script type="text/javascript" language="JavaScript">
// <![CDATA[
function putHoge() {
    // (...省略)
};
// ]]>
</script>
```

### JavaScript のみを別ファイルに記述する

<script> タグを用いれば、 JavaScript で記述された複雑な処理のプログラムを、 HTML ファイルとは別のファイルに記述して実行させることも可能です。

その場合、 HTML ファイルは以下のように記述します。

```text
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN">
<html>

<head>
  <meta http-equiv="content-type" content="text/html; charset=UTF-8">
  <title>hoge</title>
  <script src="hoge.js" type="text/javascript" language="JavaScript"></script>
</head>

<body>

<form>
  <input type="button" value="hoge" onclick="putHoge();">
</form>

<div style="position: absolute; left: 0; top: 0;" id="hoge_place"></div>

</body>

</html>
```

そして、 hoge.js ファイルは以下のような内容となります。といっても、前のサンプルで <script> タグの内側に記述していた内容と全く同じです。もちろん、 HTML コメントタグ <\-- ～ --> や CDATA セクション <\[CDATA[ ～ ]]> で囲う必要はありません。

```js
function putHoge() {
    var span = document.createElement("span");
    with (span.style) {
        position = "absolute";
        left = (Math.floor(Math.random() *
            (window.innerWidth ? window.innerWidth :
            document.documentElement.clientWidth ? document.documentElement.clientWidth :
            document.body.clientWidth))) + "px";
        top = (Math.floor(Math.random() *
            (window.innerHeight ? window.innerHeight :
            document.documentElement.clientHeight ? document.documentElement.clientHeight :
            document.body.clientHeight))) + "px";
    }
    span.appendChild(document.createTextNode("hoge"));
    document.getElementById("hoge_place").appendChild(span);
};
```

大規模な Web アプリケーションの場合、HTML 自体がサーバーサイドで動的に生成され、そこから JavaScript で記述された同一の処理を実行する、といった作りになることが多い為、このような方法で JavaScript のみを別ファイルで記述する方がより一般的です。

### 読み込みと同時に実行されるプログラム

<script> タグ内に記述したプログラムは、HTML ファイルの読み込み中に順次実行されます。その様子を、以下のサンプルプログラムで確認してみましょう。このプログラムは、ブラウザ上で HTML ファイルを読み込んだときの日時を表示するものです。

```text
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN">
<html>

<head>
	<meta http-equiv="content-type" content="text/html; charset=UTF-8">
	<meta http-equiv="content-script-type" content="text/javascript">
	<title>run in load</title>
</head>

<body>

<p>このファイルは、</p>

<p><strong>
  <script type="text/javascript" language="JavaScript"><!--
    document.write((new Date()).toLocaleString());
//--></script>
</strong></p>

<p>に読み込まれました。</p>

</body>

</html>
```

document.write() メソッドを用いることにより、 <script> タグを記述した箇所に直接文字列を挿入することができます。

が、こうした書き方は、実際にはあまりやりません。できることが限られているからです。 <script> タグは <head> の中に配置し、その中では変数や関数の定義を記述し、それらをイベントハンドラから呼び出して利用する、というスタイルがもっとも一般的です。


## JavaScript で Web ページが動く仕組み

### window オブジェクト

それでは、 JavaScript をどのように記述すると、どんな風に Web ページを動かすことができるのか、その仕組みを見ていくことにしましょう。

まずは、一番最初に示したサンプルプログラムを、もう一度見てみることにしましょう。

```text
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN">
<html>

<head>
  <meta http-equiv="content-type" content="text/html; charset=UTF-8">
  <title>hoge</title>
</head>

<body>

<form>
  <input type="button" value="hoge" onclick="alert('hoge');">
</form>

</body>

</html>
```

JavaScript によるプログラムが記述されているのは、以下の部分ですね。

```text
  <input type="button" value="hoge" onclick="alert('hoge');">
```

この <input> 要素は type="button" となっているとおりボタンであり、これをクリックすることで、 onclick イベントハンドラに指定されている以下の JavaScript プログラム

```js
  alert('hoge');
```

が実行される、という仕組みです。この alert() という関数に見えるものに、表示したい文字列を引数に渡して呼び出せば、その文字列が書かれたメッセージボックスを表示できるらしい、ということが分かると思います。

さて、この alert() ですが、これは**正確には、 window.alert() が正式名称**です。言い換えると、 window というオブジェクトが持つ alert() という名前の**メソッド**である、ということができます。

このプログラムは、もちろん以下のように書き換えても、同じように動作します。

```text
  <input type="button" value="hoge" onclick="window.alert('hoge');">  <!-- alert('hoge'); と同じこと -->
```

Web ブラウザ上で動作する JavaScript においては、 window は特別なオブジェクトで、 window オブジェクトが持っている変数 (属性) や関数 (メソッド) を用いる際には、「window.」という記述を省略することが許されています。

window オブジェクトには様々な役割の属性やメソッドがあらかじめ用意されています。それらは今後のチュートリアルで、追って紹介してゆくことにします。

それから、 JavaScript では、変数を使用する場合、 var 命令を用いて宣言する必要がありますが、

```js
  var num = 10;     // 変数 num を宣言し、値 10 で初期化する
```

var 命令を用いずにいきなり適当な名前の変数に代入を行ってしまった場合、

```js
  num = 10;         // 変数 num はまだ宣言されていないが…
```

通常エラーになることはありませんが、実際には以下のように、 window オブジェクトのメンバ変数に値を代入したものとして扱われるので、注意が必要です。

```js
  window.num = 10;  // さっきの num = 10 と同じ意味
```

### HTML を直接いじる

さて、ここからが本題です。メッセージボックスだけ出せても大したプログラムは作れませんので、実際に既に画面に表示されている内容を変更する方法について踏み込んでみることにしましょう。

JavaScript を用いて、 Web ブラウザ上の画面表示を変更するには、どうすればよいでしょうか? その答えの一つとして、 Web ブラウザが読み込んでいる HTML の内容を、 JavaScript から書き換えてしまう、という方法があります。

まず準備として、以下の HTML ファイルを見てください。

```text
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN">
<html>

<head>
  <meta http-equiv="content-type" content="text/html; charset=UTF-8">
  <meta http-equiv="content-script-type" content="text/javascript">
  <title>HTML look in test</title>
</head>

<body>

<p>これは、<strong>innerHTML プロパティ</strong>のテストです。</p>

</body>

</html>
```

この HTML ファイルは、部分的に強調表示を行ってはいるものの、文が 1行表示されるだけの非常に簡単なものです。このファイルの中に 1つだけ存在するパラグラフ、すなわち <p> タグの中に記述された HTML を JavaScript から参照するには、どうすればよいでしょうか?

その答えに当たるプログラムを、以下に示します。このプログラムは、先ほどの文の下にボタンが表示され、それをクリックすると、その文の HTML がメッセージボックスに表示されます。

```text
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN">
<html>

<head>
  <meta http-equiv="content-type" content="text/html; charset=UTF-8">
  <meta http-equiv="content-script-type" content="text/javascript">
  <title>HTML look in test</title>
  <script type="text/javascript" language="JavaScript"><!--
function lookInnerHtml() {
    var p = document.getElementById("main_paragraph");  // HTML を参照したいパラグラフを取得
    alert(p.innerHTML); // HTML をメッセージボックスに表示する
}
  //--></script>
</head>

<body>

<p id="main_paragraph">これは、<strong>innerHTML プロパティ</strong>のテストです。</p>

<form>
  <p><input type="button" value="HTML を表示" onclick="lookInnerHtml();"></p>
</form>

</body>

</html>
```

それでは解説していきましょう。まず、 HTML を参照しようとしている <p> 要素についてですが、

```text
<p id="main_paragraph">これは、<strong>innerHTML プロパティ</strong>のテストです。</p>
```

よく見ると、 <p> タグに **id 属性** が記述されています。こうすることで、個々の HTML 要素にユニークな ID を指定することができます。

これを踏まえて、ボタンをクリックしたときに呼び出される lookInnerHtml() 関数の実装を見てみましょう。

```js
function lookInnerHtml() {
    var p = document.getElementById("main_paragraph");  // HTML を参照したいパラグラフを取得
    alert(p.innerHTML); // HTML をメッセージボックスに表示する
}
```

**document オブジェクト** (言うまでもなく、正式名称は window.document です) は、Web ブラウザに読み込まれた HTML 文書を操作する際に用いるオブジェクトです。 window オブジェクトと同様、さまざまな機能があり、多くの属性とメソッドを持っていますが、その中でも document.getElementById() メソッドは、今後もっとも頻繁に利用するメソッドの一つです。このメソッドは、先ほどの id 属性に記述した ID で HTML 要素を検索し、その HTML 要素をオブジェクトとして取得する、というものです。

すなわち、

```js
    var p = document.getElementById("main_paragraph");  // HTML を参照したいパラグラフを取得
```

と記述することによって、変数 p に、「main_paragraph」という ID が設定された要素、すなわち先ほどの <p> タグが、 JavaScript 言語で扱うオブジェクトとなって代入される、という処理が行われることになります。

次に、いつもの alert() メソッドによってメッセージボックスを表示しようとしていますが、その引数に注目してください。

```js
    alert(p.innerHTML); // HTML をメッセージボックスに表示する
```

document.getElementById() メソッドなどによって取得された HTML 要素オブジェクトも、これまたさまざまな属性やメソッドを持っています。その中の一つ、 **innerHTML 属性**は、まさにその属性が内包する HTML そのものを、文字列として保持する、というものです。これによって、 <p> ～ </p> の間に挟まれている文の HTML が、メッセージボックスにそのまま表示される、というわけです。

それでは、逆にこの innerHTML 属性に対して文字列を代入した場合、何が起きるでしょうか? すでにご想像の通り、まさに、代入したとおりに HTML が書き換わり、その内容に合わせて画面の表示も変化します\!

以下のサンプルプログラムは、入力欄に HTML の文字列を入力し、ボタンを押すと、下の枠の中に、入力した HTML を Web ブラウザが解釈した結果が表示される、というものです。

```text
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN">
<html>

<head>
  <meta http-equiv="content-type" content="text/html; charset=UTF-8">
  <meta http-equiv="content-script-type" content="text/javascript">
  <title>HTML modify test</title>
  <script type="text/javascript" language="JavaScript"><!--
function displayHtml() {
    var textarea = document.getElementById("html_input");
    var view = document.getElementById("result_view");
    view.innerHTML = textarea.value;
}
  //--></script>
</head>

<body>

<form>
  <p><textarea id="html_input" cols="80" rows="5"></textarea></p>
  <p><input type="button" value="表示する" onclick="displayHtml();"></p>
</form>

<div id="result_view" style="width: 720px; height: 540px; border: 1px solid #000;"></div>

</body>

</html>
```

解説しましょう。まずは HTML で記述された部分。

```text
<form>
  <p><textarea id="html_input" cols="80" rows="5"></textarea></p>
  <p><input type="button" value="表示する" onclick="displayHtml();"></p>
</form>

<div id="result_view" style="width: 720px; height: 540px; border: 1px solid #000;"></div>
```

入力欄となる <textarea> 要素、および結果を表示する領域となる <div> 要素のそれぞれに、 id 属性による ID が設定されています。

<div> 要素には style 属性も設定されていますが、これは表示領域を固定の大きさの枠として表示する為のものです。

次に、ボタンをクリックしたときに呼び出される displayHtml() 関数の実装を見てみましょう。

```js
function displayHtml() {
    var textarea = document.getElementById("html_input");
    var view = document.getElementById("result_view");
    view.innerHTML = textarea.value;
}
```

最初の 2行で、 ID を設定していた 2つの HTML 要素のオブジェクトを取得し、それぞれ別個の変数に代入しているのが分かると思います。

そして、 3行目で <div> 要素の innerHTML 属性に、 <textarea> 要素の value 属性を代入しています。

```js
    view.innerHTML = textarea.value;
```

<textarea> 要素や <input> 要素、 <select> 要素といった、いわゆるフォーム部品を document.getElementById() などによって取得したオブジェクトには、ほとんどの場合 value という属性が用意されています。これは、その Web ページを閲覧しているユーザーが、実際にフォーム部品に入力した値を参照する属性です。この属性も、逆に代入することが可能で、その場合、代入された値でフォーム部品の表示が反映されることになります。

さておき、プログラムの内容が分かったところで、試しにこのプログラムを Web ブラウザに読み込ませて、入力欄に適当な HTML を入力し、「表示する」ボタンをクリックしてみてください。下の枠内に、その HTML が Web ブラウザに解釈された結果が表示されるのが分かると思います。1回だけではなく、入力欄の内容を書き換えては、何度でも「表示する」ボタンを試してみてください。クリックするたびに、下の枠内の表示が、書き換えた内容に応じて変化するのが分かると思います。

### DOM を扱う

実は、画面表示に関してのみいうのであれば、理論的には先ほどの innerHTML 属性だけであらゆる操作が可能です。そして、実際のところ、画面表示の操作は何でもかんでも innerHTML でやってしまおう、という考え方の Web プログラマーも少なくありません。

しかし、 innerHTML を用いる方法はあまりにも原始的である為、それを補う為には、プログラミングの際に以下の点に気をつける必要があります。

1. innerHTML に HTML を代入する場合、その HTML の文法に誤りがないよう気をつける必要がある。
1. 外部から入力した文字列を表示する場合、 HTML タグ等に誤解されないよう、 HTML マークアップ用の文字 (「<」、「>」、「&」、「"」) を実体参照 (「&lt;」等) に変換する必要がある。
1. ある要素の innerHTML 属性に代入した HTML 要素の配下にあとから HTML を記述したい場合、その要素を document.getElementById() で取得できるようにあらかじめ id 属性付きで記述しておく必要がある。

これに対して、 HTML 文書をオブジェクトのツリー構造に見立てて扱い、操作する方法があります。そのようなモデルのことを、 **DOM (Document Object Model)** と呼びます。

DOM を用いた画面操作の基本的な手順は、概ね以下の通りです。

1. document.getElementById() メソッド等を用いて、操作したい HTML 要素のオブジェクトを取得する。
1. 1 で取得した要素の配下に HTML 要素を追加したい場合は、 document.createElement() メソッドを用いて HTML 要素オブジェクトを生成し、 1 で取得した要素の appendChild() メソッド等を用いて追加する。
1. 1 で取得した要素の配下にテキスト文を追加したい場合は、 document.createTextNode() メソッドを用いてテキスト要素オブジェクトを生成し、 1 で取得した要素の appendChild() メソッド等を用いて追加する。
1. 1 で取得した要素の配下にある要素は childNodes 属性から参照する。 childNodes は配列のようになっており、 elem.childNodes[n] のようにアクセスできる。

他にも、 removeChild() メソッドで子要素を削除したり、 className 属性にクラス名を代入したり、 style 属性をいじってスタイルシートを変更したりと、さまざまな機能が存在します。

また、 HTML 要素の種類に応じて用意されている特別な属性やメソッドも存在します。すでに例示している <textarea> 要素の value 属性などは、フォーム部品の類にのみ用意されている特別な属性です。

試しに、 DOM を用いて、既存のパラグラフ (<p> タグ) に以下の HTML に相当する内容を追加するプログラムを書いてみましょう。

```text
これは、<strong>DOM</strong> を用いたサンプルです。
```

まず、既存のパラグラフに ID 「main_paragraph」が設定されているものとして、パラグラフ要素を取得します。

```js
    var p = document.getElementById("main_paragraph");
```

次に、このパラグラフ要素に、テキスト要素「これは、」を追加します。テキスト要素ですから、 document.createTextNode() メソッドを用いて生成し、それをパラグラフ要素の appendChild() メソッドに渡して追加します。

```js
    var text = document.createTextNode("これは、");
    p.appendChild(text);
```

生成したテキスト要素をわざわざ変数に持たせず、以下のように直接 appendChild() メソッドに渡してしまっても良いでしょう。

```js
    p.appendChild(document.createTextNode("これは、"));
```

次に、新たに生成した <string> 要素にテキスト要素「DOM」を生成して追加し、それをパラグラフ要素に追加します。言葉で書くとややこしいですが、プログラムにして書くとかえって分かりやすいかも知れません。

```js
    var strong = document.createElement("strong");      // <strong> 要素を生成
    strong.appendChild(document.createTextNode("DOM")); // テキスト要素「DOM」を生成して <strong> 要素に追加
    p.appendChild(strong);                              // <strong> 要素をパラグラフ要素に追加
```

最後に、残りのテキスト要素「 を用いたサンプルです。」を生成し、パラグラフ要素に追加すれば完了です。

```js
    p.appendChild(document.createTextNode(" を用いたサンプルです。"));
```

完成したプログラムを見てみましょう。ボタンをクリックするとこのプログラムが動作するように作られています。

```text
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN">
<html>

<head>
    <meta http-equiv="content-type" content="text/html; charset=UTF-8">
    <meta http-equiv="content-script-type" content="text/javascript">
    <title>small DOM sample</title>
    <script type="text/javascript" language="JavaScript"><!--
function domTest() {
    var p = document.getElementById("main_paragraph");
    p.appendChild(document.createTextNode("これは、"));
    var strong = document.createElement("strong");      // <strong> 要素を生成
    strong.appendChild(document.createTextNode("DOM")); // テキスト要素「DOM」を生成して <strong> 要素に追加
    p.appendChild(strong);                              // <strong> 要素をパラグラフ要素に追加
    p.appendChild(document.createTextNode(" を用いたサンプルです。"));
}
    //--></script>
</head>

<body>

<form>
  <input type="button" value="click!" onclick="domTest();">
</form>

<p id="main_paragraph"></p>

</body>

</html>
```

これだけでは面白くないので、もっと実践的な例を示しましょう。表示方法を設定するスタイルシートを用いれば、 HTML 要素を固定の表示位置に表示することができます。以下の HTML ファイルを見てみましょう。

```text
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN">
<html>

<head>
	<meta http-equiv="content-type" content="text/html; charset=UTF-8">
	<meta http-equiv="content-style-type" content="text/css">
	<meta http-equiv="content-script-type" content="text/javascript">
	<title>absolute position sample</title>
</head>

<body>

<div style="
  background: #09f;
  border: 1px solid #000;
  position: absolute;
  left: 100px;
  top: 50px;
  width: 100px;
  height: 100px;">
</div>

</body>

</html>
```

この HTML をブラウザに読み込ませると、青い矩形が画面上に表示されます。この矩形は <div> 要素ですが、 style 属性によっていくつかの表示方法が設定されています。

```text
<div style="
  background: #09f;
  border: 1px solid #000;
  position: absolute;
  left: 100px;
  top: 50px;
  width: 100px;
  height: 100px;">
</div>
```

わかりやすさの為、 style 属性の値はスタイルのパラメータ毎に改行してありますが、 background が背景 (矩形の中の色や模様; サンプルでは色のみ指定)、border が枠線の表示方法を指定するパラメータです。ポイントはその次の行からで、

```text
  position: absolute;
```

と書くことによって、その次の 2行、すなわち left や top パラメータを用いてその要素の表示位置を座標で指定してあげることができるようになります。 width と height はそれぞれ、矩形の幅と高さです。

DOM を用いると、このスタイルシートをパラメータ毎に変更することができるようになります。ということは、この矩形の表示位置を動かしたり、色や大きさを変えたりすることができる、ということです。試しに、ボタンを押すと矩形が右に動くだけの、簡単なプログラムを作ってみましょう。

まず、動かしたい矩形 <div> 要素のオブジェクトを取得します。あらかじめ HTML に <div> 要素を書いておいて document.getElementById() してもよいですが、せっかくなので document.createElement() を用いて 1から作るようにしてみましょう。

```js
// 最初の表示位置、大きさ
var pos_x = 0;
var pos_y = 50;
var rect_width = 100;
var rect_height = 100;

// 生成した <div> 要素のオブジェクトを入れておく変数
var div;

// HTML の読み込みが終わったら実行されるイベントハンドラ
window.onload = function() {
  // <div> 要素を作成する
  div = document.createElement("div");
```

<div> の生成は window.onload() イベントハンドラ内で行います。 window.onload() は HTML 文書の読み込みが完了したときに呼び出されます。そうすると、 HTML 文書に含まれる全ての HTML 要素に DOM としてアクセスすることができます。例えば <body> 要素にも document.body プロパティとしてアクセスできるので、ここで生成した <div> 要素を、あとで

```js
  document.body.appendChild(div);
```

と書いてやれば、<body> 要素内に、新たに作った <div> 要素を追加することができるのです。

window.onload() イベントハンドラより手前で、変数をいくつか定義しています。これらは、後で矩形の表示位置を変更する際に、役に立つものです。

次に、この <div> 要素にスタイルを設定します。 DOM を用いる場合、 <div> 要素のスタイルは、 div.style プロパティにオブジェクトとして用意されていますので、以下のようにしてスタイルを設定してやることができます。

```js
  // <div> 要素のスタイルを設定する
  div.style.background = "#09f";
  div.style.border = "1px solid #000";
  div.style.position = "absolute";
  div.style.left = pos_x + "px";
  div.style.top = pos_y + "px";
  div.style.width = rect_width + "px";
  div.style.height = rect_height + "px";
```

スタイルシートのパラメータの名前と、 style オブジェクトが持つプロパティの名前はだいたい同じだと思ってよいでしょう。但し、 background ではなく background-color を指定したい場合、 style オブジェクトのプロパティの名前は backgroundColor といったように、ハイフン "-" で繋がれた次のアルファベットを大文字にした名前となります。

ちなみにこの部分は、 JavaScript の **with** という構文を用いることで、以下のようにもう少しすっきり書くことができます。もっとも、これは好みの分かれる書き方ですが…。

```js
  // <div> 要素のスタイルを設定する
  with (div.style) {
    background = "#09f";
    border = "1px solid #000";
    position = "absolute";
    left = pos_x + "px";
    top = pos_y + "px"
    width = rect_width + "px";
    height = rect_height + "px";
  }
```

<div> 要素の準備はできたので、 <body> 要素にこの <div> 要素を突っ込んでやります。 window.onload() でやる作業はここまでです。

```js
  document.body.appendChild(div);
}
```

そして、ボタンが押されたときに呼び出される関数 moveRectangle() を実装します。この関数は、矩形の水平位置を、矩形の幅の半分ずつ右に動かす、というものです。

```js
// 矩形を右に動かす
function moveRectangle() {
  pos_x += rect_width / 2;
  div.style.left = pos_x + "px";
}
```

pos_x や rect_width といった変数を用意しておいたのは、 div.style.left プロパティ自体は直接足し算ができない文字列だからです。また、変数 div も、この <div> 要素にいちいち ID 名を設定し、あとで document.getElementById() で取得するのが面倒だったので用意しておいたのでした。

サンプルプログラムの全体像を以下に示します。

```text
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN">
<html>

<head>
    <meta http-equiv="content-type" content="text/html; charset=UTF-8">
    <meta http-equiv="content-style-type" content="text/css">
    <meta http-equiv="content-script-type" content="text/javascript">
    <title>moving rectangle sample 1</title>
    <script type="text/javascript" language="JavaScript"><!--
// 最初の表示位置、大きさ
var pos_x = 0;
var pos_y = 50;
var rect_width = 100;
var rect_height = 100;

// 生成した <div> 要素のオブジェクトを入れておく変数
var div;

// HTML の読み込みが終わったら実行されるイベントハンドラ
window.onload = function() {
  // <div> 要素を作成する
  div = document.createElement("div");
  // <div> 要素のスタイルを設定する
  with (div.style) {
    background = "#09f";
    border = "1px solid #000";
    position = "absolute";
    left = pos_x + "px";
    top = pos_y + "px"
    width = rect_width + "px";
    height = rect_height + "px";
  }
  document.body.appendChild(div);
}

// 矩形を右に動かす
function moveRectangle() {
  pos_x += rect_width / 2;
  div.style.left = pos_x + "px";
}
    //--></script>
</head>

<body>

<form>
  <input type="button" value="click!" onclick="moveRectangle();">
</form>

</body>

</html>
```

いかがですか? ここまでくると、なんだかいろいろなことに応用できそうな気がしてくるでしょう?

最後に、これのさらなる応用例を 2つ程お見せしましょう。まず 1つ目は、タイマーを用いて矩形が勝手に動き続けるサンプルです。

```text
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN">
<html>

<head>
    <meta http-equiv="content-type" content="text/html; charset=UTF-8">
    <meta http-equiv="content-style-type" content="text/css">
    <meta http-equiv="content-script-type" content="text/javascript">
    <title>moving rectangle sample 2</title>
    <script type="text/javascript" language="JavaScript"><!--
// 移動量 (制限値)
var MAX_MOVE_TIME = 8;

// タイマーのウェイト (ミリ秒)
var TIMER_WAIT = 125;

// 移動方向を表す定数オブジェクト
var Direction = {
  "RIGHT": 0,
  "BOTTOM": 1,
  "LEFT": 2,
  "TOP": 3
};

// 表示位置、大きさ
var pos_x = 0;
var pos_y = 0;
var rect_width = 100;
var rect_height = 100;

// 移動方向、カウント
var direction = Direction.RIGHT;
var count = 0;

// 生成した <div> 要素のオブジェクトを入れておく変数
var div;

// タイマーオブジェクトを入れておく変数
var timer;

window.onload = function() {
  div = document.createElement("div");
  with (div.style) {
    background = "#09f";
    border = "1px solid #000";
    position = "absolute";
    left = pos_x + "px";
    top = pos_y + "px"
    width = rect_width + "px";
    height = rect_height + "px";
  }
  document.body.appendChild(div);

  // タイマーを設定する
  timer = setInterval(moveRectangle, TIMER_WAIT);
}

// 矩形を動かす
function moveRectangle() {
  // 矩形の位置を変更
  switch (direction) {
    case Direction.RIGHT:   pos_x += rect_width / 2; break;
    case Direction.BOTTOM:  pos_y += rect_height / 2; break;
    case Direction.LEFT:    pos_x -= rect_width / 2; break;
    case Direction.TOP:     pos_y -= rect_height / 2; break;
  }
  div.style.left = pos_x + "px";
  div.style.top = pos_y + "px";

  // 方向転換
  if (++count >= MAX_MOVE_TIME) {
    direction = (direction + 1) % 4;
    count = 0;
  }
}
    //--></script>
</head>

<body>
</body>

</html>
```

このサンプルでは、矩形は右だけでなく、下、左、上にも移動し、際限なく回転をし続けます。 window.setInterval() メソッドを用いることで、このように、一定の時間間隔で処理を繰り返すプログラムを作ることができます。

もう 1つは、矩形をマウス操作のドラッグ＆ドロップで動かすことができる、というものです。

```text
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN">
<html>

<head>
    <meta http-equiv="content-type" content="text/html; charset=UTF-8">
    <meta http-equiv="content-style-type" content="text/css">
    <meta http-equiv="content-script-type" content="text/javascript">
    <title>moving rectangle sample 3</title>
    <script type="text/javascript" language="JavaScript"><!--
// 表示位置、大きさ
var pos_x = 0;
var pos_y = 0;
var rect_width = 100;
var rect_height = 100;

// マウスを動かし始めた位置
var mouse_pos_start_x;
var mouse_pos_start_y;
var pos_start_x;
var pos_start_y;

// 生成した <div> 要素のオブジェクトを入れておく変数
var div;

window.onload = function() {
  div = document.createElement("div");
  with (div.style) {
    background = "#09f";
    border = "1px solid #000";
    position = "absolute";
    left = pos_x + "px";
    top = pos_y + "px"
    width = rect_width + "px";
    height = rect_height + "px";
  }
  div.onmousedown = startMoveRectangle;
  document.body.appendChild(div);
}

// マウスの位置を取得する
function getMousePos(ev) {
  var pos = {};
  if (window.event) {   // IE の場合
    ev = window.event;
    var scroll_x = document.documentElement.scrollLeft || document.body.scrollLeft;
    var scroll_y = document.documentElement.scrollTop || document.body.scrollTop;
    pos.x = scroll_x + ev.clientX;
    pos.y = scroll_y + ev.clientY;
  }
  else {    // それ以外の場合
    pos.x = ev.pageX;
    pos.y = ev.pageY;
  }
  return pos;
}

// 矩形を動かし始める
function startMoveRectangle(ev) {
  // 動かしはじめのマウスの位置を記憶しておく
  var pos = getMousePos(ev);
  mouse_pos_start_x = pos.x;
  mouse_pos_start_y = pos.y;
  pos_start_x = pos_x;
  pos_start_y = pos_y;
  // ドキュメント全体に、マウスを動かした場合、マウスボタンを放した場合の処理を設定する
  document.onmousemove = moveRectangle;
  document.onmouseup = endMoveRectangle;

  return false; // Firefox、 Opera でテキストの選択を禁止する
}

// 矩形を動かす
function moveRectangle(ev) {
  // 矩形の位置を変更
  var pos = getMousePos(ev);
  pos_x = pos_start_x + pos.x - mouse_pos_start_x;
  pos_y = pos_start_y + pos.y - mouse_pos_start_y;
  div.style.left = pos_x + "px";
  div.style.top = pos_y + "px";

  return false; // IE でテキストの選択を禁止する
}

// 矩形を動かし終える
function endMoveRectangle(ev) {
  moveRectangle(ev);
  document.onmousemove = document.onmouseup = function() { };
}
    //--></script>
</head>

<body>
</body>

</html>
```

ブラウザ依存の部分もあるので若干わかりにくくなっていますが、イベントハンドラに渡されるイベントオブジェクトからマウスの位置を取得し、それを元に矩形の位置を移動する、という実装になっています。これをさらに応用すると、 Web ページ内でウィンドウを表示したり、ドラッグ＆ドロップを表現したりすることができるようになり、 Web アプリケーションの幅が広がります。

これら 2つのサンプルについては、あえて詳細な解説はつけません。じっくりとプログラム・ソースを吟味し、分からない部分はネットや各種書籍を参照して調べつつ、プログラムを改造してみたりしながら、覚えていくとよいでしょう。

## さいごに

以上で、 JavaScript チュートリアルは終わりです。 Web クライアントサイドプログラム言語としての JavaScript の雰囲気を掴むことはできたでしょうか?

JavaScript をうまく活用することで、 Web アプリケーション開発の幅がぐっと広がりますので、これを読んで興味がわいたならば、是非、勉強してマスターしましょう。

さて、本稿では、とりあえず JavaScript を用いて Web ページ上で実際に動くものを作り、 JavaScript を用いたクライアントサイドプログラミングの雰囲気を掴んでもらうことを目的として執筆させて頂きました。しかしながら、ここで学んだことは、 JavaScript を用いた Web 開発の、ほんの初歩の内容に過ぎません。 JavaScript 言語自体がサポートするさまざまなプログラミング手法やその文法、 Core JavaScript に用意されているさまざまなオブジェクト、 Ajax と呼ばれる手法を用いた Web サーバーとの非同期通信や JSON と呼ばれるデータ形式など、 JavaScript 周辺にはさまざまなトピックが存在します。それらの詳細については、また別のテキストで書いていきたいと思います。

それではまた。
