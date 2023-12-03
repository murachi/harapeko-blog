[author: T.MURACHI]
# サーバー設定日記: 2023/12/3
[ConoHa](https://www.conoha.jp/) の VPS を契約したので設定してます。

## サーバー追加
Ubuntu 22.04 を設定しました。詳細は略。

## 自分用のユーザー作成
とりあえずユーザー作ります。

```console
# useradd -m -G sudo -s /bin/bash murachi
# passwd murachi
```

## ssh で作業できるようにする設定
コントロールパネルからコンソールに入れるのですが、 ConoHa のコンソールは出来が悪くてわいのち○こパッドのモニターだと狭すぎて1画面に収まらなかったりテキストの転送が長文だと欠損したり Vim Vixen と併用できなかったりで色々と不便なので (そうでなくても緊急時以外はあんまり使うべきじゃないですね…)、公開鍵認証で ssh アクセスできるようにします。

まず sshd の設定を修正します (しれっと vim 使ってますがこれは `apt` してインストールしてあげる必要があります)。

```console
$ sudo vim /etc/ssh/sshd_config

PermitRootLogin no          # root での接続を無効
PasswordAuthentication no   # パスワード認証を無効
                            # この行は何故かファイルの末尾あたりにもあるのでそっちはコメントアウト
UsePAM no                   # PAM を無効

$ sudo service sshd restart
```

つぎに、公開鍵をサーバー側に配置します (認証鍵はすでに作ってあるので鍵を作るところは省略)。

```console
# su - murachi
$ mkdir .ssh
$ vim .ssh/authorized_keys
(公開鍵を貼り付ける)

$ chmod 700 .ssh
$ chmod 600 .ssh/authorized_keys
```

あとは普段から使い慣れたターミナルを使ってふつーに ssh してあげればつながります。

…さっきも書きましたが ConoHa のコンソールは長文のテキスト転送で欠損が起こりやすいらしく、公開鍵を貼り付ける作業でテキストの欠損が起こりやがったがために ssh しても `Permission denied (publickey).` のエラーが出てアクセスできない状態でハマってました。こんなところで… のっけから前途多難です orz
