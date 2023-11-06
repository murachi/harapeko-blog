[author: murachi]
# さくらの VPS セットアップメモ 〜2G プラン据え置き Ubuntu16.04 入れ替え編 (パート2)〜

[パート1](wiki::HowTo/SakuraVpsSetup3) からの続きです…。

## Web サーバー設定ふたたび (Nginx 移行)

* https://github.com/murachi/nop/issues/1

最近は割といろんなものが Nginx で動くっぽいですね (というより大抵 FastCGI でどうにかするっぽい)。一度触ってみておきたかったというのもあるのですが、 Apache のモジュールモデルに若干疲れたというのもあり…。

で、構成を単純化するため、全般的に Nginx で動かす想定で、まずは Apache2 を削除してしまうことにしました。パート1 で書いた内容が全て無駄になりますが… (((((((;/\^o\^)/

```console
$ sudo su -
# apt remove apache2
# apt install nginx
```

MySQL は代わりに mariadb をインストール済みなので不要ですね。手元 PC の HOSTS 書き換えでとりあえず Nginx がちゃんと動いているのは確認。

### SSL 設定

たまたまこの時点で Let's Encrypt による `mail.harapeko.jp` ドメインが証明期限切れ近かったので、 `certbot-auto` を実行。なお Apache から Nginx に環境を換えた影響か、この時点では `./certbot-auto renew` はうまく動かなかった。

```console
# exit
$ cd ~/certbot
$ ./certbot-auto
Requesting to rerun ./certbot-auto with root privileges...

(中略... 何故か名前の設定が見つからんと言われるのでドメイン名を入力)
No names were found in your configuration files. Please enter in your domain
name(s) (comma and/or space separated)  (Enter 'c' to cancel): mail.harapeko.jp
Cert is due for renewal, auto-renewing...

(中略... どうやら Apache を使う場合と Nginx を使う場合とで設定を残す場所が異なる模様)
Deploying Certificate to VirtualHost /etc/nginx/sites-enabled/default

(ここからおせっかい機能が走り出す... せっかくなので HTTPS へリダイレクトする設定を施してもらった)
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 2
Redirecting all traffic on port 80 to ssl in /etc/nginx/sites-enabled/default

(残りの出力は略... どうやらうまくいったようだ)
$
```

実際 `http://mail.harapeko.jp/` にアクセスすると `https://mail.harapeko.jp/` にリダイレクトされるようになった。メーラーも問題なく動いている模様。

certbot による Nginx の設定の変更は `/etc/nginx/sites-available/default` に対して行われた模様。

### メインコンテンツ

`/etc/nginx/sites-available/default` の設定内容によれば、メインコンテンツは `/var/www/html` を参照している模様。ここに配置すれば良いはず。
(以下、コメント行の類は概ね省略しています)

```
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
                try_files $uri $uri/ =404;
        }
}
```

既に当該ディレクトリには旧会社時代のコンテンツを置いていたものの、 PHP 駆動だったので `index.html` は置いておらず、 `http://www.harapeko.jp/` としてアクセスすると nginx のデフォルトページが表示される状態でした。試しに `index.html` を配置してみたところ、それが代わりに表示されるようになりました。

と、ここで [https://mail.harapeko.jp/] にアクセスしてみたら同じものが表示される状態だったので、こちらは別のファイルが表示されるよう設定を修正。

```
server {
        root /var/www/vhosts/mail/html; # <--- ここを修正

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;
    server_name mail.harapeko.jp; # managed by Certbot

        location / {
                try_files $uri $uri/ =404;
        }

    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/mail.harapeko.jp/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/mail.harapeko.jp/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}
```

当該ディレクトリを作成し、 `index.html` も別途作成したところ、そちらが表示されるようになりました。

### メインサイトの名前解決

サブドメイン www を新サーバーに切り替えます。ついでなのでこのタイミングでネームサーバーの切り替えも行ってしまうことにします。

まずは旧サーバー側のネームサーバーで www の IP アドレスを新サーバーに向くように変更。

```console
# vim /var/named/chroot/var/named/harapeko.jp.zone

(...設定を修正)

# service named restart
```

```
$TTL    1D
@               IN      SOA     onaka.harapeko.jp. root.onaka.harapeko.jp. (
                        2019102101      ; serial
                        3600            ; refresh 1h
                        900             ; retry 15m
                        3600000         ; expiry 1000h
                        3600            ; minimum 24h
                )
;
                IN      NS      ns.harapeko.jp.
                IN      MX      0 mail.harapeko.jp.
mail            IN      MX      0 mail.harapeko.jp.
onaka           IN      A       49.212.128.142
ns              IN      A       49.212.128.142
mail            IN      A       153.126.157.107
www             IN      CNAME   mail    ; <--- 現状新サーバー用に使っている mail サブドメインを参照するよう変更
daiyokujo       IN      CNAME   onaka   ; for harapeko.asablo.jp/blog
blog            IN      CNAME   onaka
developer       IN      CNAME   onaka
svn             IN      CNAME   onaka
test            IN      CNAME   onaka
```

すぐには反映されない模様。やっぱりしばらくは待つ必要があるか…。

新サーバーの設定もやってしまうことにした。 chroot で動かすつもりだったが、うまく動かせなかったので、一旦 chroot を使わない設定を施しました。
chroot を使う設定については[この辺のサイト](https:://grasso0210.hatenadiary.org/entry/20170526/1495802750)にまとまっているようなので、後ほど再チャレンジしてみることにします。

設定に際しては、[この辺の記事](https:://www.eis.co.jp/bind9_src_build_4/)を参考にさせていただきました…。

設定対象ファイルをまるっと編集してしまいましょう。

```console
# cd /etc/bind
# vim -p named.conf
```

`named.conf` は他の設定ファイルを include しているだけですが、そのうち、デフォルトゾーンの設定を記述した `named.conf.default-zones` は権威 DNS サーバーを設定するなら不要なので、コメントアウトしてしまいます。

```
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
//include "/etc/bind/named.conf.default-zones";  // <--- 不要
```

`named.conf.options` では `notify no` と `recursion no` を追記、 `auth-nxdomain` は `yes` に変更しました。

```
options {
        directory "/var/cache/bind";

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        dnssec-validation auto;

        //auth-nxdomain no;    # conform to RFC1035
        auth-nxdomain yes;     // <--- ドメインが存在しない場合の応答時に AA ビットを必ず立てる設定。いちおう。
        listen-on-v6 { any; };

        notify no;     // <--- ゾーンデータの更新通知はデフォルトで no にしておき、個別のゾーンに対して yes に設定する。
        recursion no;  // <--- 自分がマスターではないゾーンについてのクエリーを、再帰的に問い合わせしない設定。権威 DNS サーバーの場合は必須。
};
```

`named.conf.local` はコメントアウトされている `zones.rfc1918` を include するの **ではなく** 、独自に作成するファイル `zones.harapeko.jp` を include するようにします。

```
// Consider adding the 1918 zones here, if they are not used in your
// organization
// include "/etc/bind/zones.rfc1918";   // <--- これはこのまま
include "/etc/bind/zones.harapeko.jp";  // <--- こっちを追記
```

そして新たに `:tabe zones.harapeko.jp` して、以下の内容を書いて保存。

```
zone "harapeko.jp" {
    type master;
    file "/var/cache/bind/harapeko.jp.zone";
    notify yes;
};
zone "107.157.126.153.in-addr.arpa" {
    type master;
    file "/var/cache/bind/harapeko.jp.rev";
    notify yes;
};
```

実際の正引き・逆引きファイルも作成します。

```console
# cd /var/cache/bind
# vim harapeko.jp.zone

(...正引きファイル編集)

# vim harapeko.jp.rev

(...逆引きファイル編集)
```

正引きファイルは旧サーバーからコピったものを一部編集。

```
$TTL    1D
@               IN      SOA     onaka.harapeko.jp. root.onaka.harapeko.jp. (
                        2019102102      ; serial
                        3600            ; refresh 1h
                        900             ; retry 15m
                        3600000         ; expiry 1000h
                        3600            ; minimum 24h
                )
;
harapeko.jp     IN      NS      ns.harapeko.jp.    ; <-- このへんちょこっと書き方を変えてみた
harapeko.jp     IN      MX  0   mail.harapeko.jp.  ; <--
onaka           IN      A       49.212.128.142
ns              IN      A       153.126.157.107    ; <---- レジストラに登録するネームサーバーを新サーバーに切り替えた想定
mail            IN      A       153.126.157.107
www             IN      CNAME   mail
daiyokujo       IN      CNAME   onaka   ; for harapeko.asablo.jp/blog
blog            IN      CNAME   onaka
developer       IN      CNAME   onaka
svn             IN      CNAME   onaka
test            IN      CNAME   onaka
```

逆引きファイルはまんまコピってシリアル番号だけ書き換えただけなので略。

あとは bind9 を登録、再起動すれば ok 。

```console
# systemctl enable bind9
# service bind9 restart
# nslookup ns.harapeko.jp localhost
Server:		localhost
Address:	::1#53

Name:	ns.harapeko.jp
Address: 153.126.157.107

#
```

旧サーバーで設定した www サブドメインの参照先変更はこの時点でとっくに反映されていましたが、せっかくなので JPDirect のホスト登録も変更します。
サイトログイン後、「ドメイン名共通 > ホスト情報変更申請」を選択し、ホスト名に「NS.HARAPEKO.JP」と入力 (ドロップメニューによる選択肢ではありません)。新しい IP アドレスを入力して完了。

### www の SSL を設定

ふつうに certbot を動かすだけでいいはず。これは root からではなく一般ユーザーから。

```console
$ ./certbot-auto -d www.harapeko.jp
Requesting to rerun ./certbot-auto with root privileges...

(中略 ...HTTP アクセス検証)
Performing the following challenges:
http-01 challenge for www.harapeko.jp
Waiting for verification...
Cleaning up challenges
Deploying Certificate to VirtualHost /etc/nginx/sites-enabled/default

(おせっかい機能のリダイレクト設定)
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 2
Redirecting all traffic on port 80 to ssl in /etc/nginx/sites-enabled/default

(どうやら成功したらしい)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations! You have successfully enabled https://www.harapeko.jp

You should test your configuration at:
https://www.ssllabs.com/ssltest/analyze.html?d=www.harapeko.jp
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

(後略...認証鍵の在り処などの情報が表示されます)

murachi@ik1-314-17353:~/certbot$
```

certbot による HTTPS へのリダイレクト設定は、 `/etc/nginx/sites-available/default` にひたすら追記する形で施されるらしい。
あとで手でファイルを分けたほうが良いかもしれんね(´･_･`)

### ブログの稼働と名前解決

ブログの引っ越し自体は済んでいるので、 PHP を動かせるようにして名前解決と SSL 化までやってしまいましょう。

ちなみに今更ですが、 `/etc/nginx/nginx.conf` を確認すると、ユーザーは apache2 の場合と同じ `www-data` で動いているようですね。

```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
```

さて、 PHP を FastCGI で動かすため、 php-fpm パッケージを導入します。

```console
# apt install php-fpm
```

Nginx の設定にブログドメイン用の設定を追加します。まず `/etc/nginx/sites-available/blog-wp` を作成。

```console
# cd /etc/nginx/sites-available
# vim blog-wp
```

内容は、いくつかのサイトを参考にさせていただいた結果、以下の通りになりました。この時点ではまだ SSL は考慮していません。

```
server {
  listen 80;
  server_name blog.harapeko.jp;

  location / {
    root /var/www/vhosts/blog/html;
    index index.php;
    try_files $uri $uri/ @wordpress;  # パーマリンクのための設定
  }

  location ~* /wp-config.php {
    deny all;
  }

  location ~ \.php$ {
    root /var/www/vhosts/blog/html;
    fastcgi_pass unix:/run/php/php7.0-fpm.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
  }

  # パーマリンク URI の場合、 index.php を使うようにする
  location @wordpress {
    root /var/www/vhosts/blog/html;
    fastcgi_pass unix:/run/php/php7.0-fpm.sock;
    fastcgi_index index.php;
    fastcgi_split_path_info ^(.+\.php)(.*)$;
    fastcgi_param SCRIPT_FILENAME $document_root/index.php;
    include fastcgi_params;
  }

  location ~ /\.ht {
    deny all;
  }
}
```

で、これのシンボリックリンクを `/etc/nginx/sites-enabled` の下に作ります。

```console
# cd ../sites-enabled
# ln -s ../sites-available/blog-wp
```

PHP の FastCGI 用の設定ファイル `/etc/php/7.0/fpm/pool.d/www.conf` については、 Nginx のユーザーを `www-data` のまま変更していないため、こちらも特に変更すべき点はありません。
(この辺、 Apache2 と併用して動かす場合なんかはユーザーを別途作って区別したほうが良いのかもしれません)

PHP-FPM をサービス登録し、 Nginx と共に再起動すれば準備完了です。

```console
# systemctl enable php7.0-fpm
Synchronizing state of php7.0-fpm.service with SysV init with /lib/systemd/systemd-sysv-install...
Executing /lib/systemd/systemd-sysv-install enable php7.0-fpm
# service php7.0-fpm restart
# service nginx restart
```

ローカルマシンの `/etc/hosts` を書き換えてアクセスしてみたところ、ちゃんとブログが動いているのを確認できました。

名前解決は `/var/cache/bind/harapeko.jp.zone` を書き換えて bind9 を restart させるだけです。 `blog` サブドメインの `CNAME` 先を `mail` に切り替えるだけなのでこれは流石に略。

ローカルマシンからサブドメイン名で新サーバーにアクセスできることを確認後、一般ユーザーから `certbot-auto` を実行し、 `blog` サブドメインも SSL 化。これも他のサブドメインの場合とやってることは変わらないので略。なお、 `https` へのリダイレクト設定もやってもらったところ、ちゃんとよしなに `/etc/nginx/sites-available/blog-wp` に対して設定を施してくれた。ホント便利 (´･_･`)

### 大浴場の画像等

まだファイルを移動してなかったですね。空っぽの perl ディレクトリはおいら自身用途を覚えていないので (\^_\^;、とりあえずスタティックファイルだけガッと移動します。

まず新サーバー側で場所を用意。

```console
# mkdir -p /var/www/vhosts/daiyokujo/html
# chown murachi:www-data /var/www/vhosts/daiyokujo/html
```

新サーバーに旧サーバーへアクセスするための秘密鍵は持ってきていないので、旧サーバーから新サーバーへ rsync 。

```console
$ cd /var/www/vhosts/daiyokujo/htdocs
$ rsync -ae ssh * murachi@mail.harapeko.jp:/var/www/vhosts/daiyokujo/html/
Enter passphrase for key '/home/murachi/.ssh/id_rsa':
$
```

新サーバー側でコピーされたファイルの所有権が `murachi:murachi` になっちゃっているのを `murachi:www-data` に修正。

```console
$ cd /var/www/vhosts/daiyokujo/html
$ sudo chown -R murachi:www-data *
```

`/etc/nginx/sites-available/daiyokujo` を作成。

```console
$ sudo su -
# cd /etc/nginx/
# vim sites-available/daiyokujo

(...大浴場用ファイル置き場の設定を記述)

# cd sites-enabled/
# ln -s ../sites-available/daiyokujo
# service nginx restart
```

内容は以下の通り。

```
server {
  listen 80;
  listen [::]:80;

  root /var/www/vhosts/daiyokujo/html;
  index index.html index.htm;
  server_name daiyokujo.harapeko.jp;

  location / {
    try_files $uri $uri/ =404;
  }
}
```

後は名前解決と SSL 化。この辺は略。

### Trac 設定

Trac インストールしてから大分時間が経っているので、 Python モジュール関連を一通り最新に upgrade してしまうことにします。

```console
$ sudo su -
# pip install --upgrade pip
# pip list --outdated --format=freeze | perl -pe 's/^(.+?)==(.+)$/$1/' | xargs pip install --upgrade
```

かなり強引ですが…。

最後の最後に

```
OSError: [Errno 2] No such file or directory: '/usr/local/lib/python2.7/dist-packages/Trac-1.2.3-py2.7.egg'
```

とかいうエラーが出ちゃいましたが、その後 `pip list --outdated` しても何も出てこないので、まぁ良しとします。

ていうかリスト確認したら Trac が 1.4 になってる…。

```console
# pip list

(2.7系は 2020年からはサポート無くなるぜという警告メッセージは略)

Package           Version
----------------- -------
Babel             2.7.0
docutils          0.15.2
Genshi            0.7.3
Jinja2            2.10.3
MarkupSafe        1.1.1
MySQL-python      1.2.5
pip               19.3.1
Pygments          2.4.2
pytz              2019.3
setuptools        41.4.0
Trac              1.4
TracFootNoteMacro 1.6
virtualenv        16.7.7
wheel             0.33.6
#
```

いつの間に 1.4 がリリースされてたの… orz

そんなわけで ideanote を upgrade

```console
# exit
$ cd /var/Developer/trac/original/ideanote
$ trac-admin .
Trac [/var/Developer/trac/original/ideanote]> upgrade
エラー: Unable to check for upgrade of trac.db.api.DatabaseManager: TracError: "mysql" データベースをサポートしていません
Trac [/var/Developer/trac/original/ideanote]>
```

ひぇっ(´･_･`)

[リリースノート](https:://trac.edgewall.org/wiki/TracDev/ReleaseNotes/1.4#MaintenanceReleases)見てみたら MySQL-python から PyMySQL に乗り換えていらっさった模様…(´･_･`)

```console
$ sudo pip install PyMySQL

(...インストール)

$ trac-admin .
trac-admin 1.4 へようこそ
Trac 管理コンソール(対話モード)です。
Copyright (C) 2003-2019 Edgewall Software

'?' コマンドか 'help' コマンドでヘルプを表示します。

Trac [/var/Developer/trac/original/ideanote]> upgrade
アップグレードが失敗しました。問題を解消させてもう一度試してください。

TracError: すべてのテーブルは照合順序として utf8_bin または utf8mb4_bin で作成されている必要があります。次のテーブルがその照合順序で作成されていません: attachment, auth_cookie, cache, component, enum, milestone, node_change, permission, report, repository, revision, session, session_attribute, system, ticket, ticket_change, ticket_custom, version, wiki
Trac [/var/Developer/trac/original/ideanote]>
```

あー、前回中断したときと同じエラーまでたどり着きました(´･_･`)

えーっと。こんな感じかな。

```console
$ cd
$ echo attachment, auth_cookie, cache, component, enum, milestone, node_change, permission, report, repository, revision, session, session_attribute, system, ticket, ticket_change, ticket_custom, version, wiki >trac_tables.txt
$ perl -e '$l=<>; chomp $l; @t=split /\s*,\s*/, $l; print "alter table $_ collate utf8mb4_bin;\n" for @t' trac_tables.txt > trac_tables_mod_u8b.sql
$ mysql -u trac_master -pパスワード trac_ideanote < trac_tables_mod_u8b.sql
```

これでどうか。

```console
$ cd /var/Developer/trac/original/ideanote/
$ trac-admin .
trac-admin 1.4 へようこそ
Trac 管理コンソール(対話モード)です。
Copyright (C) 2003-2019 Edgewall Software

'?' コマンドか 'help' コマンドでヘルプを表示します。

Trac [/var/Developer/trac/original/ideanote]> upgrade
アップグレードが失敗しました。問題を解消させてもう一度試してください。

TracError: データベースのキャラクタセットと照合順序が 'utf8' と 'utf8_general_ci' になっています。データベースは (('utf8', 'utf8_bin'), ('utf8mb4', 'utf8mb4_bin')) の1つでなければいけません。
Trac [/var/Developer/trac/original/ideanote]>
```

(ﾟДﾟ)ﾊｧ?

そもそも今の DB の状態で utf8mb4 を選ぶべきではなかったかも(´･_･`)
`'utf8', 'utf8_bin'` の組み合わせで行きますか(´･_･`)

```console
$ cd
$ perl -e '$l=<>; chomp $l; @t=split /\s*,\s*/, $l; print "alter table $_ collate utf8_bin;\n" for @t' trac_tables.txt > trac_tables_mod_u8b.sql
$ mysql -u trac_master -pパスワード trac_ideanote < trac_tables_mod_u8b.sql
murachi@ik1-314-17353:~$ sudo mysql -u root
[sudo] murachi のパスワード:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 6829
Server version: 10.0.38-MariaDB-0ubuntu0.16.04.1 Ubuntu 16.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> alter database trac_ideanote collate utf8_bin;
Query OK, 1 row affected (0.01 sec)

MariaDB [(none)]> Bye
$
```

今度こそどうだ…

```console
$ cd /var/Developer/trac/original/ideanote/
$ trac-admin .
trac-admin 1.4 へようこそ
Trac 管理コンソール(対話モード)です。
Copyright (C) 2003-2019 Edgewall Software

'?' コマンドか 'help' コマンドでヘルプを表示します。

Trac [/var/Developer/trac/original/ideanote]> upgrade
レポート 1, 2, 3, 4, 5, 7, 8 をアップグレードできなかったため
"ambiguous column name" エラーが起きないように手動で変更する必要が
あるかも知れません。
詳細は https://trac.edgewall.org/wiki/1.3/TracUpgrade#enum-description-field を参照してください。

アップグレードが終了しました。

次のコマンドを実行すると Trac のドキュメントをアップグレードできます:

  trac-admin "/var/Developer/trac/original/ideanote" wiki upgrade
Trac [/var/Developer/trac/original/ideanote]>
```

一部で失敗しているみたいですが、とりあえずアップグレードを完了させられました。

さて、ここまでやったところで Trac リポジトリの所有者が `murachi:murachi` になっちゃっていたことに気づいたので、所有者を変更。

```console
$ cd /var/Developer/
$ sudo chown -R murachi:www-data trac
$ sudo chmod g+s trac
```

リポジトリの準備が出来たと思うので、 uWSGI を使って動かしてみることにします。 uWSGI に関しては以下のサイトを参考にさせていただいています。

* [uWSGI入門 - Python学習講座](https:://www.python.ambitious-engineer.com/archives/1959)
* [ちゃんと運用するときのuWSGI設定メモ](https:://qiita.com/yasunori/items/64606e63b36b396cf695)
* [nginx + uwsgi でアプリケーションをサブディレクトリで動かす設定](https:://qiita.com/methane/items/e0949a37c112eedf2b74)
* [Nginx + uwsgi で Trac環境を作った時のメモ](http:://laugh-labo.blogspot.com/2012/02/nginx-uwsgi-trac.html)

uWSGI をインストール。 pip で入れちゃいましょう。

```console
$ sudo su -
# pip install uWSGI
Collecting uWSGI
  Downloading https://files.pythonhosted.org/packages/e7/1e/3dcca007f974fe4eb369bf1b8629d5e342bb3055e2001b2e5340aaefae7a/uwsgi-2.0.18.tar.gz (801kB)
     |████████████████████████████████| 808kB 2.1MB/s
Building wheels for collected packages: uWSGI
  Building wheel for uWSGI (setup.py) ... done
  Created wheel for uWSGI: filename=uWSGI-2.0.18-cp27-cp27mu-linux_x86_64.whl size=519527 sha256=0e76554e1428e185c0ccb066bb4afaeac1c3c97da5af73c127dd86ccc531f89a
  Stored in directory: /root/.cache/pip/wheels/2d/0c/b0/f3ba1bbce35c3766c9dac8c3d15d5431cac57e7a8c4111c268
Successfully built uWSGI
Installing collected packages: uWSGI
Successfully installed uWSGI-2.0.18
#
```

WSGI ファイルは既にあるので、 `uwsgi` コマンドを使えば適当なポートで動作確認できるはず。動作確認用に穴を開けますか。

```console
# cd
# vim -p ip*tables.sh

(両方とも編集...)

# ./iptables.sh
# ./ip6tables.sh
# service netfilter-persistent save
# service netfilter-persistent reload
```

`iptables.sh` と `ip6tables.sh` は以下の通りに穴を開けたいポート番号を追記するだけ。

```
#/sbin/ip6tables -A INPUT -p tcp -m state --state NEW -m multiport --dports 22,25,587,993,80,443 -j ACCEPT
/sbin/ip6tables -A INPUT -p tcp -m state --state NEW -m multiport --dports 22,25,587,993,80,443,6543 -j ACCEPT
```

uwsgi を実行してみます。

```console
# exit
$ cd /var/Developer/trac/original
$ uwsgi --http=0.0.0.0:6543 --wsgi-file=trac-original.wsgi --callable=application
```

http://mail.harapeko.jp:6543/ideanote にアクセスしてみたところ、それっぽい画面が表示されました。ちなみに `--callable` 引数に指定するのは実際の WSGI ファイルが caller として用意しているオブジェクト名 (関数名) で良いみたいです。

uWSGI の動作確認は出来たので、これを Nginx 経由で動かせるように設定していきます。まずは uwsgi のパラメータを定義するファイルを `/var/Developer/trac/original/trac-original.ini` に作成します。

```console
$ vim trac-original.ini
```

内容としては以下の通りになりました。

```
[uwsgi]
uid = www-data
gid = www-data

current_release = /var/Developer/trac/original
chdir = %(current_release)

#wsgi-file = %(current_release)/trac-original.wsgi
mount = /trac/original=%(current_release)/trac-original.wsgi
callable = application
manage-script-name = true
socket = /var/Developer/trac/run/uwsgi-original.sock
pidfile = /var/Developer/trac/run/uwsgi-original.pid
chmod-socket = 666
vacuum = true

daemonize = /var/Developer/trac/log/uwsgi-original.log
log-reopen = true
log-maxsize = 5242880
logfile-chown = on
logfile-chmod = 644

processes = 1
threads = 4
max-requests = 500
max-requests-delta = 300
master = true
```

軽く解説入れます。

`uwsgi` コマンドを実行するときのカレントディレクトリにしたいディレクトリを指定したい場合、 `current_release` にそのディレクトリパスを指定し、それと同じ値を `chdir` にも指定すれば良いようです。

`wsgi-file` パラメータを指定する代わりに `mount` パラメータを使用しています。これを使うと、実際に公開するときの URL prefix を指定できます。 Idea note は `http(s)://developer.harapeko.jp/trac/original/ideanote` として公開したいので、 prefix に `/trac/original` を指定しています。

`mount` パラメータは複数指定できるらしいので、同じ場所に複数アプリを置いて同時に公開するような書き方もできるっぽいです。

`manage-script-name` は URI prefix を uWSGI 側で管理するかどうかを決めるパラメータで、 `true` を指定すると、リクエストに渡された URL から `mount` に指定した prefix を取り除いた値を URL としてアプリに渡すようになります。例えば `/trac/original/ideanote/wiki/HowTo` という URL でリクエストされた場合、これを `/ideanote/wiki/HowTo` に置き換えた値がアプリに渡されます。これを実現するために Nginx 側の設定を工夫する方法もあるのですが (`location` を正規表現指定にして抜き出したい部分をキャプチャし、 `SCRIPT_NAME` と `PATH_INFO` を指定する)、今回はこの方法で書いてみました。

HTTP の公開は Nginx に任せるので、 Nginx との通信のために UNIX socket だけを公開します。それを指定するのが `socket` パラメータです。 `pidfile` パラメータはプロセス ID を書き出すファイルで、デーモン実行した uWSGI プロセスを閉じるのに必要になるものです。 `vacuum = true` を指定しておくと、デーモンを開く際にソケットファイルや PID ファイルが残っていても、ちゃんとクリアしてから起動してくれるようになります。

`deamonize` パラメータを指定すると、 uWSGI はデーモンモードで起動するようになります。パラメータには実行中のログを吐き出すファイル名を指定します。 `master` パラメータはマスタープロセスを持つべきか否かを指定するもので、デーモン実行する場合は通常 `true` を指定するものらしいです ([この辺](https:://stackoverflow.com/questions/35072176/what-is-the-uwsgi-master-process-for)に詳しい解説がありました)。

重要そうなのはそんなところでしょうか。

Unix ソケットと PID ファイルを置くディレクトリと、ログファイルを吐き出すディレクトリを用意します。これらのディレクトリはオーナーが `murachi:www-data` になるので、グループに書き込み権限を与えておきます。

```console
$ cd ..
$ mkdir run
$ mkdir log
$ chmod g+w run log
```

uWSGI の準備は整ったので、とりあえず動かしておきましょう。

```console
$ cd original/
$ uwsgi --ini trac-original.ini
```

デーモンモードで動くので、すぐにシェルに処理が返されます。先程作った `run` ディレクトリの下にソケットファイルと PID ファイルが生成されているのが確認できます (そのへんは省略)。

Nginx の設定も施しましょう。 `/etc/nginx/sites-available/developer` に `developer.harapeko.jp` サブドメイン用の設定を作成します。

```console
$ sudo su -
# cd /etc/nginx/
# vim sites-available/developer

(設定を記述...)

# cd sites-enabled/
# ln -s ../sites-available/developer
```

設定内容はとりあえず以下の通りです。(HTTPS 化は相変わらず certbot 任せ)

```
server {
  listen 80;
  listen [::]:80;

  root /var/www/vhosts/developer/html;
  index index.html index.htm;
  server_name developer.harapeko.jp;

  location /trac/original {
    include uwsgi_params;
    uwsgi_pass unix:/var/Developer/trac/run/uwsgi-original.sock;
  }

  location ~ ^/trac/original/[^/]+/login$ {
    include uwsgi_params;
    uwsgi_param REMOTE_USER $remote_user;
    uwsgi_pass unix:/var/Developer/trac/run/uwsgi-original.sock;
    auth_basic "trac authentication";
    auth_basic_user_file /var/Developer/trac/original/.htpasswd;
  }

  location / {
    try_files $uri $uri/ =404;
  }
}
```

当面 developer サブドメインを Idea Note 以外の用途で使う予定はないのですが、一応 `/trac/original` 以外のところにアクセスされた場合用にドキュメントルートも設定してあります。ファイルは用意していないので単に 404 になります。

Trac 用の設定としては、基本は単に UNIX ソケットとの通信設定を施すのみです。 `uwsgi_params` というのを include していますが、このファイルは `pip install uWSGI` した時点で `/etc/nginx` の下によしなに作られたようで (もしくは元からそこにあった?)、 uWSGI に渡す基本的なパラメータのデフォルト値が定義されています。

Trac ではログイン処理も施す必要があるので、ログイン用の URL にアクセスした場合に `REMOTE_USER` パラメータに基本認証されたユーザー名を渡す設定と、基本認証自体が動くようにする設定を追加しています。なお、 Nginx には Apache2 で使えていた **Digest 認証には対応していません** (対応させるにはパッチを使ってソースからコンパイルしてあげる必要がある)。元々 Digest 認証用のパスワードファイルがあって、それを使っていたのですが、今見たら昔仕事で共用していたユーザーの設定がそのまま残っていて、むしろこれは捨てたほうが良いなと思ったので、基本認証に切り替えて済ませることにしました (SSL を使うつもりなので問題はないはず)。

ドキュメントルートを作っていなかったので用意します。

```console
# mkdir -p /var/www/vhosts/developer/html
# chown murachi:www-data /var/www/vhosts/developer/html
```

それから基本認証用のパスワードファイルも用意します。

```console
# exit
$ cd /var/Developer/trac/original/
$ htpasswd -c .htpasswd murachi
New password:
Re-type new password:
Adding password for user murachi
$
```

Nginx を再起動します。

```console
$ sudo service nginx restart
```

ローカルマシン上の `/etc/hosts` を書き換えて、所定の URL で Idea Note にアクセスできることが確認できました。

もはや動作確認用のポート開放は不要なので、 iptables の設定をもとに戻しておきましょう。

```console
$ sudo su -
# vim ip*tables.sh

(ポート 6543 を含まない設定に戻す)

# ./iptables.sh
# ./ip6tables.sh
# service netfilter-presistent save
# service netfilter-presistent reload
```

名前解決と SSL の設定も施します。これは略。

さて、 uWSGI はデーモンとして動いてはいますが、サーバーを再起動した場合に自動で起動する設定にはなっていません。最後にその設定を施しておくことにします。

最初は頑張って `/etc/init.d` の下にスクリプトを書こうかとも思ったのですが、 systemd を使う方法が楽そうだったのでそっちにしました。
`/etc/systemd/system/trac.service` ファイルに設定を記述すれば、 `trac` という名前のサービスを登録することが出来ます。

```console
$ sudo vim /etc/systemd/system/trac.service
```

記述内容は以下の通り。

```
[Unit]
Description=Trac service on uWSGI
After=network.target
ConditionPathExists=/var/Developer/trac/original

[Service]
WorkingDirectory=/var/Developer/trac/original
ExecStart=/usr/local/bin/uwsgi --ini trac-original.ini
ExecReload=/usr/local/bin/uwsgi --stop /var/Developer/trac/run/uwsgi-original.pid
ExecStop=/usr/local/bin/uwsgi --stop /var/Developer/trac/run/uwsgi-original.pid
Restart=on-failure
Type=forking

[Install]
WantedBy=multi-user.target
```

uWSGI はデーモンモードで動かす場合、起動時に処理をすぐにシェルに返す (=子プロセスを起動しっぱなしにして PID ファイルを置いて終了する) ので、 `Type` は `forking` にしておきます。 `User` は不要です (恐らく起動/終了は root で実行されます… アプリケーションの実行ユーザーは uWSGI 側の設定ファイルに記述されているのでそこは気にする必要ないです)。

既存の設定ファイルやネットでの情報を見ていて `ExecReload` があれば `ExecStop` は不要なのかとも思ったのですが、両方書いておいたほうが終了を伴う操作時の動作は安定するような気がします (この辺まだちょっと判然とはしてません…)。なお、 `ExecReload` には単に終了するコマンドを記述します (一般的には `/bin/kill -HUP $MAINPD` を呼ぶケースが多いようです…もちろんこれは `Type = simple` や `Type = notify` で動いて systemd 自身が PID を握っている場合の書き方ですが…)。

サービスを登録し、サービス経由で起動します。念のため、事前に手動で uWSGI を終了しておきましょう。

```console
$ uwsgi --stop /var/Developer/trac/run/uwsgi-original.pid
$ sudo systemctl enable trac
$ sudo service trac start
```

Idea Note がちゃんと動いていることを確認。 PID ファイルや UNIX ソケットファイルの存在も確認。 `restart` でちゃんと PID ファイルの PID が変化することや、サーバーを `reboot` しても自動で起動することなども確認済みです。

### WebDAV 設定

なんと、ここに来て、 [nginx](https:://packages.ubuntu.com/xenial/nginx) パッケージとしてインストールした Nginx では WebDAV サポートが不十分であることが発覚しました。

Nginx には静的モジュールおよび動的モジュールとして様々なモジュールを追加できるのですが、それらは Nginx のビルド時に指定する必要があります。 Ubuntu (16.04) では利便性のため、用途のレベルに応じて採用されるモジュールの数が異なる以下の 4つのバージョンのパッケージを提供しています。

* [nginx-core](https:://packages.ubuntu.com/xenial/nginx-core)
* [nginx-extras](https:://packages.ubuntu.com/ja/xenial/nginx-extras)
* [nginx-full](https:://packages.ubuntu.com/xenial/nginx-full)
* [nginx-light](https:://packages.ubuntu.com/xenial/nginx-light)

で、普通に `apt install nginx` を行った場合、インストールされるのは nginx-core になるようで、これには公式の [http_dav_module](http:://nginx.org/en/docs/http/ngx_http_dav_module.html) は含まれるものの、サードパーティの [nginx-dav-ext-module](https:://github.com/arut/nginx-dav-ext-module) は含まれないのだそうです。

いずれも若干情報が古いですが、インストールするパッケージに応じて具体的にどのへんのモジュールが適用されるのかについては以下のサイトにまとめられていました。

* [nginx の light,full,extra の違い ( ubuntu編 ) - 紐付けな日々](http:://blog.it.churaumi.tv/nginx-light-full-extra-configure-compare)

また、 Debian のパッケージングに係る立場の人から、これらのバージョンごとの違いについての説明が寄せられている Q&A が以下にありました。

* [What is the difference between the core, full, extras and light packages for nginx? - Ask Ubuntu](https:://askubuntu.com/questions/553937/what-is-the-difference-between-the-core-full-extras-and-light-packages-for-ngi)

nginx-core は nginx-full からサードパーティ製のモジュールを取り除いたもの、ということらしいです。

で、 WebDAV を動かすのに何故サードパーティモジュールである [nginx-dav-ext-module](https:://github.com/arut/nginx-dav-ext-module) が必要になるのかというと、 WebDAV をいわゆる Web フォルダとして使用するために必要となるいくつかの HTTP ステートメントのサポートが、公式のモジュールには含まれておらず、それを補完するのがこのモジュールであるから、ということみたいです。特に `PROPFIND` (属性情報の取得、ディレクトリのファイル一覧を取得するのにも必要) と `OPTIONS` (サポートする HTTP ステートメントを確認する、これが確認できないとそもそも動作しない WebDAV クライアントもある) はどう考えても必須です。

そんなわけで、 nginx-core を捨てて、 nginx-extras か nginx-full に入れ替える必要があります。 extras にあって full にない機能については、 Perl スクリプトを埋め込む機能や、 MP4 や flv をストリーミングする機能などがあるようですが、すぐには使わないかなと思ったので、 nginx-full を採用することにしました。

まずは nginx-core をアンインストールします…

```console
# apt remove nginx-core
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
  apache2-data apache2-utils linux-headers-4.4.0-138 linux-headers-4.4.0-138-generic linux-headers-4.4.0-62
  linux-headers-4.4.0-62-generic linux-headers-4.4.0-78 linux-headers-4.4.0-78-generic linux-image-4.4.0-138-generic
  linux-image-4.4.0-62-generic linux-image-4.4.0-78-generic linux-image-extra-4.4.0-138-generic
  linux-image-extra-4.4.0-62-generic linux-image-extra-4.4.0-78-generic
Use 'apt autoremove' to remove them.
The following additional packages will be installed:
  nginx-full
Suggested packages:
  nginx-doc
The following packages will be REMOVED:
  nginx-core
The following NEW packages will be installed:
  nginx-full
0 upgraded, 1 newly installed, 1 to remove and 0 not upgraded.
Need to get 454 kB of archives.
After this operation, 71.7 kB of additional disk space will be used.
Do you want to continue? [Y/n]
```

なんと、驚くべきことに、ここで nginx-core だけを消そうとすると、 nginx の依存性解決のために、代わりに nginx-full をインストールするというメッセージが表示されます。まさにやろうとしていたことがこれだけで解決されるようです (想像もしていなかった挙動 \^_\^;)。

やってしまいましょう。

```console
Do you want to continue? [Y/n] Y
Get:1 http://jp.archive.ubuntu.com/ubuntu xenial-updates/universe amd64 nginx-full amd64 1.10.3-0ubuntu0.16.04.4 [454 kB]
Fetched 454 kB in 0s (2,130 kB/s)
dpkg: nginx-core: dependency problems, but removing anyway as you requested:
 nginx depends on nginx-core (>= 1.10.3-0ubuntu0.16.04.4) | nginx-full (>= 1.10.3-0ubuntu0.16.04.4) | nginx-light (>= 1.10.3-0ubuntu0.16.04.4) | nginx-extras (>= 1.10.3-0ubuntu0.16.04.4); however:
  Package nginx-core is to be removed.
  Package nginx-full is not installed.
  Package nginx-light is not installed.
  Package nginx-extras is not installed.
 nginx depends on nginx-core (<< 1.10.3-0ubuntu0.16.04.4.1~) | nginx-full (<< 1.10.3-0ubuntu0.16.04.4.1~) | nginx-light (<< 1.10.3-0ubuntu0.16.04.4.1~) | nginx-extras (<< 1.10.3-0ubuntu0.16.04.4.1~); however:
  Package nginx-core is to be removed.
  Package nginx-full is not installed.
  Package nginx-light is not installed.
  Package nginx-extras is not installed.
 nginx depends on nginx-core (>= 1.10.3-0ubuntu0.16.04.4) | nginx-full (>= 1.10.3-0ubuntu0.16.04.4) | nginx-light (>= 1.10.3-0ubuntu0.16.04.4) | nginx-extras (>= 1.10.3-0ubuntu0.16.04.4); however:
  Package nginx-core is to be r
(Reading database ... 230375 files and directories currently installed.)
Removing nginx-core (1.10.3-0ubuntu0.16.04.4) ...
Selecting previously unselected package nginx-full.
(Reading database ... 230373 files and directories currently installed.)
Preparing to unpack .../nginx-full_1.10.3-0ubuntu0.16.04.4_amd64.deb ...
Unpacking nginx-full (1.10.3-0ubuntu0.16.04.4) ...
Processing triggers for man-db (2.7.5-1) ...
Setting up nginx-full (1.10.3-0ubuntu0.16.04.4) ...
# nginx -V
nginx version: nginx/1.10.3 (Ubuntu)
built with OpenSSL 1.0.2g  1 Mar 2016
TLS SNI support enabled
configure arguments: --with-cc-opt='-g -O2 -fPIE -fstack-protector-strong -Wformat -Werror=format-security -Wdate-time -D_FORTIFY_SOURCE=2' --with-ld-opt='-Wl,-Bsymbolic-functions -fPIE -pie -Wl,-z,relro -Wl,-z,now' --prefix=/usr/share/nginx --conf-path=/etc/nginx/nginx.conf --http-log-path=/var/log/nginx/access.log --error-log-path=/var/log/nginx/error.log --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --with-debug --with-pcre-jit --with-ipv6 --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_addition_module --with-http_dav_module --with-http_geoip_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_image_filter_module --with-http_v2_module --with-http_sub_module --with-http_xslt_module --with-stream --with-stream_ssl_module --with-mail --with-mail_ssl_module --with-threads --add-module=/build/nginx-U6uhdy/nginx-1.10.3/debian/modules/nginx-auth-pam --add-module=/build/nginx-U6uhdy/nginx-1.10.3/debian/modules/nginx-dav-ext-module --add-module=/build/nginx-U6uhdy/nginx-1.10.3/debian/modules/nginx-echo --add-module=/build/nginx-U6uhdy/nginx-1.10.3/debian/modules/nginx-upstream-fair --add-module=/build/nginx-U6uhdy/nginx-1.10.3/debian/modules/ngx_http_substitutions_filter_module
#
```

ちゃんと `nginx-dav-ext-module` も入ってます。素晴らしい…。

それでは実際の設定に入っていきます。まず、 WebDAV のためにサブドメイン `dav.harapeko.jp` を作成します。

```console
# vim /var/cache/bind/harapeko.jp.zone
```

```
$TTL    1D
@               IN      SOA     ns.harapeko.jp. root.harapeko.jp. (
                        2019110401      ; serial
                        3600            ; refresh 1h
                        900             ; retry 15m
                        3600000         ; expiry 1000h
                        3600            ; minimum 24h
                )
;
@               IN      NS      ns.harapeko.jp.
harapeko.jp     IN      MX  0   mail.harapeko.jp.
ns              IN      A       153.126.157.107
mail            IN      A       153.126.157.107
onaka           IN      A       49.212.128.142
www             IN      CNAME   mail
daiyokujo       IN      CNAME   mail    ; for harapeko.asablo.jp/blog
blog            IN      CNAME   mail
developer       IN      CNAME   mail
dav             IN      CNAME   mail
```

`SOA` レコードを今更修正。もはや使う予定のない `svn` や `test` を削除。そして `dav` を追加。

```console
# vim /var/cache/bind/harapeko.jp.rev
```

```
$TTL 1D
@       IN      SOA     ns.harapeko.jp.      root.harapeko.jp. (
                2019110401      ; Serial
                3600            ; Refresh 1h
                900             ; Retry 15m
                3600000         ; Expire 1000h
                3600            ; Minimum
        )
;
        IN      NS      ns.harapeko.jp.
```

zone の `SOA` を変更したのでこっちも合わせて変更。

```console
# service bind9 restart
```

SOA をちゃんと指定したからか、今度はすぐにローカルマシンから `dav.harapeko.jp` が見えるようになった。素晴らしい。

次は Nginx ですが、 WebDAV の設定はまだしない。なぜなら SSL を設定してセキュリティを確保してからじゃないと危ないから。

```console
# cd /etc/nginx/sites-available/
# vim dav

(設定を記述...)

# cd ../sites-enabled/
# ln -s ../sites-available/dav
# service nginx restart
```

```
server {
    listen 80;
    listen [::]:80;
    server_name dav.harapeko.jp;
    root /var/www/vhosts/dav/html;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

とりあえずの設定はこんなんでいいかな。 404 だとちょっと悲しいのでフォルダも用意しましょう。

```console
# cd /var/www/vhosts/
# mkdir -p dav/html dav/files
# chown -r murachi:www-data dav
# chmod g+w dav/*
# chmod g+s dav/*
# ls -la dav
total 16
drwxr-xr-x 4 root    root     4096 11月  4 17:17 .
drwxr-xr-x 7 root    root     4096 11月  4 17:17 ..
drwxrwsr-x 2 murachi www-data 4096 11月  4 17:17 files
drwxrwsr-x 2 murachi www-data 4096 11月  4 17:17 html
# exit
$ cd /var/www/vhosts/dav/html/
$ vim index.html

(内容はてきとう)
```

そしたら certbot を使って SSL の設定をします。作業内容は省略。もちろん HTTP -> HTTPS にリダイレクトする設定も施しました。

ここでいよいよ WebDAV の設定です。もう一度 Nginx の設定ファイルを開いて編集します。

```console
$ sudo su -
# cd /etc/nginx/sites-available/
# vim dav
```

設定内容の全容は以下の通りです。

```
server {
    server_name dav.harapeko.jp;
    root /var/www/vhosts/dav/files;

    location / {
        charset utf-8;

        autoindex on;
        autoindex_exact_size off;
        autoindex_localtime on;

        # enable creating directories without trailing slash (MaxOS/iOS bug support)
        set $x $uri$request_method;
        if ($x ~ [^/]MKCOL$) {
            rewrite ^(.*)$ $1/;
        }

        dav_methods PUT DELETE MKCOL COPY MOVE;
        dav_ext_methods PROPFIND OPTIONS;
        #dav_ext_methods PROPFIND OPTIONS LOCK UNLOCK;
        #dav_ext_lock zone=harapeko_dav;

        dav_access group:rw all:r;
        client_body_temp_path /var/www/vhosts/dav/tmp;
        create_full_put_path on;

        auth_basic "WebDAV Authentication";
        auth_basic_user_file /var/www/vhosts/dav/.htpasswd;
    }

    listen [::]:443 ssl; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/dav.harapeko.jp/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/dav.harapeko.jp/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}

server {
    if ($host = dav.harapeko.jp) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    listen 80;
    listen [::]:80;
    server_name dav.harapeko.jp;
    return 404; # managed by Certbot


}
```

[nginx-dav-ext-module 公式リポジトリ](https:://github.com/arut/nginx-dav-ext-module)の README を見ながら `LOCK`, `UNLOCK` にも対応させてみたかったのですが、 `dav_ext_lock_zone` ディレクティブなんて知らんぞ、と怒られてしまったので (モジュールのバージョンが古いのかも… orz)、とりあえずロックなしで設定してしまっています。

`autoindex` 関連はブラウザでアクセスしたときにとりあえずファイル一覧を見せておく用です。

`root` は WebDAV のファイル置き場にするディレクトリパスに変えておきます。 `index` はもはや不要なので削除。

で、以下の部分ですが、

```
        # enable creating directories without trailing slash (MaxOS/iOS bug support)
        set $x $uri$request_method;
        if ($x ~ [^/]MKCOL$) {
            rewrite ^(.*)$ $1/;
        }
```

これは `MKCOL` リクエスト (=コレクション作成、よーするにディレクトリ作成) が来たにも関わらず URI がスラッシュで終わっていない場合に、スラッシュ付きの URI に rewrite してあげるというもので、 MacOS や iOS で実際この手のリクエストが来てエラーになる場合があるらしいことへの対応らしいです。想定されるユーザーに Apple ユーザーがいる場合は必須ですね (うちには居ないんですが)。

SSL に関する設定内容は certbot が勝手にやってくれたものをそのまま残しています。

基本認証の設定も書き加えていますが、肝心のパスワードファイルがまだ未作成なので作成します。また、 WebDAV 用のテンポラリファイルも作っておきます。

```console
# cd /var/www/vhosts/dav
# htpasswd -c .htpasswd murachi
New password:
Re-type new password:
Adding password for user murachi
# mkdir tmp
# chown murachi:www-data .htpasswd tmp
# ls -la
total 24
drwxr-xr-x 5 root     root     4096 11月  4 23:51 .
drwxr-xr-x 7 root     root     4096 11月  4 17:17 ..
-rw-r--r-- 1 murachi  www-data   46 11月  4 22:41 .htpasswd
drwxrwsr-x 3 murachi  www-data 4096 11月  5 00:00 files
drwxrwsr-x 2 murachi  www-data 4096 11月  4 17:23 html
drwxr-xr-x 2 www-data www-data 4096 11月  4 23:51 tmp
#
```

これで Nginx を再起動すれば、 WebDAV が使えるようになるはずです…。

```console
# service nginx restart
```

Ubuntu のファイラーからは無事にアクセスできるようになったのですが、 Windows7 の Web フォルダ機能からは接続できませんでした…。 CarotDAV というツールを使えばいけるのですが… あとファイルアップロードで上書きがどうこうとかいう警告が出ちゃいますね (Ubuntu でも Win + CarotDAV でも)。これだったら sftp とかでも使い勝手はそこまで変わらんかなぁ… (´･_･`)

## 引越し作業仕上げ

移行したいものについては全て済んだので、仕上げします。

まず名前解決でもはや不要な onaka サブドメインはこの際削除します。作業内容は省略 (シリアル番号変えて行削除するだけなので…)。

それから旧サーバーをシャットダウンし (これはさくらの VPS のコントロールパネルから行いました)、解約手続きも行いました (来年 5月いっぱいまでは使えるらしい…)。

図らずも 2台体制で余計にお金がかかっていたのもこれで解消されます… orz

## 反省

なかなかこれの作業に時間を費やせなかったのが全てなのですが、 Nginx の設定が思っていたよりもずっとシンプルで楽だったので、もっと早くこいつに乗り換える決断をしてしまうべきだったなとは思いました。

メールサーバーの SASL 設定も反省点です。何をやるにしても情報は常にアップデートし続けないとダメですね…。それから DNS に関しても不勉強に過ぎるので、これは書籍をあたるなどしてどうにかします(´･_･`)。
