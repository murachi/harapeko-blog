[author: murachi]
# データベーステーブル定義

## テーブル定義

### ERK_BOARD$USER_INFO

ユーザー情報テーブル。

||= フィールド名 =||= 型 =||= Index =||= 概要 =||= 備考 =||
||USER_ID||VARCHAR(64) NOT NULL||-||ユーザー ID||||
||PASSWORD||CHAR(32) NOT NULL||-||Digest (MD5) 認証用の暗号化されたパスワード。||realm は 'erk_board' とする。 Perl で Digest::MD5 を用いる場合、 「md5_hex("$user_id:erk_board:$password")」の戻り値。||
||USER_NAME||VARCHAR(64) NOT NULL||ERK_BOARD$IDX_USER_NAME||ユーザーの表示名。||||
||MAIL_ADDR||TINYTEXT||-||ユーザーのメールアドレス。||||

* PK: USER_ID

### ERK_BOARD$THREAD

スレッドテーブル。

||= フィールド名 =||= 型 =||= Index =||= 概要 =||= 備考 =||
||THREAD_NO||INT AUTO INCREMENT NOT NULL||-||スレッド識別番号。||||
||THREAD_SUBJECT||TINYTEXT NOT NULL||-||スレッドタイトル。||||
||CREATE_USER||VARCHAR(64) NOT NULL||ERK_BOARD$IDX_THREAD_USER||スレッド投稿者 ID。||||
||CREATE_DT||DATETIME NOT NULL||-||スレッド投稿日時。||||
||UPDATE_USER||VARCHAR(64) NOT NULL||ERK_BOARD$IDX_THREAD_USER||最終更新者 ID。||||
||UPDATE_DT||DATETIME NOT NULL||-||最終更新日時。||||
||EXPLANATION||TEXT NOT NULL||-||スレッドコメント。||||
||COMMENT_NO||INT NOT NULL DEFAULT 0||-||投稿されたコメントの数。||||
||IS_CLOSE||TINYINT NOT NULL DEFAULT 0||-||閉鎖フラグ。||閉鎖する場合は非零を設定する。||

* PK: THREAD_NO

### ERK_BOARD$COMMENT_ATTR

コメント属性テーブル。

||= フィールド名 =||= 型 =||= Index =||= 概要 =||= 備考 =||
||THREAD_NO||INT NOT NULL||-||スレッド識別番号。||||
||COMMENT_NO||INT NOT NULL||-||コメント番号。||ERK_BOARD$THREAD.COMMENT_NO を利用する。||
||CREATE_USER||VARCHAR(64) NOT NULL||ERK_BOARD$IDX_COMMENT_ATTR_USER||コメント投稿者 ID。||||
||CREATE_DT||DATETIME NOT NULL||-||コメント投稿日時。||||
||UPDATE_USER||VARCHAR(64) NOT NULL||ERK_BOARD$IDX_COMMENT_ATTR_USER||最終更新者 ID。||||
||UPDATE_DT||DATETIME NOT NULL||-||最終更新日時。||||
||ATTACH_NAME||VARCHAR(256)||-||添付ファイルのオリジナルファイル名。添付ファイルがない場合は NULL。||||
||ATTACH_EXT||VARCHAR(16)||-||添付ファイルの拡張子。||||
||BRANCH_NO||INT NOT NULL DEFAULT 1||-||最新の枝番。||||
||IS_DELETE||TINYINT NOT NULL DEFAULT 0||-||削除フラグ。||削除する場合は非零を設定する。||

* PK: THREAD_NO, COMMENT_NO

### ERK_BOARD$COMMENT_DETAIL

コメント内容テーブル。

||= フィールド名 =||= 型 =||= Index =||= 概要 =||= 備考 =||
||THREAD_NO||INT NOT NULL||-||スレッド識別番号。||||
||COMMENT_NO||INT NOT NULL||-||コメント番号。||||
||BRANCH_NO||INT NOT NULL||-||枝番。||ERK_BOARD$COMMENT_ATTR.BRANCH_NO を利用する。||
||UPDATE_USER||VARCHAR(64) NOT NULL||ERK_BOARD$IDX_COMMENT_DET_USER||更新者 ID。||||
||UPDATE_DT||DATETIME NOT NULL||-||更新日時。||||
||DETAIL||TEXT NOT NULL||-||コメント内容。||||

* PK: THREAD_NO, COMMENT_NO, BRANCH_NO
