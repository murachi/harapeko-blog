[author: murachi]
# どんな言語?

Action Script 2 言語の大まかな特徴を以下に列挙します。

## 基本は JavaScript と同じ

言語仕様は基本的には JavaScript に酷似しています。変数の使い方、演算子、関数の書き方、分岐やループなどの制御命令、無名関数や、それをクロージャとして使えるところまで、同じように書くことができます。

## 変数の型指定

JavaScript では変数自体に型を持たせることはできませんでしたが、 Action Script 2 では変数を宣言する際に、その変数に代入できる値の型を指定することができます。例えば、数値のみを代入できる変数を宣言するには、以下のように記述します。

```js
var hoge:Number;
```

このとき、変数 hoge に数値以外のものを代入するようなプログラムを書くと、コンパイルエラーになります。

```js
var hoge:Number;
hoge = 10;      // ok.
hoge = "fuga";  // コンパイルエラー
```

型指定は関数の引数や戻り値に対しても指定できます。例えば、 3つの数値を引数に取り、その平均を求める関数は、以下のように書くことができます。

```js
function calcAverage3(a:Number, b:Number, c:Number):Number {
    return (a + b + c) / 3;
}
```

このとき、この関数に渡す引数の型が数値以外であったり、この関数の戻り値を代入する変数が Number 型ではなかったりすると、コンパイルエラーになります。

## Java ライクなオブジェクト指向

JavaScript がプロトタイプ型のオブジェクト指向モデルであるのに対して、 Action Script 2 のオブジェクト指向モデルはやや Java に似たスタイルになっています。

例えば、会社の名簿に載せる人の情報を表すオブジェクト Human について、 JavaScript では以下のように記述することになりますが、

```js
// Human オブジェクト定義、兼コンストラクタ
function Human(number, name, age, sex, section, role) {
    this.number = number;
    this.name = name;
    this.age = age;
    // ...
}

// メソッド定義
Human.prototype = {
    "getNumber": function() { return this.number; },
    "getName": function() { return this.name; },
    // ...
};

// オブジェクト生成と利用
var murachi = new Human(1, "Toshiyuki Murayama", 32, "M", "Engineering Department", "President");
var name = murachi.getName();  // "Toshiyuki Murayama"
```

同様のオブジェクトについて、 Action Script では以下のようにクラスとして記述することができます。

```js
class Human {
    // メンバ変数定義
    var number:Number;
    var name:String;
    var age:Number;
    // ...

    // コンストラクタ
    function Human(number:Number, name:String, age:Number, sex:String, section:String, role:String) {
        this.number = number;
        this.name = name;
        // ...
    }

    // メソッド定義
    function getNumber():Number { return this.number; }
    function getName():String { return this.name; }
    // ...
}

// オブジェクト生成と利用
var murachi:Human = new Human(1, "Toshiyuki Murayama", 32, "M", "Engineering Department", "President");
var name:String = murachi.getName();  // "Toshiyuki Murayama"
```
