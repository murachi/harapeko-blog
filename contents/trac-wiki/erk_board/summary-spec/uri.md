[author: murachi]
# URI 規則設計書

## 共通事項

* スクリプト名は、 erk-board.cgi とする。
  * CGI 版のみの暫定名で、 mod-perl 版を作る場合には変更する可能性が高い。
* スクリプトを設置する場所については、ここでは論じない。ここでは仮に / とする。
  * 例えば、スクリプトの URI は、ここでは /erk-board.cgi と記述する。
* スクリプト URI 以下の階層では、末尾にスラッシュ / がついていた場合、ついていない URI にリダイレクトする。
  * すなわち、 /erk-board.cgi/ にアクセスされた場合、 /erk-board.cgi にリダイレクトする。

## 機能別 URI 規則

### トップページ

```
/erk-board.cgi
```

* GET パラメータは無視する。
* ユーザー名表示と、スレッド一覧 (最初の 20件) を表示する。
  * 21件目以降の表示はスレッド一覧機能の URI 規則に則って表示する。

### スレッド一覧

```
/erk-board.cgi/thread-list/<ページ番号>
```

* ページ番号は 1 から始まる通し番号とする。
* スレッド一覧は常に最終更新日時の新しい順にソートされる。

### スレッド

```
/erk-board.cgi/thread/<スレッド番号>
/erk-board.cgi/thread/<スレッド番号>/p<ページ番号>
/erk-board.cgi/thread/<スレッド番号>/c<コメント番号...>
/erk-board.cgi/thread/<スレッド番号>/all
```

* ページ番号は 1 から始まる通し番号とする。
* スレッド番号で終わる URI では、ページ 1 が指定された場合と同様に表示する。
  * これらの場合、コメントは新しい順に 20件ずつ表示する。
* コメント番号指定では、カンマ、およびハイフンを用いた複数指定を可能とする。
  * ex) 3,7,15-19 とした場合、コメント番号 3, 7 および 15～19 のコメントを表示する。
* all を指定した場合、スレッド内のすべてのコメントを表示する。
* どの場合でも、スレッドタイトル、スレッドコメント、およびコメント入力フォームを表示する。

### スレッド投稿ページ

```
/erk-board.cgi/thread-form
```

### スレッド投稿 POST URI

```
/erk-board.cgi/thread-post
```

### コメント投稿 POST URI

```
/erk-board.cgi/comment-post
```

### 添付ファイル

```
/attach/<スレッド番号>/<コメント番号>.<拡張子>
```

* 1つのコメントに付けられる添付ファイルは 1つまで。

### ログイン URI

```
/erk-board.cgi/login
```

* 401 Unauthorized を返す? (検討中…)
