[author: murachi]
# Boost.勉強会 \#14

* 日時: 2014/3/1 10:00 ~ 18:00
* 場所: IIJ@神保町
* レジュメとか: [Boost.勉強会 #14 東京 - boostjp](https:://sites.google.com/site/boostjp/study_meeting/study14)

## cpprefjp を支える技術

* Google Sites でリファレンスを書くのは結構大変… 統一的な書き方ができるフレームワークがほしい
* 作業環境を GitHub に移行。
  * プレーンテキストで書きたい
  * バージョン管理しやすくしたい
  * →Markdown
* Gogle Sites はそのままに… GitHub で書いた Markdown を自動的に HTML に変換して Google Sites に同期する。
* cpprefjp → HTML → GitHub (MarkDown)
* GitHub (MarkDown) → (Python) → HTML → cpprefjp
* cpprefjp → HTML
  * Google Apps Script を使用
  * 何回かに分けた (API 制限)
* HTML → Markdown
  * Ruby 1.9 の正規表現でがんがった
  * 手動で手直しもしました。→ツールは使い捨て (;_;)
* GitHub → Google Sites
  * めるぽんさんが作成、運営も…
* 変換サーバー (andare)
  * GitHub から Markdown 取得し、 HTML に変換
  * Python+Dhango製
  * Markdown という Python ライブラリを使用
* アップロードスクリプト
  * Google Apps Script を使用
* シンタックスハイライト
  * Markdown ライブラリにもともとあった。
  * 執筆者指定で特定の文字列をハイライトできるようにしたい。
    * 指定した文字列を検索し、一旦ランダムな文字列を前後に入れてから、その文字列を HTML に置換し直すというやり方をゴリゴリコーディングしましたよ
* GitHub の差分管理
  * site ブランチ
    * A - B
  * master ブランチ
    * A - B - C - D
  * master ブランチと site ブランチの違いを見て更新ファイルを判断
  * Google Sites 更新後、 master ブランチを site ブランチへ merge (上図の C と D)
* アップロード
  * スクリプトの制限
    * 実行時間は 5分まで
    * 1ファイル更新に 5秒ぐらいかかる
    * 制限1: 1回あたり 60ファイルぐらいしか更新できない
      * どこまで更新したかをストレージに保存
      * 4分過ぎたら次のタイマーを仕掛けて終了
      * 次の実行では中断したところから再開
    * …では甘かった orz
    * 制限2: ストレージの 1つのキーは 9KB まで…
    * 実行時の情報 (JSON) を格納したら漏れた
      * JSON 文字列を自動的にぶった切って複数のキーに格納
      * 取り出すときは自動的に結合して取得
  * エラー通知
    * GitHub の Issue に自動登録するようにしたよ
    * 次の実行で変換エラーがなくなれば自動で Issue を閉じるようにしてあるよ
    * 一時的なエラーの可能性もあるので失敗しても何度かはリトライ
    * エラーのあったファイルを変換サーバに POST
    * あとは変換サーバ側でいい感じに Issue に登録してくれる

## Boost.Graphの設計と、最短経路アルゴリズムの使い方いろいろ

* Boost Graph Library の設計がメインのお話。
* グラフのデータ構造とアルゴリズムのライブラリ。STL の概念に基づく。後発のグラフライブラリに大きな影響を与える。
* グラフとは…
  * 頂点 vertex と辺 edge からなるデータ構造
    * 棒グラフや折れ線グラフとは関係ない。
  * 辺が方向を持つかどうかで有向グラフ(directed)、無向グラフ(undirected) の2つの分類があるよ。
  * ex)
    * 電車の路線
    * 道路
    * network
    * ファイルの依存関係
    * クラス図
    * 行列
      * チェス盤
    * etc...
* グラフの用途
  * 最短経路を求める
  * 一筆書き
  * ネットワーク分析
    * コミュニティ、情報を広げる中心にいるのは誰か
    * コミュニティ同士を繋ぐ架け橋、またはボトルネックは誰か
  * ページランクの算出 (Google検索)
  * 迷路を解く
* 文献
  * [最短経路の本](http:://www.amazon.co.jp/%E6%9C%80%E7%9F%AD%E7%B5%8C%E8%B7%AF%E3%81%AE%E6%9C%AC-%E3%83%AC%E3%83%8A%E3%81%AE%E3%81%B5%E3%81%97%E3%81%8E%E3%81%AA%E6%95%B0%E5%AD%A6%E3%81%AE%E6%97%85-P-%E3%82%B0%E3%83%AA%E3%83%83%E3%83%84%E3%83%9E%E3%83%B3/dp/4621061364/ref=sr_1_1?ie=UTF8&qid=1393639481&sr=8-1&keywords=%E6%9C%80%E7%9F%AD%E7%B5%8C%E8%B7%AF%E3%81%AE%E6%9C%AC)
  * [オープンソースで学ぶ社会ネットワーク分析](http:://www.oreilly.co.jp/books/9784873115504/)
* グラフ型
  * adjacency_list ...隣接リスト
  * adjacency_matrix ...
* 操作
  * グラフ型への統一的な操作を提供
  * グラフ型が直接持つインタフェース(メンバ)を使うのではなく、オーバーロード可能なフリー関数を介してグラフを操作。
  * Boost.Graph のコンセプトを満たす、あらゆるグラフ型を適用可能。
    * std::vector もグラフ型として使えるよ
      ```c
      std::vector<Hoge> g;
      proc_graph(g);
      ```
  * vertices() や edges() といった**グラフコンセプトのフリー関数**を、
    グラフ構造とアルゴリズムの中間インタフェースとしている。
    * STL の反復子コンセプトと対を成すもの、と考えればいい。
* プロパティマップ
  * グラフには様々な種類のデータが付加される。
  * ex) 電車の路線図
    * 頂点 ... 駅名
    * 辺 ... 駅間の距離
    * 全体 ... 路線名
  * adjacency_list のテンプレートパラメータ
    * OutEdgeListS ... グラフの隣接構造を表すためのコンテナ
      * vecS: std::vector
      * listS: std::list
      * setS: std::set
    * VertexListS ... 頂点コンテナ
    * DirectedS ... 有向か無向か
    * VertexProperties ... 頂点
    * EdgeProperties ... 辺
    * GraphProperties ... グラフ全体
  * 取得と設定は boost::get(), boost::set() を使用。
* 使い方いろいろ…
  * dijkstra_shortest_paths() に絞ってみていくよ
  * 最短経路を求める
  * 先行マップ
    * ある頂点の前に通る頂点を集めた辞書
    * 目的地 Z を与えると、 Z の前を通る頂点 F が取得できる。
    * 逆順の最短経路を頂点のリストとして取得できる。
      ```c
      std::deque<Vertex> route;
      for(Vertex v  to; v != from; v = parents[v]){
      	route.push_front(v);
      }
      route.push_front(from);
      ```
    * 先行マップは、 dijkstra_shortest_paths() に boost::predecessor_map() という関数を通して渡していた。
      * boost::predecessor_map() : 名前付き引数を使う関数。(使わないバージョンもあるけど引数やたら多くて大変)
  * 最短経路全体でどれくらいの距離があるか知りたい場合は、 distance_map を指定する。
    ```c
    boost::dijkstra_shortest_paths(g, from,
    	boost::predecessor_map(&parents[0]).
    	distance_map(&distance[0]));
    
    int d = distance[to];
    ```
  * イベントビジター ... アルゴリズムの各ポイントで、任意の関数を呼び出せる。
    ```c
    struct MyVisitor : dijkstra_visitor<> {
    	template <class Vertex, class Graph>
    	void discover_vertex(Vertex v, const Graph&)
    	{ ...event... }
    };
    
    boost::dijkstra_shortest_paths(...
    ```
    * 特定の頂点が見つかったら例外を投げて、アルゴリズムのループを脱出する。
    * 既存のアルゴリズムを使って、新たなアルゴリズムを作る。
      * dijkstra_shortest_paths() は、幅優先探索をイベントビジターでラップして作られている。
    * コルーチンでアルゴリズムを中断
    * アルゴリズム計算状況の可視化。
* アルゴリズムの設計
  * ユーザビリティとは、その分野に精通していないユーザーにとっての公開されているアルゴリズムインタフェースのわかりやすさ。
    同時に、精通しているユーザーにとってのアルゴリズムを構成できる幅のこと。

## 一体いつから FIFOがスケールしないと錯覚していた?

* Lock-free?
  * スレッドをブロッキングせずに全体での進行が保証される
* Lock-free stack
  * 最後のノードが常に先頭…
* Lock-free queue もできるよ
* stack に積みに行くスレッドがいっぱいあると push と pop で当然渋滞するよね…
  * 対策:
    * ウェイトを入れる - 200 clock ぐらい?
      * 無駄な衝突がなくなってスループット向上。
    * 待機中にマッチングすればよくね?
      * EliminationArray
        * EliminationArray 自体にはランダムな位置に突っ込む。その一が null であることを期待して…
          * ランダムにしないとこっちはこっちで衝突しちゃうから。
* EliminationArray を FIFO に適用する - Elimination Queue
  * Queue の外に EliminationArray を設ける
  * Enqueue, Dequeu をカウントするカウンタをそれぞれ設ける
  * CAS に失敗したら EliminationArray に乗せる。このノードには Enqueue カウンタ番号がついてる。
  * Dequeu カウンタのほうが大きかったら↑を拾って持っていく。
* EliminationArray を使わなくても 8スレッドぐらいまではスケールする。
  * 1CPU を 8スレッドまでに制限するか、 EliminationArray を使うかの選択と言うのはありうるかも…

## glfw3 OpenGL を使ったGUI フレームワーク

* マルチプラットホームのフレームワーク
  * windows, linux, OS X
  * unmodified zlib/libpng license
* GLFW とは?
  * 必要最小限の機能、シンプルな構造
  * サンプルあるよ!
  * glut の貧弱な部分を強化した感じ (glut はもはやメンテナンスされてない)
  * リアルタイム性重視
  * OpenGL/ES とマッチ
  * GUI 構造や構成が C++ 向き
  * かなり前から少しずつ構築しているため、最近の流行とは逆行する部分も…
    * boost, C++11 の機能を的確に利用できてない
  * 組み込みでも使えるよ (iostream には注意…)
* 依存性
  * GLFW3, GLEW, Freetype2, OpenAL, libz, libpng, libjpeg, openjpeg, Mad(mp3), AAC, Mupdf, jbig2dec, bullet(physics)
* core
  * 機種依存性が高いコード
  * GLFW3 をラップした I/F
  * Freetype2 I/F
  * 漢字 FEP関係
  * I/O デバイスラッパー
* GL_FW
  * OpenGL/ES を使った描画クラスなど
  * OpenGL C++ ラッパー
  * フォントの描画・管理
  * 画像管理 (モーションオブジェクト)
  * ライト (開発中)
  * シーングラフ (開発中)
  * シェーダー (開発中)
* IMG_IO
  * 色、画像を扱う基本クラス
* SNG_IO
  * 単音、ストリーム
  * OpenAL との橋渡しなど
* WIDGETS
  * GUI 部品
  * Widget 管理
  * 組み込み機器にも移植可能な軽量コンパクト設計
* UTILS
  * 頂点、行列など線形代数関係
  * ファイル I/O
  * 文字列操作
  * プリファレンス
  * シーン制御
* GLFW3 の改造
  * ドラッグアンドドロップ (Windows のみ)
  * FEP 関係のメッセージ(予定)
* 開発環境
  * MinGW (MSYS)
* GUI フレームワークに期待されること
  * 日常、非日常な処理
  * ありがちな処理は _簡単な設定_ だけで可能
    * 現状では enum ハードコーディングなど (組み込み機器などを考慮すると XML ファイルとかは使いたくない)
  * 複雑で特殊な処理を実装することが可能であること
  * 拡張性
  * 予想できるスマートな挙動
  * シンプルな構造
  * マウス、ジョイスティック、タッチパッドなどでも操作が可能 (今後の課題)
* 次世代の GUI?
  * マウスでもタッチ操作でも可能なポリシー
  * Windows, OS X, X11(Qt) の操作はある程度継承
  * 「UI ガイドラインを守れば何もかもうまく行く」は幻想にすぎない
  * case by case
  * CPU もグラフィックスも十分高速なので、もっとヘビーな GUI を考えてもいいかも… (見た目だけの問題でなく)
    * ジェスチャーとか?
* アプリケーションの構造
  * メッセージを使わない
  * 毎フレーム直列同期処理
  * ゲーム向きの構造
  * シーンごとの処理
    * initialize
    * update
    * render
    * destroy

## データサイエンスワールドからC++を眺めてみる

* イントロ
  * データサイエンティストもてもて?
  * どんなツールを使ってるんだろ??
    * 1st = R, 2nd = python, 3rd = MySQL ... C/C++ は 9位
  * R は処理が遅い。
    * ループとか
  * R から C++ を呼んで高速化 ...結構多い
  * C++ から R の関数を呼ぶやりかたもあるよ
* R から C++ を使ってみる
  * Rcpp パッケージ
    * R から C++ を実行するパッケージ
    * R から C++ ソースをコンパイルして実行。
      * R から呼びたい関数にコメント行でマークアップする
* C++ での統計解析
  * Boost.Accumulators
    * 記述等軽量には対応してるけど、統計的検定、多変量解析、機械学習には対応してない模様。
    * 統計量の計算などの統計処理を提供。
    * 使い勝手は良さげ
  * Apophenia
  * ROOT
  * GSL
  * ALGLIB
    * これが一番対応範囲が多い?
    * 数値計算及びデータ処理のためのソフト
    * C++, C#, Python, VBA などから呼び出せる
    * 商用版もあるとか…
    * データ構造を文字列に持たせるなど、 C++ からの使い勝手は癖がある。
  * Rllvm なんてのもあるらしい…
  * C++ から R を呼ぶ - RInside パッケージ
    * RInside オブジェクトを生成、計算式を文字列で渡して実行、値を取り出す、ということができる。
    * Qt に R を埋め込むことも可能。
* まとめ
  * C++ で統計解析を行う良いツールがあれば @sfchaos さんまで。

* [cpprefjp でも使ってた](https:://sites.google.com/site/cpprefjp/editors_doc/random_figure)

## .NET とかが当たり前にやってること、C++だとどうするか

* C# や .NET Framework がやっていること
  * どうしてそういう機能が
  * 自前で実装するならどうやれば
  * 自前でやる上で気をつける必要がある点
* Managed
  * .Net Framework の管理下にある
    * メモリーとか
  * GC
    * Mark & Sweep 3rd
    * Compaction (メモリー移動)
    * 一度 GC かかったオブジェクトはしばらく放置
  * C++ での GC
    * 通常、参照カウント (shared_ptr, weak_ptr)
    * M&S GC lib もあるけど
      * 型がはっきりしないので保守的にならざるを
        * すべての数値をポインタとみなし Mark
  * M&S と参照カウント比較 - 一長一短
    * Throughput トータル性能は M&S のが良さげ
  * 自前メモリ管理との混在
    * スコープ短いならスタックに取ればいいのに…
      * C# は class は必ずヒープ, structはスタック
  * Python とか Go とかの事情…
* 境界チェック
  * .NET の配列は厳しい - 常に境界チェックやっとる
  * buffer overrun 病死
  * C# にもポインターはあるよ (unmanaged)
    * unsafe ブロック内のみ
    * class (参照型) はアドレス取れない
    * struct (値型) は、メンバーが全部値型であればアドレス取れる
    * コンパイルオプション /unsafe が必要。
* AppDomain - 実行環境の分離、セキュリティ保証
  * 他のアプリの機能を自分のアプリから呼びたい (Word, Excel とか)
  * サーバー上に自分のアプリを配置したい (IIS上 (ASP.NET)、SQL Server上に)
  * 単一プロセス内で、分離された複数の実行領域 (domain) を提供
  * ドメイン間には壁がある
  * ドメイン間の通信 - marshaling
    * marshalの命令通りにしか壁を超えられない
    * データの受け渡しは一旦シリアライズして相手ドメイン上のメモリーに複製を作らせる、とか。
  * 2種類あるよ
    * marshal by ref
      * const ポインタみたいなのを渡す
    * marshal by value
      * シリアライズ - BinarySerializer
  * marshal by value でも文字列は特殊扱い。
    * コピーしない
    * 参照をそのまま渡してしまう
  * コスト
    * ライブラリ読み込みはブロックごとに
      * GAC は共有される
  * C++ でも
    * COM とか
    * COM ぐらい仰々しいものが必要…
* メタデータ
  * DLL にどういう方が含まれるかという情報
  * 外部のどういう DLL を参照しているか
  * バージョン
  * プログラミング言語をまたげる
  * 動的リンクのバージョン管理
  * 動的リンク
  * メタデータの作成
    * COM の頃
      * 自分で書いてた
      * .tlb/.olb ファイル
    * .NET
      * メタデータの規格を持ってる
      * C# コンパイル時に自動生成
    * C++/CX
      * C++ Component Extensions (M$)
* JIT
  * .NET Framework の中間言語 (IL)
    * セキュリティチェックやりやすい
    * 動的リンク時にコード修正の影響を JIT で吸収
    * コンパイラ作る人と CPU 最適化やる人を分けたい
  * 中間コードまでは型情報メタデータが残るよ
    * メンバの出現順が変わっても中間コードまでは変化しない
  * Ngen.exe ... IL をネイティブコード化するツール
  * Auto-Ngen ... Ngen が Windows サービスとして動いている
  * MDIL ... 部分的にネイティブ化 (コード内容はほぼネイティブだが、型情報等は残している)
  * C++ だと
    * 動的にやろうと思ったら結局 COM とかになっちゃう
    * コンパイル時間短縮で良ければ… setter/getter とか pimpl とかが現実的
* Portable Class Library
  * 実行環境ごとに別の「標準ライブラリ」メタデータを用意
  * C++
    * ターゲット広すぎていっそう大変…
    * メタデータ持ってないと…
    * #ifdef?
* ジェネリック
  * C++ template 的な
  * C++ で generics やるとしたら…
    * 一炭汎用クラスをつくり
    * キャストだけ template で…
  * インタフェース制約かけないとメソッドすら呼べない…
    * 演算子呼べない
  * C++/CX はジェネリック持ってる - 制限キツイ
* イテレータ
  * IEnumerable<T>
  * C++ でもマクロ書けば yield return 作れちゃうYO!
  * 他の言語では generator って呼ばれることが多い
  * C++ 1y にも generator 載るかも…。
* LINQ
  * データ処理を言語統合 (SQL 的なもの)
  * ラムダ式、匿名型、拡張メソッドとの相性が良い
* dynamic
  * 動的型
  * 多重ディスパッチ
  * ダックタイピング
  * DLR 連携
  * 重い dynamic
    * 多くの場合、過剰スペック
      * JSON みたいなスキーマレスなデータ読み書き
* async/await
  * 非同期処理しんどい
    * begin/end地獄
    * event地獄
    * then地獄 (ContinueWith地獄)
  * 同期処理みたいな書き方で非同期処理が書ける\! のが await
  * イテレータ (generator) と似た仕組み
  * C++1y にも await 載るかも…
* Compiler as a Service
  * コンパイラの内部データを活用
  * IDE 連携のための機能…
    * これのためにコンパイラをフルスクラッチで書きなおしているとか何とか…
  * clang はその辺りをゴールの1つに掲げている模様…

## 新しい並列for構文のご提案

* std::thread 使おう
* ループの並列化といえば… OpenMP!
  ```c
  #Pragma omp parallel for
  for (int i= 0; i < N; i++) {
  	// ...
  }
  ```
* 並行じゃなくて並列… 早くならなきゃ意味が無い!
* タスク並列性(ex: パイプライン)、データ並列性(ex: 並列ループ)
* スレッドレベル並列 (マルチスレッドプログラミング)
  * OpenMP
    * OpenMP スレッドをまたぐ C++ 例外送出はNG ... ループ内に try - catch を書かなきゃいけない (ださい)
  * libstdc++ Parallel Mode 拡張
    * foreach で書けてお手軽
    * バックエンドは OpenMP
  * Intel TBB
    * tbb::parallel_for_each
    * 例外送出できる\!
  * Microsofot ConcRT/PPL
    * msvcrt にビルトイン
    * concurrency::parallel_for_each
    * 例外送出できる
    * MSVC/Windows 専用 orz
  * Intel CilkPlus
    * cilk_for って書くだけ
    * gcc, clang にも対応していくっぽい (現行はβ)
* ベクトルレベル並列
  * SIMD intrinsic (組み込み関数)
    * 要アセンブラ知識
    * 命令セットごとに異なるコードが必要 (SSE2, AVX, NEON...)
  * Boost.SIMD (NT^2^)
    * SIMD 処理を一般化 (x86/SSE系列、AVX と PPC/AltiVec に対応)
  * Intel CilkPlus / Array Notation
    * 宣言的なコーディング (ベクトル演算のセマンティクス)
    * clang は未対応…
  * Thrust (GPGPU; CUDA)
    * CPU向け (TBB, OpenMP) バックエンドも提供
  * Microsoft C++ AMP (GPGPU; DirectX)
    * restrict(amp) ブロック内が並列化
    * CPU/GPU処理を統一的に記述
* スレッドレベル vs ベクトルレベル
  * スレッドレベル + ベクトルレベル = C++1y ParallelTS (N3850)
    * C++ 並列ライブラリの広範な実装実験が目的
  * 実行ポリシー ...これで実行してほしいというヒント
    * vec ...ベクトル演算
    * par ...マルチスレッド
    * seq ...シングルスレッド
  * データ数が多い場合に実行ポリシーを切り替える、といった動的ポリシー可能。
  * exception_list
    * seq と par は補足
    * vec は例外使用禁止 (コケる)
  * C++標準ではない
  * まだ Working Draft 段階ですよ

## 魔導書発売記念：GPGPUの今とこれから

* OpenAcc
  * アクセラレータ向け言語仕様
  * pragma 挿入でコンパイラに指示 (C/C++/Fortran)
  * GPU だけに限定される仕様ではない
* アクセラレータとは
  * 日本語では「演算加速装置」
  * GPU
    * 特に 3D Graphics
  * NIC
  * 動画のデコーダ・エンコーダ
* 汎用アクセラレータ (割りとなんでもできるやつ)
  * GPU
    * NVIDIA, AMD, Intel
    * 特に NVIDIA GPU
    * 汎用性を獲得したのはここ4,5年
    * 画面出力機能のない GPU(?) すらある
  * Intel Xeon Phi
    * Intel MIC (Meny Integrated Core) とも
    * GPU の対抗馬
    * 画面出力機能はない
  * スモールコア・メニーコア
    * 遅くて
    * 小さくて多数のコアがある
    * メモリ帯域重視
    * データ並列向け
  * 増設カードの形で提供されている
    * アクセラレータ単体では動かせない
* 自立性
  * 子機として動作 - CPU がいないと起動できない
  * GPU
    * 自律動作はできない
    * CPU の指示に従って動作
  * MIC
    * 自立動作はできる
* プログラミング
  * CUDA C, C++ (NVIDIA)
* 小回りがきかない
  * CPU 向きの処理と GPU 向きの処理を分けたい
  * 現状のシステムではオーバーヘッドがあり難しい
    * PCI Express のレイテンシ、帯域
    * メモリー断絶
  * I/O は基本的にできない
    * 最近、GPUメモリーを直接転送できるネットワークが登場
* APU
  * AMD
  * AMD の CPU と GPU を密に結合したもの
    * CPU と GPU のメモリー結合を実現 (hUMA)
      * ポインタをそのまま GPU に渡せる\!
  * 対応ソフトウェアがまだあまりない
  * メモリー周辺が CPU のまま
    * メモリー帯域がボトルネック
* Hyper Memory Cube (HMC)
  * 次世代メモリー規格
  * NVIDIA Volta 世代で採用?
