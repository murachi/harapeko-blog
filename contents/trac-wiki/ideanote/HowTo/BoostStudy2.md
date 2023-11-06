[author: murachi]
# Boost.勉強会 \#2 ノート

## Boost 世界一周の旅

* Property Tree

木構造を持つデータプロパティ管理。 XML (RapidXml), JSON, INI ファイルパーサ提供

boost::property_tree::ptree型

* Uuid

ユニークID生成。乱数生成器のデフォルトは mt19937

* Range2.0

Rangeアルゴリズム (イテレータの組ではなく範囲を渡す)、 Rangeアダプタ (遅延評価) が追加された。

要チェック→Oven

* Filesystem v3

path 日本語対応等

string と wstrint の両方を使用するためにオーバーロードが必要なくなった?

* Polygon

平面多角形 (2D) のアルゴリズム。

* Meta State Machine (MSM)

新たな状態マシンライブラリ。状態遷移表を直接記述。

### まとめ

* 1.44.0 はかなり充実。特に Range 2.0 はオススメ


## Boost.Graph 入門

### グラフ

* 頂点 vertex と辺 edge からなる
* 配列・線形リンクリスト (直線、木)

### 有向グラフ・無向グラフ

* 辺に向きがあるか否か

### 操作

頂点と線を作ったり、できたグラフの中を辿ったり…

### JR 最長片道切符

とにかく遠回りして一番長い距離の切符 (のルート) を求める

* →挫折した。データ入力が大変 (そりゃそーだｗ)
  * でも北海道だけは全部入力したよ。
* プログラムは作ったよ。
  * 長万部駅を検索の起点にしたよ (青函トンネル超えた最初の駅)
  * 実際動かしてみたよ

### なぜ Boost.Graph?

* イテレータでのアクセスがある
  * 辺・頂点
  * 一部は pair<iter, iter> で Range !
* malloc/for → STL(コンテナ/アルゴリズム)

### 質疑応答

* 直線、木もグラフだよね?
  * →正確にはサブセット?
* テンプレート引数の書く順番間違えたら MS-VS だとエラー箇所の赤線表示でる?
  * でた。 VS2010パネェ


## Boost.GIL 画像処理入門

Generic Image Library

### 目標

(初歩的な) 動体検出ができるようになる!

### GIL とな?

* 画像データ保持用コンテナとイテレータ。

### 前提知識

* 色空間
  * RGB/RGBA ... o
  * HSV ... x
  * YCbCr ... x
  * CMY/CMYK ... o
  * L*a*b* ... x
  * Grayscale ... o
  * etc...
* 種類
  * ベクタ ... x
  * ラスタ ... o

* 画像の構造
  * ピクセルの集まり
  * データ構造
    * データの持ち方は多様。配列を色別に分けたり分けなかったり…
* 動画像の構造
  * コマ送り

### 基礎の基礎

* 画像の入出力
  * JPEG, PNG, TIFF をサポート
    ```cpp
    rgb8_image_t img;
    jpeg_read_image("hoge.jpg", img); // 入力
    jpeg_write_view("fuga.jpg", view(img)); // 出力
    ```
* 型宣言は ColorSpace, BitDepth, ClassType の組み合わせで表現
  * ColorSpace: rgb, bgr, cmyk, ...
  * BitDepth: 8, 8s, 16, 16s, 32f, ...
  * ClassType: image, view, loc, pixcel, ptr, ...
* 画像へのアクセス
  * 直接 Image は操作しない。 Image View を使用
    ```cpp
    jpeg_read_image("hoge.jpg", img); // 読み込み
    step1 = view(img); // View を取得
    step2 = subimage_view(step1, 200, 300, 150, 150); // 切り取り
    step3 = color_converted_view<rgb8_view_t, gray8_pixel_t>(step2); // グレースケール化
    step4 = rotated180_view(step3); // 回転
    step5 = subsampled_view(step4, 2, 1);
    jpeg_write_view("hoge_transform.jpg", step5);
    ```
* ピクセルへのアクセス
  * image_view::operator()
  * Iterator
    * 画像の上から下に向かって水平行ごとに走査する。

### 動体検出アルゴリズム作成

フレーム間の差分を引くところ

* グレースケール化した画像の同位置ピクセルを引き算するだけ

### 参考情報

* http://sites.google.com/site/twinkleofsilence/
* http://sssm.sakura.ne.jp/dev/gil.html
* http://sourceforge.net/projects/stllcv/

### 質疑応答

* ビューを 1 から作ることは可能?
  * 作れるけど簡単じゃないかも…
* ビューへの操作をイメージに固定することは可能?
  * イメージへコピーするメソッドがある
* 色空間の独自実装は可能?
  * 比較的簡単らしい
* 型名が Boost っぽくない
  * 実は typedef 。裏ではすんげー長いテンプレート引数が…
* ビューのフィルタ操作による変換コストは?
  * 色空間の変換を除けばコストは 0 のハズ
* 中身が const な image を作るには
  * rgb8c_image_t のように BitDepth のうしろに "c" が入った型名を使う
  * 関数が view を受け取るときは const 参照で受け取ることが推奨されている。この const は view の設定が const なのであって、画像自体が const になるわけではない。


## とある関数の接合部 (スィーム)

sexyhook2 の使い方と実装について

### イントロダクション

* テストを書いていると一時的に関数の挙動を書き換えたいことがある。
  * ex) システム時刻に依存して true しか返さないような関数が false を返す所を見たい。
* → sexyhook を使えば、一時的に API や関数を置き換えることができる!

### 使い方

* http://code.google.com/p/sexyhook/
* VS, gcc 対応
* デバッグビルドのみ対象
* さまざまなフック
  * 関数
    ```cpp
    {
    	SEXYHOOK_BEGIN(time_t, SEXY_HOOK_CDECL, &time, (time_t * a) )
    	{
    		return /* なんか別な値 */;
    	}
    	SEXYHOOK_END();
    
    	/* フックした time を使う処理 */
    }
    /* ブロックを抜ければフックは無効 */
    ```
  * クラスメソッド
    * &クラス名::メソッド名
    * 仮想関数はちょっと特殊: SEXYHOOK_END に this を渡す必要がある
  * Win32 API
    * SEXY_HOOK_STDCALL を使う
* フック中に this を使う
  ```cpp
  SEXYHOOK_THIS(MyClass)->hoge();
  ```
  *

### 実装方法

* マクロ展開してるよ!
  * なんかクラス内クラスができるよ。
* アセンブリいじってるよ!
  * スタック積む代わりに jmp ねじ込んでるよ\!
* 差し替える関数ってどうやって呼んでるの?
  * 実体を見つけるのが結構大変だったよ\! とくに仮想関数…。
* API フック
  * DLL を読み込んだプログラム領域を直接書き換えちゃうよ!
  * ↑大丈夫なの??
    * 見た感じ大丈夫っぽい
    * NT カーネルに詳しい方いらっさいますか? ｗ

### 未来のセクシー

* private メソッドへの対応
  * Win32 は dbgutil がつかえる!
    * でも何故か VS2010 で動かない…
    * C++ でリフレクションができるかも…

### 質疑応答

* 多重継承は大丈夫?
  * ダイヤモンド継承は無理。
  * 単に多重しているだけなら…多分…大丈夫…じゃないかな。
* デバッグ＆コンティニューは
  * 想定外
* 64 bits にも対応して～
* マルチスレッド
  * クラス内クラスだから多分大丈夫…
* #define private public じゃ駄目なの?
  * ちょｗｗｗｗ


## Boost.Flyweight

* Flyweight パターンを実装
  * 等価なインスタンスを別々の箇所で使用する際、1つのインスタンスを再利用することによって省リソース化。

### 導入

* **const** の難点
  * 「変更しない」と「変更されない」は割と別物だが、 C++ の const はそれらを一緒くたに扱っている。
* そこで immutable
  * 値を共有できる
    * その為には GC が欲しい…
  * std::shared_ptr<T const> で実装する
* Flyweight デザインパターンは「値の共有」という考え方を突き詰めたもの。
* 基本的には shared_ptr<T const> と変わらん。
  * 等値のオブジェクトを1つの値にまとめ上げる点だけ異なる。

### 利点

* 気軽に使える
  * 非侵入的
    * const T が要件を満たせばどんな型でも Flyweight にできるよ
  * T const& への暗黙変換
* 細かく条件を設定できる
  * オブジェクトを保持するデータ構造、 GC の方法、 key-value flyweight による遅延構築、etc...
* パフォーマンス
  * 公式ページ参照
  * std::string のようなクラスの場合
    * 構築は遅い
    * コピーや等値比較は非常に高速
    * ハッシュ関数として保持されたオブジェクトのアドレスを利用可能。
* 特に unordered な連想コンテナに格納する場合に非常に強力\!

### ぼくのかんがえたさいきょうのもじれつクラス

```cpp
typedef boost::flyweight<std::string> fstring;
```

### 問題点

* ハッシュ関数はどうしよう?
  * アドレスを使うのは楽だが、プログラムを実行する度に値が変わる
  * 空間効率を落として良いなら、 flyweight に格納する文字列にハッシュ値を紛れ込ませられる
* 実装がかなり面倒くさい
  * std::string ってメンバ関数が多い
  * どこまで?
* Boost.Flyweight を使わずに

### より実際的な例

* Key-Value Flyweight を使った例
  ```cpp
  boost::flyweight<boost::flyweight::flyweights<Key, Value> >
  ```
  * 構築時は key, 使用時は value として扱える
  * Value の生成は遅延される
* Lua ネタ
  * Lua のソースファイルを扱うクラス
  * コンパイル済みのチャンクを保存し対象ファイルが更新されない限り次からはそちらを使うことで高速化

### まとめ

* 便利だよ!


## MessagePack Project

RPC とシリアライズ

### MessagePack とは?

"It's like JSON, but very fast and small."

* 論理的には JSON だが
* それをバイナリに置き換えてデータ量をひたすら小さくしたもの
* C++、 Ruby を始め、さまざまな言語で利用可能になっている。

### プロジェクト成り立ち

何度も似たようなプロトコルを設計… RPC ライブラリが欲しい!

1. 汎用的なバイナリプロトコルを実装
  * MessagePack C++/Ruby版
1. kumofs の RPC に採用
1. kumofs の RPC 機能全体を切り出して汎用化
1. オープンソース・各種言語版!
1. 国際展開中\!!

### 質疑応答

* エンディアン
  * BE 統一
* 文字コード
  * UTF-8 統一
* float 配列とかが欲しい…
  * 悩ましい
  * バイト列としてパックすれば一応サイズ縮小できるが…
  * 検討してみます
* C++ 版はイテレータ対応して欲しい
  * 次バージョンで実装したい
* 分散システムは kumofs とは別のもの?
* エラー処理
  * タイムアウトは実装済み: 時間を指定する
  * それ以外はひとくくりに「それ以外のエラー」
    * 詳細情報が取れるようにしたい ...検討中
* エラーは例外ではなくオプションでも取れるようにして欲しい。
* MessagePack を圧縮するのに適したアルゴリズムは? あるいは出現頻度の多い値への圧縮対応など。
  * 圧縮・展開が処理のオーバーヘッドになっては本末転倒
  * 多言語対応のため実装をシンプルに保ちたい - その範囲内で対応できることがあれば。


## マスタリングバベル

### バベルってなに?

文字エンコーディング変換モジュールです。

* shift_jis, JIS, EUC, UNICODE(UTF-8, UTF-16, UTF-32)
* 国際化対応のためのものではありません。日本語のみ。
* 高い文字エンコーディング判別精度
* 高い移植性!
  * AIX でも動くよ\!
* 利用実績 ... 結構ありそう

### 使い方

1. サイトから「バベル全ファイルZIPパック」をダウンロード
1. バベルを利用するプログラムのソースを置いているディレクトリにコピー\!
1. #include "babel.h"
1. babel::init_babel()
1. babel::auto_translate<>(source); or babel::sjis_to_euc(source); etc...
1. あとは babel.cpp ともどもまとめてコンパイル・リンク


## Boost.Interfaces

...is Not a Boost library (えっ)

### モチベーション

* テンプレートは持ち運べない。

### 使い方

```cpp
BOOST_IDL_BEGIN(Point)
	BOOST_IDL_CONST_FN0(getX, int)
	BOOST_IDL_CONST_FN0(getY, int)
	BOOST_IDL_FN1(setX, void, int)
	BOOST_IDL_FN1(setY, void, int)
	// ...
BOOST_IDL_END()
```

代入はコピーではない (参照が渡される)。 boost::interfaces::shared_ptr や boost::interfaces::shared_obj で持ち運ぶ。
