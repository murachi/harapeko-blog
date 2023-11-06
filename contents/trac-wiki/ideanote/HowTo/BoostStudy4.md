[author: murachi]
# Boost.勉強会 \#4 ノート

## Boost Fusion Library

### タプルとは

* Boost Tuple Library
* std::pair の N個版

### ヘテロなコンテナとは

あらゆる型を格納できるコンテナ。 std::vector<boost::any>
タプルもヘテロの一種

### Boost.Fusion

タプルをヘテロとみなして STL チックな種々のアルゴリズムを適用するライブラリ。

### Fusion シーケンス

* vector, list, deque が使える
* イテレータもいける
* for_each の実装例 ... 要素毎に型が異なるので再起関数として実装する

### アルゴリズム

* View を用いた遅延評価が特徴。

### 使いどころ

* 名前がついているがシーケンスとしても扱いたい場合 (RGB など)
  * 構造体として扱うと名前を付けられるがシーケンスとして扱えない
  * 配列だと名前が…
* DSEL の内部実装 (Boost.Spirit.Qj/Karma)
  * 異なる型のシーケンスを扱う機会が多い。Regexp やパーサコンビネータ。
    ```
    fusion::vector<int, char, double> result;
    parse("1 a 3.14", int_ >> char_ >> double_, result);
    ```
* Fusion Sequence をコンセプトとするライブラリへの一括アダプト (Boost.Geometry)
  * Fusion シーケンスとしてアダプトされたすべての型を、 Geometry の coordinate として扱うことが出来る。

### ライブラリの設計

* タプルは値 (メンバ変数) と値の集合 (クラス) に、名前のない複合データ型。
* Boost.Fusion では、ユーザー定義型を Fusion シーケンスにアダプトすることで、名前有りタプルとみなすことが出来るようになる。
* ユーザーコード: 名前有り世界 / ライブラリコード: 名無し世界 として棲み分けするとよさげ…。
  * タプルはクラスを作るのが面倒くさいときに即興で使われることが多い。... メンテナンス性が低下する
  * データを単なるシーケンスとして扱って良いのはライブラリの中だけ。
* ユーザーコードではユーザー定義型を fusion シーケンスへアダプトすることで有用なアルゴリズムが使えるようになる。

### まとめ

* タプルをリストとみなす
* タプルに名前をもたらす
* ライブラリの設計に取り入れればユーザーコードが柔軟に


## C++プログラマの為のセキュリティ入門

* 基本的且つ普遍的な事柄が中心ですよ～

### セキュリティって?

* ツッコミが恐い世界ｗ
* ここ数年のトレンドは常に追っていこう
* こうしておけば安心というのは幻想
* セキュリティはトータル
* コスパは重要
  * 存在時の被害よりコストが増大では意味がない
  * 攻撃に成功するのにかかるコストが十分に大きければよい
* リスクアセスメント
  * 情報資産の棚卸し
  * CIA の観点からリスクを列挙 (機密性、完全性、可用性)
  * コストパフォーマンス
* そもそも何を防ぐべき?
  * 意図しない/許さない
    * 権限の取得
    * 処理の実行
  * 情報の漏洩、盗聴、削除、改ざん
  * DoS
  * spam/嵐
  * H/W の損壊/盗難
* どう防ぐべき?
  * 権限管理
  * 入力チェック <- ?
  * 入出力の正確なエンコード/デコード/エスケープ/アンエスケープ
  * 暗号
  * 証明
  * ケンジントンロック
  * 法的圧力
  * etc...
* 公開されているロジックで (暗号の話)
* 安全性の定義
* 暗号の安全性
  * 情報理論的安全性
    * 正解が (理論的に) わからない →強秘匿性
      * 暗号文と同じ分量の鍵情報が必要になる。使い回しできない。
  * 計算量的安全性
    * 大量のコスト (時間、CPU) をかけなければ解けない
      * コンピュータの進化は早い…
* 事例を追いかけよう

### さまざまな問題

* 通信の盗聴
* バッファオーバーフロー
* 整数オーバーフロー
* エスケープ
* セッションハイジャック
* 辞書攻撃
* ファイルパス
* ファイルコンテンツ

### 歴史

* DES の最強っぷりと衰退、そして AES へ

### さまざまな技術

* ハッシュ関数
  * MD5, SHA1, SHA2
    * MD5, SHA1 は非推奨…
  * 一方向性関数
* 暗号
  * 秘密分割
    * XOR
  * 秘密分散
    * 多項式+有限体
  * 上記 2つは情報理論的暗号性、強秘匿性があるよ
* 乱数
  * 真性乱数
    * 偏りが現れる。
    * 一般的に遅い。
  * 疑似乱数
* PKI
  * 実在証明でしかない
  * おれおれ証明書だとなりすましを防げないよ
  * 証明書の更新時には秘密鍵もちゃんと更新しよう!
* TPI

### C++ では

* クライアントで動作するコードであれば
  * すべてクラック可能
  * システム管理者とユーザーとゲストとリモートアクセスに対してどのようにあるべきか
  * クラックにも難易度が
  * コードサイニング (署名)
* サーバーで動作するコードであれば
  * 入出力チェックを厳格に
* バッファーオーバーラン対策
  * 文字数上限チェック
    * マルチバイト周り、サロゲートペア周りでちょんぼしないこと
  * 整数オーバーフロー対策→値域上限チェック
  * 正しくエスケープ
* 不正な文字エンコードの検出
  * UTF-8 でのチェック逃れ
  * マルチバイト文字列での閉じ文字喰い
* ファイルパス
  * 相対パスのチェック
  * NGワードの除外
  * 偽バックスラッシュ
    * UNICODE 外のエンコードに変換時にバックスラッシュに化ける
  * UNICODE 制御文字
* Shift と XOR を使った文字列の難読化
  * nul 文字を nul 文字のままに出来る
* セキュアな乱数
  * Windows: CryptGenRandom()
  * Linux: /dev/random, /dev/urandom
    * /dev/random は長時間ブロックする場合がある。 /dev/urandom は必ずしもセキュアではない。
* ライブラリ
  * Crypto++
  * Crypto API
* Windows の Crypt API を使用する上での注意
* JISEC と JCMVP

### 参考情報

* IPA
  * セキュアプログラミング講座
  * JISEC
  * JCMVP
  * セミナー・イベント
* JPCERT CC
* セキュリティーホールmemo
* 本当は恐い文字コードの話
* それ Unicode で
* 暗号技術大全


## OpenALで今日から始めるオーディオプログラミング

### OpenAL って何

* どんなスピーカーで再生するかによらず意図したサウンドを鳴らすプログラムを書きたい
  * ステレオ環境か 5.1ch 環境かに応じて動作が切り替わるようにしたい

### 3D オーディオのモデル

* 音源ソースとリスナーの位置を定義する。

### 歴史

* Loki が DirectSound に対応する Linux で動作するオーディオライブラリを欲した
* Loki と Creative で共同開発
* Loki 亡き今、 Creative が単独でメンテナンス

### 関数の分類

* al* ... オーディオ周り
* alc* ...

### デバイス選択

* alcGetString

### コンテキスト作成

* alcCreateContext
* alcMakeContextCurrent

### ソースを作る

* alGenSources

### バッファ

* ソースに複数のバッファを登録
* alGenBuffers
* alBufferData
* alSourceQueBuffer

### 再生

* alSourcePlay
* alGetSourcei(source, AL_SOURCE_STATE, &stat) ... 再生状態を監視

### 後片付け

* alDelete*

### 複数の音を鳴らす

* alSourcePlayv(2, source)

### ソースを動かす

* alSource3f

### 鳴らしながら移動

* alSource3f に速度情報を渡すとドップラー効果も再現する

### ストリーミング再生

* alGetSourcei(, AL_SOURCE_STATE
* alGetSourcei(, AL_BUFFERS_PROCESSED
* alSourceUnqueueBuffers 取り出し
* alBufferData
* alSourceQueueBuffers

### 録音

* alcGetString(, ALC_CAPTURE_DEVICE_SPECIFIER
* alcCaptureOpenDevice
* alcCaptureStart
* alcCaptureStop
* alcGetIntegerv

### 拡張

* 調べる - allsExtensionPresent("AL_EXT_FLOAT32"
* 関数を取得 - alGetProcAddress


## C++の悪口を言う

### Discriminating hackers が選ぶ言語

* ICFP 関数プログラミング学会が毎年プログラミングコンテストを開催
* 前回の優勝者 shinh ＆言語 C++
* よりにもよって手続き型の C++ が関数プログラミングの学会主催のコンテストで (ry
* 今回は C++ & Haskell & Python だたーよ
  * C++ 担当してたのが tanakh さ
    * 俺だけは悪口を言う権利がある\! (tanakh さ)

### 型

* 同じ型を何度も書かされる。
  * typedef 使え → 意味のない名前を考えなきゃいけない、名前空間も無駄に汚れる
  * 0x 使え (auto) → 0x 使わせて (´;ω;｀)
* 読ませる気のないコンパイルエラー
  * 行番号しか参考にならん
  * gcc4.5 デフォルト型引数の省略などが入ったが…
    * そも型が大きいと意味がない
    * そも構造上の問題?
* 型推論が非力
  * そも型推論とは Type Reconstruction である
  * C++0x では、すべての型を推論できるわけではない
    * ○ローカル変数 (auto)
    * ○ラムダ式の返値
    * ×関数の引数
    * ×クラスメンバ
  * Hindly-Milner型の型推論が出来ない

### テンプレート

* template の型パラメータは forall ではない
  ```
  template <class T>
  T id(T v) { return v; }
  ```
  * 引数に対して任意の操作が可能
    ```
    template<class T>
    void id(T v){ v.foo{}; }
    ```
  * どういう型がつくのが妥当なのだろうか?
  * 実際には C++ ではテンプレート関数自体には型はつかない
* C++ のテンプレートは何なの?
  * 見た目 structual な sub typing を実現している
  * sub type relation の解決をインスタンス時に各々行う
  * コンパイル時に型レベルでコードを実行するようなもの
* その代償
  * テンプレート関数を宣言時ではなく使用時にインスタンス化、それに対して型つけ
  * つまり
    * テンプレート関数を分割コンパイルできない
    * テンプレート関数自体には型がつかない
    * コンパイルエラーが意味不明に…
* 分割コンパイルできない
* コンパイル速度 ... 最悪の倍、コンパイル時間がファイル数の二乗のオーダーに
* ファイル感の依存性
  * pinpl のようなバッドノウハウ
    * 実行速度低下
    * コードがまどろっこしい
* ヘッダに書くことによる問題
  * namespace地獄
    * using namespace させて…
* テンプレートに型がつかない
  * 型は非常によいドキュメントになるんだが…
    * 人がある程度注釈することは出来るが…
* 今は亡きコンセプトさん…
  * テンプレート引数への要件の明示によるドキュメント性の改善
  * インスタンス化->型チェックによらずに型チェックが出来るようになり、コンパイルエラーの改善
  * コンセプトさえあれば…

### 関数プログラミング

* オブジェクト指向+関数型
  * すべてのものはオブジェクトであるべき
  * すべてのものは関数であるべき
  * 関数がオブジェクトであると言うこと
    * 関数につく型が凄くいびつに
      * 関数型ならシンプルなのに…
* ラムダ式
  * [](int x){ ... }(); ... とにかくキモイｗ
  * ラムダ式の型がいけてない
* 関数の型
  * function クラスを用いると簡単
    * しかしパフォーマンス低下
      * inline 展開されない
    * 単なるテンプレートパラメータが好まれている
* たそう関数
* テンプレート関数の値
* 総称型がない
* Boost.Lambda でのラムダ式
  * 非常に分かりにくい型になってしまう…
* パターンマッチ
* カリー化
  * 純粋関数的に状態を持ち回るのがとてもやりやすくなる
  * オブジェクト vs 関数
  * 関数プログラミングでは次の状態として関数を返すことがよくある

### まとめ

* まだまだ言いたいことはあるが…
* C++ にも良いところはあるよ
  * 適当に書いてもそれなりに速い
  * 0x でそれなりに改善

### スライド

* http://www.slideshare.net/tanakh/ccdis


## なぜ僕はProtoをぺろぺろするのか

### Boost.Proto 概要

* Expression Template の構築を補助するライブラリ
  * a + b * c -> plus<A, multiply<B, C> >
* Boost 1.37
* Headers only

### Boost.Proto は縁の下の力持ち

* 単体で使って嬉しいライブラリではない
* Boost 内で利用しているライブラリ:
  * Boost.Spirit
  * Boost.Phoenix v3
  * Boost.MSM.eUML
  * Boost.Xpressive

### Expressoin Template (ET) の型

* 割と重要
  * エラーメッセ維持で頻出
    * 恐ろしく長い
  * Qi で Proto 関係の部分でエラーを出すと遭遇

### とりあえず使ってみる

```
default_context const ctx;
std::const << eval(lit(1) + 2 + 3, ctx)
           << std::endl;
```

実行結果
```
6
```

* + を - とするコンテキスト
  ```
  struct my_context {
    template <
      class Expr,
      class Enable = void
    > struct eval;
  };
  ```

### SFINAE

* Substitution Failure Is not An Error
  * 置き換え失敗はエラーにあらず
* boost::enable_if など
  ```
  template <class T>
  typename enable_if<is_const<T> >::type foo();
  ```
* SFINAE を用いて eval を分岐する。

* + を - とするコンテキスト
  ```
  template <class Expr>
  struct eval<
    Expr,
    typename boost::enable_if<
      matches<Expr, terminal<int> >
    >::type
  > {
    typedef int result_type;
  
    result_type
    operator ()(Expr& e, my_context const&)
    const {
      return value(e);
    }
  }
  ```

### Boost.Proto の拡張方法

* Context
  * 評価に関する情報の提供
* Domain
  * 式の生成手段の提供
* Extends
  * 式を表す型を自分で提供
* Grammer
  * 文法を定義。演算子オーバーロード制御。

### 数値計算に Boost.Proto

* ET と数値計算は相性が良い
  * 「アルゴリズム」と「計算実装」を分離できる
    * 組み合わせの爆発を抑制
  * 見た目が良い (+ が加算を表す)
* 単純な計算の組み合わせで表現できる。
  * Basic Linear Algebra Subprograms (BLAS)

### Boost.uBLAS では満足できない理由

* Boost.uBLAS は可読性を重視している
* よって遅い
* 拡張できない

### General Purpose computing on GPU (GPGPU) について

* アルゴリズムは CPU, GPU 共通
  * コードを共有したい→ bug が減る
* 計算実装は CPU と GPU で異なり、内部実装が多種多様


## Boost.Optional

* 構文はポインタとほとんど同じ
  * if でチェックし、 * で取り出す
  * C++0x なら auto が使える
* boost::optional<const T> は不可


## dtl - Diff Template Library

* diff のライブラリ (C++)
* diff template library
* 名前の由来は C++ の STL

* C++ で diff が使える
  * diff3, patch も
* 差分データを構造体として扱える
* Unified Format 対応
* 修正 BSD
* ヘッダファイルのみで hoge

* Google Code にて配布中

### 使い方

```
#include <dtl/dtl.hpp>
```

* int型配列の差分を取ったりすることも可能。
* VCS から dtl を使った diff を呼び出す

* diff3 は merge のみ可能
* patch, uniPatch

### そも diff

* ファイル間の差分を取る
  * Levenstein Distance (編集距離)
  * LCS (最長共通部分)
  * SES (最小の編集回数)
* アルゴリズム
  * 動的計画法
  * Myers
  * Wu
  * Gestalt Approach
* dtl で採用したのは Wu


## 実践 shared_ptr

### ガベコレ

* GC のほうが適した問題領域も…
* 無いものはないので我慢

### カスタムデリータ

* 動的にリソースの解放方法を決める
* 所有者が居なくなったら呼ばれる (参照カウンタ)
* 同じ型でも確保、解放処理は異なる
* shared_ptr = カウンタ + デリータ
  * 大事です

### 具体例

* boost::checked_deleter
  * 単に delete する
* boost::checked_array_deleter
  * delete[]
* 関数 (オブジェクト) なので何でもできます
  * ログ仕込んだり…
* 典型例: ハンドルのクローズ
  ```
  HHOGE raw_handle = ::OpenHoge(...);
  ::ReleaseHoge(raw_handle);
  ```
* 確実にハンドルをクローズするには…?
* 勝手に解放して下さい
  ```
  typedef
    shared_ptr<remove_pointer<HHOGE>::type>
    HogePtr;
  HHOGE raw_handle = ::OpenHoge(...);
  HogePtr hoge_h(raw_handle, ::CloseHoge);
  ```

### リソース間の依存

* 権利だけ主張する人もいます
  ```
  struct Legacy {
    Buffer* buffer;  // ちゃんと有効なバッファ指してね!
  };
  ```
* Legacy::buffer の管理は誰の責任?
* 所有権ちょいたし
  ```
  struct LegacyDeleter {
    explicit LegacyDeleter(shared_ptr<Buffer> buffer)
    : buffer(buffer) {}
    void operator()(Legacy* ptr) const { delete ptr; }
    shared_ptr<Buffer> buffer; // 所有
  };
  shared_ptr<Legacy> create(shared_ptr<Buffer> buffer) {
    LegacyDeleter deleter(buffer); // デリータが所有
    shared_ptr<Legacy> ret(new legacy(), deleter);
    ret->bufer = buffer.get();
    return ret;
  };
  ```

### Null デリータ

* 何もしないデリータです。
  ```
  struct NullDeleter {
    void operator()(void*) const{}
  };
  
  void nullDeleter(void *){
  }
  ```
* 誰かが管理しているリソース
  * コンテキストと生死を共にしている
    * スタック、data領域、TLS ...
  * 何らかの仕組みで管理されている
    * 強調できるのが一番良いのですが…
* ポインタのフリしてるだけ
  * reinterpret_cast<void *>(...)
    ```
    static vecter<int> vi;
    shared_ptr<vector<int> > vi_ptr(&vi, NullDeleter());
    
    void* val = reinterpret_cast<void*>(-1);
    shared_ptr<void> val_ptr(val, NullDeleter());
    
    shared_ptr<pair<int, int> > iip = ...;
    shared_ptr<int> iptr(&iip->first, NullDeleter());
    ```
* ところで (上記 3つ目) もし iip が有効じゃなかったら…
  * 他の手を考えましょう。
* boost もしくは 0x なら...
  * 参照カウンタの共有!
    * iip.use_count() == iptr.use_count()
  * でも tr1 にはありません…
  * tr1 なら Null デリータ + 延命
    ```
    struct FirstDeleter {
      explicet firstDeleter(shared_ptr<pair<int, int> > parent)
      :parent(parent){}
      void operator()(int*) const {};
      shared_ptr<pair<int, int> > parent;
    };
    ```

### 応用

* 参照はしたい、所有はしたくない、リソースが有効じゃなくても良い、でも安全だと嬉しい。
  * ↑お前は何を(ry
* どうしても所有してしまう
  ```
  weak_ptr<T> weak = ...;
  shared_ptr<T> ptr = weak.lock();
  ```
* リソース安全に扱うための仕組み
* 安全とか要らん
* これでもくらえ
  ```
  struct Deleter {
    void operator()(T* ptr) const { not_onwer.reset(); ... }
    shared_ptr<T> not_owner;
  } deleter;
  shared_ptr<T> owner(ptr, deleter);
  deleter.not_owner.reset(owner.get(), NullDeleter());
  weak_ptr<T> not_owner_ref = deleter.not_owner;
  ```
* ex) HWND ... ウィンドウを生成したのとは別のスレッドで解放してはいけない

### 注意

* コンテキストに注意
  * デリータはどこでも呼び出される

### みんな大好き void*

* 何でも参照できる魔法のポインタ型
* ただのポインタ
  * 参照なのか所有なのか?
  * 有効なのか無効なのか?
* shared_ptr<void>
  ```
  vector<shared_ptr<void>> delay;
  delay.push_back(make_shared<int>(1));
  delay.push_back(make_shared<string>("one"));
  ```
* 何でも入るぞ－
* カスタムデリータ－で正しく解放できる
* とりあえず寿命伸ばしたいときとかに
* 実践例: Blob
  ```
  struct Blob {
    char* data;
    size_t size;
    shared_ptr<void> resource;
  };
  ```
  ```
  Blob createBlob(const vector<char>& v){
    shared_ptr<vector<char>> res(new vector<char>(v));
    return Blob(res->data().res->size(), res);
  }
  Blob createBlob(const string& str){
    shared_ptr<string> res(new string(str));
    return Blob(res->data().res->size(), res);
  }
  ```

### 複数のリソースを返す API

* unique_ptr -> shared_ptr


## TR18015:2006を読む

* ISO C++ 標準化委員会より発行されている C++ のパフォーマンスに関する技術レポート


## ゲーム開発のC++

### 趣旨

* C++ での雰囲気

* 仕様変更に強い設計を

* new は自作
* try/catch は使わない
* ESSTL


## C++ ではまる

### C++ は難しい

* 処理系依存とか

* cin, cout は 2つの同期をもつ。遅くなるので切っとけ。
  * それでも cin.get() はさすがに遅い
* locale() に気をつけろ!
* 参考演算子を使うときは確実に期待される型に cast せよ。
