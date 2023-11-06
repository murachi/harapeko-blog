[author: murachi]
# どんな言語?

JavaScript 言語の大まかな特徴を、ざっと列挙してみましょう。

## 比較的小さな実装のスクリプト言語

Web ブラウザの機能の一つとして実装される JavaScript は、その為に非常に軽量に実装されています。つまり、言語自体が元々持つ機能が少ない、ということです。例えば、ローカルファイルへの入出力を行う命令などは用意されていません。

また、スクリプト言語なので、処理系はプログラムをステップ毎に逐次読み込みながら実行します。最近は JIT コンパイラ (内部でコンパイルを行ってから実行するタイプ) の処理系が多いので、基本的な構文チェックだけを先に通してから実行される、という動作も多くなってきていますが、基本的にはエラーが発生する前の部分までは実行される、といった動作になります。

## 文法は C や Java にやや似ている

JavaScript という名前の通り、言語の文法は Java に似ていなくもありません。が、値の型が曖昧な点や、インタプリタとして動作する点などを理由に、初期の頃は C に似た書き方の BASIC みたいな言語、といった見られ方をしていました。

近年、 JavaScript の持つ言語としての特徴が再評価され、むしろ C に似た書き方の LISP みたいな言語、といった見られ方に移りつつあるようです。

文法が C や Java に似ていると言われる所以は何か。まず、処理の単位となるブロックをブレース "{" ～ "}" で括る書き方。

```js
function hoge() {
    // 処理...
}

if (a == b) {
    // 処理...
}
```

それから、処理の単位をセミコロン ";" で区切る書き方 (但し、行末のセミコロンは必須ではない)。

```js
var a = 10;  var b = a + 20;

// 行末のセミコロンは省略可
var a = 10
var b = a + 20
```

コメントの書き方も Java と同じです。 "/*" ～ "*/" で括るか、 "//" の後ろに書きます。

```js
/*
 * 複数行
 * コメント
 */

alert("コメントサンプル");  // 一行コメント
```

その他、 if, for, while, switch といった処理の流れを記述するおなじみの構文や、 C言語風の演算子 (==, ++, -- など)、さらには関数呼び出し時には関数名の後ろに引数をかっこで括る書き方なども、まさに C や Java そっくりの書き方です。

## 数値と文字列の型は曖昧

多くのスクリプト言語と同様、 JavaScript もまた、数値と文字列を自由に行き来する言語です。例えば、数値と文字列の足し算は、文字列の連結として解釈されます。

```js
var a = 10;
var b = 20;

alert("a + b = " + (a + b));  // "a + b = 30" と表示
```

また、数字だけの文字列を数値演算に含めることもできます。その場合、足し算においては文字列の連結として誤解されないよう注意が必要です。

```js
var a = document.getElementById("input_a").value - 0;  // 入力欄の文字列を取得。
var b = document.getElementById("input_b").value - 0;  // 0 で減算することで、変数には確実に数値として代入される。

document.getElementById("result").value = a + b;       // 足し算した結果を別のフォーム部品に表示。
```

<h2 id="introduction-hashmap">複雑なデータ構造を自由に書ける</h2>

C や Java がデータ構造の構成を構造体やクラスとしてあらかじめ定義しなければならないのに対し、 JavaScript のオブジェクトはいつでも好きなだけメンバを追加できます。

```js
// オブジェクトを作成
var murachi = {
    real_name = "村山 俊之",
    real_name_kana = "ムラヤマ トシユキ",
    birth = new Date(1978, 1, 7),
    sex = "Male"
};

// 本名を表示
alert("名前: " + murachi.real_name + " (読み: " + murachi.real_name_kana + ")");

// オブジェクトに後からメンバを追加できる
murachi.disp_name = "T.MURACHI";
murachi.tall = 166;
murachi.weight = 70;

alert(murachi.disp_name + "の身長は" + murachi.tall "cm、体重は" + murachi.weight + "kg");
```

もちろん、メンバにオブジェクトや配列を持たせることもできますし、配列にオブジェクトを持たせることもできます。

```js
var harapeko = {
    name = "株式会社はらぺこ",
    name_e = "Harapeko Inc.",
    birth = new Date(2008, 4, 1),
    // オブジェクトの入れ子
    address = {
        post_no = 2750015,
        county = "千葉県",
        city = "習志野市",
        town = "鷺沼台1-8-27",
        building = "マウント・ヴィレッヂⅠ 101"
    },
    // メンバに配列を追加
    address_history = [
        // オブジェクトの配列
        {
            post_no = 2760017,
            county = "千葉県",
            city = "八千代市",
            town = "八千代台北1-10-11",
            building = "マメール八千代台 502"
        },
        {
            post_no = 2760017,
            county = "千葉県",
            city = "八千代市",
            town = "八千代台北1-10-11",
            building = "ファイン八千代台 502"
        },
        {
            post_no = 2750015,
            county = "千葉県",
            city = "習志野市",
            town = "鷺沼台1-8-27",
            building = "マウント・ヴィレッヂⅠ 101"
        }
    ]
};

// オブジェクトの内容を一通り表示
document.write("<p>会社名: " + harapeko.name + "</p>");
document.write("<p>創立: " + harapeko.birth.toLocaleString() + "</p>");
document.write("<p>現住所: 〒" + harapeko.address.post_no + " " +
    harapeko.address.county + harapeko.address.city + harapeko.address.town + " " +
    harapeko.building + "</p>");

document.write("<p>過去の住所 (古い順に):</p><ul>");

for (var i = 0; i < harapeko.address_history.length; i++) {
    var old_address = harapeko.address_history[i];
    document.write("<li>〒" + old_address.post_no + " " +
        old_address.county + old_address.city + old_address.town + " " +
        old_address.building + "</li>");
}

document.write("</ul>");
```

## 割と本格的な関数型言語

JavaScript では、関数の定義内容を変数に代入することができます。

```js
// 変数 hoge に関数を代入
var hoge = function(text) {
    alert("hoge: " + text);
}

// hoge に代入した関数を呼び出す
hoge("fuga");
```

かっこでうまく括ってやれば、わざわざ変数に代入せずとも、その場で呼び出してしまうことも可能です。

```js
// 無名の関数を即座に呼び出す例
(function(text) {
    alert("hoge: " + text);
})("fuga");
```

こうして作られた無名関数は、**クロージャ**として機能します。つまり、無名関数が生成されたときの変数の値を覚えています<small> (正確には、変数のスコープが失われたときの値を覚えています。この性質は若干ややこしいので、クロージャを用いる際には関数の生成の仕方に注意する必要があります。詳細は後述します。) </small>。

```js
// クロージャを生成する関数
function generateAlertClosure(text) {
    // 無名関数を返す
    return function() {
        alert(text);
    };
}

var hoge = generateAlertClosure("ほげっ!");
var fuga = generateAlertClosure("ふんがーヽ(ﾟ口ﾟﾐ)ノ");

alert("まだ何も起こっていないはず…");  // ここまではまだ alert が呼ばれていないことを確認

hoge(); // ここで "ほげっ!" と表示
fuga(); // "ふんがー(ry" と表示
```

[[FootNote]]

## プロトタイプ型オブジェクト指向

JavaScript でオブジェクトと言えば、複雑なデータ構造を表す**連想配列**を指します。「[複雑なデータ構造を自由に書ける](#introduction-hashmap)」の節で示したアレですね。

```js
// オブジェクト
var obj = {
    "foo": "hoge",
    "bar": "fuga",
    "baz": "oyoyo"
};
```

ところで、オブジェクト指向プログラミングといえば、オブジェクトの生成・破棄を行う処理を定義するコンストラクタ・デストラクタがあり、メンバメソッドがあり、カプセル化のためのアクセス制約があり、クラスの継承がある、といったいわゆる Java 的な世界観が連想されます。 JavaScript の場合はクラスではなく**プロトタイプ**というアプローチではありますが、似たようなことを行うための記法が用意されています。

まず、関数の定義はそのままコンストラクタとしても機能します。

```js
// クラス定義のようなもの
function MyObject() {
    // メンバ変数の定義
    this.foo = "hoge";
    this.bar = "fuga";
    this.baz = "oyoyo";
}

// 上記を用いてオブジェクトインスタンスを生成
var obj = new MyObject();

alert("foo: " + obj.foo);  // "foo: hoge" と表示される
```

デストラクタは残念ながら存在しません。オブジェクトが必要なくなるタイミングで特定の処理を行いたい場合は、自分でメソッドを用意して、自分で確実に呼び出すよう記述する必要があります。

メンバメソッドはメンバ変数に関数を代入でも良いのですが、通常はプロトタイプのメンバに代入します。

```js
// クラス定義のようなもの
function MyObject() {
    // メンバ変数の定義
    this.foo = "hoge";
    this.bar = "fuga";
    this.baz = "oyoyo";
}

// メンバメソッド定義
MyObject.prototype = {
    "show" = function(member) {
        if (this[member] == null)
            alert(member + "なんてメンバはねーよ!!");
        else
            alert(member + ": " + this[member]);
    },
    "showAll" = function() {
        var dump = "";
        for (var member in this) {
            if (dump != "")
                dump += "\n";
            if (typeof this[member] != "string")
                continue;
            dump += member + ": " + this[member];
        }
        alert(dump);
    }
}

// 上記を用いてオブジェクトインスタンスを生成
var obj = new MyObject();

obj.show("bar");    // "bar: fuga" と表示
obj.showAll();      // foo, bar, baz すべて表示
```

以上が、 Java や C++ などで言うところのクラス定義に相当する、といって良いでしょう。継承を行いたい場合は、新たに定義するコンストラクタの関数名に、プロトタイプの複製をコピーしてあげれば ok です。以下は、人を表す基本クラス Human からコンストラクタとメソッドを継承し、働く人を表す派生クラス Worker を実装するサンプルです。

```js
function Human(name, birth, sex) {
    if (name == null)  // プロトタイプの複製をコピーする際には、このコンストラクタは何もしない
        return;
    this.name = name;
    this.birth = birth instanceof Date ? birth : new Date(birth);
    this.sex = /^m/i.test(sex) ? "male" : "female";
}

Human.prototype = {
    "introduce": function() {
        var message = "私の名前は" + this.name + "。" +
            this.birth.getFullYear() + "年" + (this.birth.getMonth() + 1) + "月" +
            this.birth.getDay() + "日生まれの" + (this.sex == "male" ? "男性" : "女性") + "です。";
        var message += this.makeAdditionalIntroduction();
        alert(message);
    },
    "makeAdditionalIntroduction": function() {
        // 基本クラスでは何もせず、空文字列を返す
        return "";
    }
};

function Worker(name, birth, sex, company, roll, post) {
    Human.call(this, name, birth, sex);  // 親クラスのコンストラクタを呼び出す
    this.company = company;
    this.roll = roll;
    this.post = post;
}

Worker.prototype = new Human();  // クラスメソッドの継承 (プロトタイプの複製をコピー)

// Worker 独自のメソッドを実装
Worker.prototype.makeAdditionalIntroduction = function() {
    // 自己紹介に追加する内容を返す
    return "\n" + this.company + (typeof this.post == "string" ? "の" + this.post : "") +
        "にて、" + this.roll + "をやっています。";
};

var murachi = new Human("村山 俊之", new Date(1978, 1, 7), "male");
murachi.introduce();

murachi = new Worker("村山 俊之", new Date(1978, 1, 7), "male", "株式会社はらぺこ", "代表取締役社長");
murachi.introduce();
```

最後に、メンバの private 宣言に相当するアクセス制約についてですが、 JavaScript にその為の簡潔な記法が用意されているわけではないものの、それに近いものをクロージャを用いて実現することは可能である、という例を以下に示します。ここまでやるとかなり大きなグループ開発にも耐えうる運用となり得ますが、 JavaScript 界隈ではあまり見かけない習慣かも知れません。

```js
var IS_DEBUG = true;

function Human(name, birth, sex) {
    // プライベートフィールド
    var fields = {
        "name": name,
        "birth": birth instanceof Date ? birth : new Date(birth),
        "sex": /^m/i.test(sex) ? "male" : "female"
    };

    // メソッドがプライベートフィールドを参照するためのメソッド
    this.getFields = function() {
        if (IS_DEBUG && !this.permitPrivate())
            throw new Error("private method access denied.");
        return fields;
    };
}

Human.prototype = {
    // プライベートメソッドの呼び出し元をチェックしてアクセス許可を判定する
    "permitPrivate": function() {
        var is_permitted = false;
        for (var method in Human.prototype) {
            if (Human.prototype[method] == arguments.callee.caller.caller) {
                is_permitted = true;
                break;
            }
        }
        return is_permitted;
    },
    // private:
    "getAge": function() {
        if (IS_DEBUG && !this.permitPrivate())
            throw new Error("private method access denied.");
        var now = new Date();
        var today = now.getFullYear() + ("0" + (now.getMonth() + 1)).slice(-2) +
            ("0" + now.getDay()).slice(-2);
        var birth = this.getFields().birth;
        var birth_day = birth.getFullYear() + ("0" + (birth.getMonth() + 1)).slice(-2) +
            ("0" + now.getDay()).slice(-2);
        return (today - birth_day) / 10000 | 0;
    },
    // public:
    "introduce": function() {
        var fields = this.getFields();
        alert("私の名前は" + fields.name + "。" +
            fields.birth.getFullYear() + "年" + (fields.birth.getMonth() + 1) + "月" +
            fields.birth.getDay() + "日生まれの" + this.getAge() + "歳・" +
            (fields.sex == "male" ? "男性" : "女性") + "です。");
    }
};

var murachi = new Human("村山 俊之", new Date(1978, 1, 7), "male");

murachi.introduce();            // 紹介文を表示
alert(murachi.getAge() + "歳"); // エラー: プライベートメソッドにアクセスできない
```

## JavaScript 言語を学ぶということの意味

以上、 JavaScript 言語の風景を駆け足で紹介しました。ここまでに示したサンプルプログラムについて、まだ理解できる必要はありません。これから、これらについて一つ一つ仕組みを理解し、使えるようになってゆきましょう。

最後に、ここで学ぶ事柄についてもう一度確認しておきましょう。

JavaScript という言語は、必ずしも Web ブラウザにのみ搭載されている Web クライアントプログラムのためだけの言語というわけではありません。かつて JavaScript を産んだ Netscape 社から派生し、 Firefox などの優れた製品を開発している Mozilla も、アプリケーションの開発者が自身の開発するアプリケーション上で JavaScript を扱えるようにするための JavaScript エンジンである [SpiderMonkey](https:://developer.mozilla.org/ja/SpiderMonkey) を公開しています。このことは、即ち Web ブラウザ以外のありとあらゆるアプリケーションに、言語としての JavaScript が搭載される可能性があることを示しています。

ここでは、 JavaScript エンジンを搭載するアプリケーションの種類に依存しない、 JavaScript の純粋な言語仕様について学びます。その中には、 Web ブラウザに特化したオブジェクトや命令は含まれない点に注意して下さい。例えば、 Web ブラウザに搭載される JavaScript エンジンにおいて特別な意味を持つオブジェクト window については、ここでは特に触れることはない、ということです。

しかしながら、 JavaScript 言語を学ぶ上で、実際にそれを動かす方法として最もありふれているのはやはり Web ブラウザですので、サンプルプログラムは Web ブラウザ上で動作するものとして記述することはご容赦下さい。

最後の最後に、本稿はあくまで概論であり、初心者を意識した内容となっていることを付け加えておきます。 JavaScript 言語仕様をより正確に理解したい方は、 [Mozilla Developer Center (MDC) が公開している資料](https:://developer.mozilla.org/ja/JavaScript)をご参照頂いた方がよいかも知れません。
