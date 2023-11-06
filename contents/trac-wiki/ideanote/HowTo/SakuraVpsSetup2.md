[author: murachi]
# さくらの VPS セットアップメモ ～2G プラン乗り換え + Scientific Linux 編～

## はじめに

* [前回の記事はこちら](wiki::HowTo/SakuraVpsSetup)。
* CentOS のバージョンが古くていろいろと支障を来していたので、プランラインナップ変更を期に乗り換えを行うことにした。
* 今回は Scientific Linux を利用することにする。
* 当面は前回との相違点を中心にメモを残す。

## 申し込み

プランラインナップ変更に伴う[乗り換え優遇施策](http:://vps.sakura.ad.jp/news/sakurainfo/newsentry.php?id=628)を利用。

## Scientific Linux セットアップ

VPS コントロールパネルから OS を選択し、基本的なネットワークの設定を行うだけで ok. かんたん。

1. 送られてきた IP アドレスとパスワードで [VPS コントロールパネル](https:://secure.sakura.ad.jp/vpscontrol/)にログインする。
1. 画面左の「OS再インストール」をクリックし、遷移先のページにてさらに「カスタムOSインストールへ」をクリックする。
1. 「OS選択」コンボボックスにて「Scientific Linux 6 x86_64」を選択し、「確認」をクリックする。
1. インストール開始を告げるページに遷移し、インストーラ画面が起動する。
  * インストーラ画面を表示するには Java SE が必要らしい。
1. [Scientific Linux 6｜カスタムOSインストールガイド](http:://support.sakura.ad.jp/manual/vps/mainte/custom_scientificlinux.html)の記述に倣ってセットアップを進める。
  * IPv4 はマニュアル設定、 IPv6 は設定なし。
    * [6rd トライアルを利用した IPv6 運用もできるらしい](http:://research.sakura.ad.jp/2011/01/21/tunnel-6rd-linux1/)。あとで試してみようかなぁ…。
  * STEP 3 がキーボード設定になってるけど、実際にはネットワーク設定よりも前だったような…。
  * 「※さくらのVPS 1G以上のサービスには、ディスクの選択画面が表示されます。」<2G だけど出てこなかったよ?
  * 「OSシャットダウン後、VNC接続も切断されます。インフォメーションを「OK」で閉じてください。」<「OK」なんて無いのでふつーに「×」で画面を閉じませう…。
1. あとはコントロールパネルから VPS を起動してやれば ok.

## ソフトウェアの更新

ViM は入ってた。前回に倣ってとりあえず yum -y update だけしておく。

## ユーザー登録と ssh 接続のためのセキュリティ設定

前回と同じ手順で ok.

## シェルのロケール設定

これも同じく。

## BIND の設定

### chroot 版の BIND をインストールする

* [CentOS 6 - DNSサーバー - chroot環境にする ： Server World](http:://www.server-world.info/query?os=CentOS_6&p=dns&f=4)

BIND を chroot 環境で動かしたい場合、最新の Red Hat クローンでは専用のパッケージである bind-chroot を入れてしまえばいい。知ってさえいれば設定は超楽ちんだ。…上記のサイトを見つけるまで延々と嵌っていた訳だが orz

```
$ sudo yum -y install bind-chroot
```

これで、 chroot 環境である /var/named/chroot 以下に必要なファイルが大体インストールされる。あとはドメインのための設定を加えていけばいいだけ。

ちなみに、インストールされるバージョンは 9.7.3 (2012/5/28 現在)。[幽霊ドメイン名問題](http:://www.geekpage.jp/blog/?id=2012/3/21/1)の件については、このバージョンであれば修正も入るだろう。とりあえず一安心。

### named.conf の設定例等

驚くべき事に、 chroot 環境下における named.conf 等の設定ファイルの類は、 named が実行中の間だけ存在する。仕組みとしては、起動時に実際の /etc/named.conf 等を chroot 環境下にコピーし、それを読み込んでいる模様。面白いことに、 named を稼働させておいて、 chroot 環境下の named.conf 等を編集し、 named を restart させると、編集された内容が反映された上に、ちゃんと /etc/named.conf 等にも編集結果が反映される。つまり、設定の変更はどちらのファイルを用いてもいいようになっているようだ。

named.conf にはデフォルトでいろいろと設定が書かれていて面食らったので、ちょっといろいろ調べてみた。以下、参考になった URL。

* [named.conf の設定](http:://www.nina.jp/server/redhat/bind/named.conf.html)
* [Manpage of NAMED.CONF](http:://linuxreviews.org/man/named.conf/index.html.ja)
* [8.9. その他のステートメント | Turbolinux 11 Server: ユーザーガイド](http:://www.turbolinux.com/products/server/11s/user_guide/x1583.html)
  * 公開 DNS なので listen-on とか allow-query とかは any にする。
  * directory は設定しないとデフォルトではカレントディレクトリ。それだと困る場合は "/var/named" とかに設定する (今回はデフォルトでそうなってる)。
  * dump-file は SIGINT 時のダンプファイル出力先。 statistics-file は SIGILL 時の統計ログ出力先。 memstatistics-file は deallocate-on-exit が yes なら終了時のメモリー統計ログ出力先だが、そも deallocate-on-exit は yes にすると効率悪くなるらしいので no にしておくべき (デフォルトは no)。
* [DNSSEC の設定](http:://xoops.fens.net/modules/wiki/?Linux%2FMemo%2FDNSSEC%20%E3%81%AE%E8%A8%AD%E5%AE%9A)
* [Fujiwara's DNSSEC materials](http:://member.wide.ad.jp/~fujiwara/dnssec/)
  * DNSSEC に対応する方法が書かれてる。
  * とりあえず、今は DNSSEC プロトコルで検証が行われるように設定しておく (デフォルトの named.conf の内容をそのまま使う)。自ゾーンを DNSSEC に対応させて ISC DLV に登録するのはあとで。
  * directory で指定したディレクトリは BIND が書き込み可能である必要がある。 "/var/named/chroot/var/named" を chmod g+w する。

### DNS cache poisoning (Kaminsky attack) への対処

**(2013/11/28 追記)**

名前解決の問い合わせに対して、自己解決しない場合に上位へ問い合わせを再起する設定になっている場合、このサーバー自体を DNS キャッシュサーバーとして利用できるようになる。デフォルトではその設定は有効になっているが、独自ドメイン対応のためだけに DNS を立てている場合には不要な設定である。その上、キャッシュも有効になっていると、 Kaminsky attack を受けてしまう状態となり、 DNS cache poisoning の踏み台にされてしまう。それはまずいので、 recursion は off にしておくべきである。

```
options {

        // ...

        recursion no;
        additional-from-cache no;

        // ...

};
```

### ゾーンの設定

ゾーン設定は 9.7 では named.conf 自体には書かれておらず、 named.rfc1912.zones というファイルを include する形を取っている。ドメインのゾーン設定はこちらに書き加える。書き加える内容自体は以前と同じ。

```
# vim /etc/named.rfc1912.zones

// 以下をファイル末尾に書き加える
zone "harapeko.jp" IN {
        type master;
        file "harapeko.jp.zone";
};

zone "142.128.212.49.in-addr.arpa" IN {
        type master;
        file "harapeko.jp.rev";
};

# vim /var/named/chroot/var/named/harapeko.jp.zone

// 以前の設定内容をそのままコピーし、 A クラスの IP アドレスと SOA クラスの serial だけ新しい値に修正する。
// 但し、ネームサーバーホストは CNAME クラスではなく A クラスを用いるよう修正 (後述)。

# vim /var/named/chroot/var/named/harapeko.jp.rev

// 以前の設定内容をそのままコピーし、 SOA クラスの serial だけ新しい値に修正する。

# chown root:named /var/named/chroot/var/named/harapeko.jp.*
# chmod 640 /var/named/chroot/var/named/harapeko.jp.*
```

なお、正引きのゾーン設定において、ネームサービスを提供するホスト名のアドレスを CNAME (別名) で設定するのは、 9.7 系では illegal となっているらしい。以前まで harapeko.jp.zone ファイルの当該箇所において
```
onaka           IN      A       49.212.128.142
ns              IN      CNAME   onaka
```
としていたのを
```
onaka           IN      A       49.212.128.142
ns              IN      A       49.212.128.142
```
に修正する必要がある。

## パケットフィルタリング設定

iptables はやはり導入済みだった。 設定ファイル /etc/sysconfig/iptables は前回のサーバーのファイルを ViM でコピペした。内容は、前のサーバーを稼働中に以下の行を追加していたのを、そのまま踏襲している。
```
# SMTP Submission port
-A RH-Firewall-1-INPUT -m state --state NEW -m udp -p udp --dport 587 -j ACCEPT
```

### SNMP ブロック対応

**(2013/11/28 追記)**

これまでの内容では SNMP を扱うポート 161, 162 の入力は REJECT していたが、 **出力** までは REJECT していなかったので、以下の行を設定に追加した。

```
-A OUTPUT -m udp -p udp --dport 161 -j REJECT
-A OUTPUT -m udp -p udp --dport 162 -j REJECT
```

## NTP 設定

前回とほぼ同じだが、 restrict 周りはデフォルトで基本 ignore だったので、 NICT へのサーバー参照を追加するのみとした。

## Postfix 設定

Red Hat 6 クローンにおいては、デフォルトのメールデーモンは Sendmail ではなく Postfix となっている。つまり、デフォルトでインストールされている。
従って、行うべきは main.cf の修正とスケルトンへの Maildir/ の追加、そして再起動のみ。

## Web サーバー設定

### Apache のインストールと設定

前回の設定時に入れてないが、 ~~mod_perl を CPAN から自力で入れたりするのに~~ (←結局できなかった) httpd-devel が~~必要になって~~入れていたっぽい (←だから本当に必要だったかも不明)。今回も一応踏襲する。
```
# yum -y install httpd httpd-devel
```

設定は基本的には前のサーバーで記述していたものをそのまま拝借する。バージョンの差異故か、気になる点がいくつかあったのでメモ。

* Timeout は 120 だったのが 60 になっていた。これはそのままにしてみる。
* prefork モジュール下の設定は前のサーバーでの記述を踏襲。詳しくは[この辺](http:://blog.harapeko.jp/2010/11/05/apache-webdav-setting/)を参照。
  * メモリーは以前の 512MB から 2GB に拡張されているのでもうちょっと大きい値を設定してもいいかもしれないけど、現状この設定で特に困ったことにもなっていないので…。
  * モジュールのコメントアウトも同様に対応。 mod_substitute ってのが新たに追加されてたけど、レスポンスヘッダを正規表現で書き換えるためのものらしい。要らん。

rsync によるファイルのコピーでは、 ERK 掲示板のテスト環境で CGI が生成した添付ファイルなどがアクセス権限のエラーでコピーに失敗した。ファイルの所有者が murachi ではなく apache になっていて、かつパーミッションが 600 になっていたため。これは確か、開発中の段階でバグがあったためにそうなっちゃったのを (備忘録も兼ねて) 敢えて放置していたものだが、もはや不要だろうと思うので無視することにする。

### PHP のセットアップ

yum で php-mysql をインストールする際に MySQL が一緒にインストールされなかった。別途インストールする必要有り。こちらもあとで使いそうなのでついでに mysql-devel も入れちゃうことにする。

### Python のセットアップ

Trac の導入に際して setuptools が必要になる。 yum から入れる場合のパッケージ名は python-setuptools である。現時点でのバージョンは 0.6.10 。一応 Trac 0.12 における setuptools の必須バージョンは 0.6 以上となっているので、とりあえずこれで無問題。

それから、前回は mod_python をインストールしたが、現在既に Trac では mod_python を使用していないのでインストールしない。代わりに mod_wsgi をインストールする。

```
# yum -y install python-setuptools mod_wsgi MySQL-python
```

### Perl のセットアップ

httpd.conf をほぼそのまま引き継いだおかげで mod_perl が入っていない状態では Apache が起動すらしない状態だった (\^_\^;)。

とりあえず Perl は (**Perl だけは**) 最新版を入れておきたいので、前回のやり方は踏襲しません。基本的には[こちら](http:://blog.harapeko.jp/2011/01/11/centos5-cpan-update/)を参照。

1. yum から perl-CPAN をインストール。 perl-CPANPLUS は不要。
   ```
   # yum -y install perl-CPAN
   ```
1. cpan コマンドの初期設定。
   ```
   # cpan
   
   (なんか聞かれてデフォルトが [yes] なのでそのままエンター)
   
   (勝手に自動設定が走ってプロンプトへ)
   
   cpan[1]> o conf init
   
   (snip...)
   Would you like me to configure as much as possible automatically? [yes] no
   
   (なんかいろいろ聞かれるけど基本的にはそのままエンター)
   
    <make_install_make_command>
   or some such. Your choice: [/usr/bin/make] sudo /usr/bin/make
   
    <make_install_arg>
   Your choice: [] UNINST=1
   
   (snip...)
   
    <mbuild_install_build_command>
   or some such. Your choice: [./Build] sudo ./Build
   
    <mbuild_install_arg>
   Your choice: [] --uninst 1
   
   (snip...)
   
    <colorize_output>
   Do you want to turn on colored output? [no] yes
   
   (snip...)
   
    <term_is_latin>
   Your terminal expects ISO-8859-1 (yes/no)? [yes] no
   
   (snip...)
   
   (1) Africa
   (2) Asia
   (3) Europe
   (4) North America
   (5) Oceania
   (6) South America
   Select your continent (or several nearby continents) [] 2
   
   (1) China
   (2) India
   (3) Indonesia
   (4) Israel
   (5) Japan
   (6) Kazakhstan
   (7) Pakistan
   (8) Republic of Korea
   (9) Saudi Arabia
   (10) Singapore
   (11) Taiwan
   (12) Thailand
   (13) Turkey
   (14) Viet Nam
   Select your country (or several nearby countries) [] 5
   
   (1) ftp://ftp.dti.ad.jp/pub/lang/CPAN/
   (2) ftp://ftp.jaist.ac.jp/pub/CPAN/
   (3) ftp://ftp.kddilabs.jp/CPAN/
   (4) ftp://ftp.nara.wide.ad.jp/pub/CPAN/
   (5) ftp://ftp.riken.jp/lang/CPAN/
   (6) ftp://ftp.ring.gr.jp/pub/lang/perl/CPAN/
   (7) ftp://ftp.u-aizu.ac.jp/pub/CPAN/
   (8) ftp://ftp.yz.yamagata-u.ac.jp/pub/lang/cpan/
   Select as many URLs as you like (by number),
   put them on one line, separated by blanks, hyphenated ranges allowed
    e.g. '1 4 5' or '7 1-4 8' [] 2 7 4 8 6 5 1 3
   
   cpan[1]> quit
   ```
   以下のサイトが参考になりました。こうすることで CPAN からモジュールインストール時に make install を sudo してくれるので一般ユーザーからでもインストールが楽ちんになるんだそうな。…まぁ色つきプロンプトは微妙だけど。
  * [もう CPANPLUS は使わなくてもいいのかも - daily dayflower](http:://d.hatena.ne.jp/dayflower/20070814/1187084177)
1. CPAN 自体を最新版にアップグレード。途中、YAML.pm がないから YAML が読めないって警告がいくつか出て気持ち悪いけど、その YAML.pm 自体もアップグレード中に勝手に入れてくれるので気にしなくて ok 。
   ```
   # cpan -i Bundle::CPAN
   
   (途中いくつか聞かれるけど基本そのままエンター)
   ```
1. モジュールを一括更新
   ```
   # perl -MCPAN -e 'CPAN::Shell->install(CPAN::Shell->r)'
   
   (途中いくつか聞かれるけど基本そのままエンター)
   ```
1. mod_perl をインストール。こいつは Apache のソースが必要で、 httpd-devel を入れただけではビルドできず、 cpan コマンドからのインストールはできなかった。仕方ないのでとりあえず yum で入れる。
   ```
   # yum -y install mod_perl
   ```
   んでもって設定。設定内容は前のサーバーでのものをそのまま踏襲。
   ```
   # vim /etc/httpd/conf.d/perl.conf
   
   (前のサーバーから必要に応じてコピペ)
   ```
   [あほプログラム](http:://blog.harapeko.jp/2008/12/29/mod-perl-aho/)やその他の mod_perl 向けに作ったプログラムをコピー。
   ```
   # mkdir /etc/httpd/MpTest
   # chown murachi:apache /etc/httpd/MpTest
   # chmod 750 /etc/httpd/MpTest/
   # chmod g+s /etc/httpd/MpTest
   # exit
   logout
   $ rsync -ae ssh onaka.harapeko.jp:/etc/httpd/MpTest/* .
   ```
1. ここで Apache を起動。やっと動かせた。
   ```
   $ sudo /etc/init.d/httpd start
   ```
1. Apache をデフォルトで起動するようにする。
   ```
   $ sudo /sbin/chkconfig --add httpd
   $ sudo /sbin/chkconfig httpd on
   ```

## MySQL の設定

やることは前回と大体同じだが、 PHP のインストール時にクライアントコマンドである mysql パッケージがインストールされなかった点に注意する必要がある。

my.cnf ファイルの設定内容では、前回時と違ってデフォルトファイルにいくつかの差異があったが、以下の理由でそのままにした。

* old_passwords=1 の行が削除されていたが、パスワードは手入力で入力し直すのであれば互換性を気にする必要はないので、そのままにした。
* symbolic-links=0 の行が追加されていた。これはセキュリティ上の理由で推奨される設定らしい。前回もそうコメントに書かれていたのだが、恐らくは互換性の理由でなのか、コメントアウトされていた。

## 開発リポジトリの設定

### Subversion の設定

前回と同じく、無意味に mod_dav_svn などを入れてみる。

### Trac の設定

こちらは前回とは設定方法が全く異なる。基本的には TrackInstall からリンクを辿って得られる情報に則って行った。ちなみにここまでで既に Python と setuptools、そして mod_wsgi は導入済み。

1. Genshi をインストール。
   ```
   $ sudo su -
   # easy_install Genshi
   ```
1. Babel, docutils, Pygments, pytz をインストール。
   ```
   # easy_install Babel
   # easy_install Docutils
   # easy_install Pygments
   # easy_install pytz
   ```
   ここまでやってから pip コマンドの存在に気づいたけど、まぁいいや。 (\^_\^;A
1. [FootNoteMacro プラグイン](http:://trac-hacks.org/wiki/FootNoteMacro)をインストール。こいつは手動でリポジトリから落としてインストールしてやる必要がある。
   ```
   # exit
   $ cd
   $ mkdir -p download/site-packages
   $ cd download/site-packages
   $ svn co http://trac-hacks.org/svn/footnotemacro/
   $ cd footnotemacro/0.11
   $ python setup.py build
   $ sudo python setup.py install
   ```
1. Trac をインストール。
   ```
   $ sudo easy_install Trac
   ```

### データを移行する

リポジトリのデータは単に rsync するだけ。

```
$ sudo mkdir /var/Developer
$ sudo chown murachi:apache /var/Developer
$ sudo chmod 755 /var/Developer
$ cd /var/Developer/
$ rsync -ae ssh onaka.harapeko.jp:/var/Developer/* .
```

Trac の DB データは blog と同様、 mysqldump コマンドを利用する。但し、今回は Trac 関連の DB はすべて同一の DB ユーザーで管理しているので、前回よりコマンド数は少なく済む (grant する回数はむしろ増えたけど)。

* 旧サーバー側での操作:
  ```
  $ cd ~/tempsql
  $ mysqldump -u trac_master -p --databases trac_otoco trac_ideanote trac_erk_board trac_muzplay trac_boeboe trac_paynote >trac_all.sql
  ```
* 新サーバー側での操作:
  ```
  $ cd ~/tempsql/
  $ rsync -ae ssh onaka.harapeko.jp:/home/murachi/tempsql/trac_all.sql .
  $ mysql -u root -p <trac_all.sql
  $ mysql -u root -p
  
  mysql> grant all privileges on trac_otoco.* to trac_master@localhost identified by 'パスワード';
  mysql> grant all privileges on trac_ideanote.* to trac_master@localhost;
  mysql> grant all privileges on trac_erk_board.* to trac_master@localhost;
  mysql> grant all privileges on trac_muzplay.* to trac_master@localhost;
  mysql> grant all privileges on trac_boeboe.* to trac_master@localhost;
  mysql> grant all privileges on trac_paynote.* to trac_master@localhost;
  mysql> quit
  ```

### WSGI の設定

今回は Trac を WSGI で動かすので、 WSGI スクリプトを用意します。手順は以下の通り。

1. 本当は従来も必要だったんですが、 Python eggs を展開するためのキャッシュディレクトリを作成する。
   ```
   $ cd /var/Developer/trac/original
   $ mkdir egg-cache
   $ chmod g+w egg-cache
   ```
1. WSGI スクリプトを記述する。プロジェクトが複数有るので TRAC_ENV ではなく TRAC_ENV_PARENT_DIR を指定する。
   ```
   $ vim trac-original.wsgi
   
   import os
   
   os.environ['TRAC_ENV_PARENT_DIR'] = '/var/Developer/trac/original'
   os.environ['PYTHON_EGG_CACHE'] = '/var/Developer/trac/original/egg-cache'
   
   import trac.web.main
   application = trac.web.main.dispatch_request
   
   $ chmod 640 trac-original.wsgi
   ```
1. httpd.conf にて、かつて mod_python 向けに記述していた Trac の設定を、 WSGI 向けに修正する。
   ```
   $ sudo vim /etc/httpd/conf/httpd.conf
   
   # この辺は思いっきり削除
   #     <IfModule mod_python.c>
   #     <Location /trac/original>
   #         SetHandler mod_python
   #         PythonDebug On
   #             #PythonPath "['/usr/lib/python2.4/site-packages']"
   #         PythonHandler trac.web.modpython_frontend
   #         PythonOption TracEnvParentDir /var/Developer/trac/original
   #         PythonOption TracUriRoot /trac/original
   #     </Location>
   
   # 代わりに以下を挿入
       <IfModule mod_wsgi.c>
       WSGIScriptAlias /trac/original /var/Developer/trac/original/trac-original.wsgi
   
       <Directory /var/Developer/trac/original>
           WSGIApplicationGroup %{GLOBAL}
           Order deny,allow
           Allow from all
       </Directory>
   ```
1. Apache を再起動。

## SSL 証明書を更新

前回はここでドメイン参照先の変更を行ったが、今回はそこは最後に回すことにする。

基本的には前回と同様。以下、気になる点、および差異をメモ。

* [RapidSSL](http:://www.rapidssl.com/) の Reissue SSL 申し込みフォームにおいて、 Firefox では CAPTCHA が表示されなかった。仕方なしに IE でアクセスしたのだが、これは SSL 認証局のあり方としてどうなんだろう…。
* 秘密鍵生成時に指定するデータ長は、 RapidSSL でも現在 2048 bits 以上が必須となっている点に注意。前回は 1024 でやっていたが、今回は 2048 で鍵を生成した。
  ```
  $ openssl genrsa -out developer.harapeko.jp.key 2048
  $ openssl req -new -key developer.harapeko.jp.key -out developer.harapeko.jp.csr
  ```
* 承認を済ませると、SSL 証明書がメールで送られてくるが、 2つ書かれているように見える証明書らしきものの後者は中間証明書である。[RapidSSL 公式の説明](https:://knowledge.rapidssl.com/support/ssl-certificate-support/index?page=content&id=SO6252&actp=search&viewlocale=en_US&searchid=1291957903082)に基づき、こちらは intermediate.crt ファイルに保存した上で、 /etc/httpd/conf.d/ssl.conf ファイル中の SSLCertificateChainFile ディレクティブ (SSLCACertificateFile ディレクティブ**ではなく**) に設定する。
  ```
  SSLCertificateFile /etc/httpd/conf/ssl/developer.harapeko.jp.crt
  SSLCertificateKeyFile /etc/httpd/conf/ssl/developer.harapeko.jp.key
  SSLCertificateChainFile /etc/httpd/conf/ssl/intermediate.crt
  ```
* バージョンの違い故か、もしくは Scientific Linux を採用したが故か、 https://developer.harapeko.jp/ にアクセスした際に表示される 403 ページがあんまり 403 ページらしく見えないが、ヘッダ情報を確認したところちゃんと 403 ページだった。ちなみに前回は何らかの内容が表示されるようなことを書いているが、 https プロトコル用の DocumentRoot は http プロトコル用のそれとは別にしてあるので、ファイルが存在せず、 Forbidden となるのが正しい。

## メールサーバーの認証周り設定

### ユーザーアカウントの引き継ぎ

メールのユーザー情報は MySQL に保存してあるので、まずはそいつを移転する。

* 旧サーバー側:
  ```
  $ cd ~/tempsql
  $ mysqldump -u mailusers -p mailusers >mailusers.sql
  ```
* 新サーバー側:
  ```
  $ cd ~/tempsql
  $ rsync -ae ssh onaka.harapeko.jp:/home/murachi/tempsql/mailusers.sql .
  $ mysql -u root -p
  
  mysql> create database mailusers;
  mysql> grant all privileges on mailusers.* to mailusers@localhost identified by 'パスワード';
  mysql> quit
  
  $ mysql -u mailusers -p mailusers <mailusers.sql
  ```

必要に応じてユーザーアカウントも作ってしまおう。

```
$ sudo su -
# useradd -m -s /sbin/nologin toshiyuki.murayama
# passwd toshiyuki.murayama
# useradd -m -s /sbin/nologin sysadmin
# passwd sysadmin
(...snip)
```

転送設定をしてたアカウントはそれも引き継がないとね。

```
# su -s /bin/bash toshiyuki.murayama
$ cd
$ vim .forward

(旧サーバーからコピペ)

$ exit
(...他のユーザーも)
```

そして mailusers DB に保存されている uid と gid も新しいものに修正しますよと。
```
$ grep toshiyuki.murayama /etc/passwd
(uid と gid をチェック...)

$ mysql -u mailusers -p mailusers

mysql> update users set uid=XXX, gid=XXX where userid='toshiyuki.murayama';
mysql> quit

(...他のユーザーも)
```

この辺の作業は流石にターミナル 2つ開いてやった方がいいかもね。あんまりユーザー数多いならスクリプト書いた方がいいかも…。

### POP3S の設定

dovecot と dovecot-mysql をインストール。

```
$ sudo su -
# yum -y install dovecot dovecot-mysql
```

dovecot のメジャーバージョンが変わっているので、設定ファイルの書き方ががらっと変わってる。設定項目毎にファイルが切り分けられ、ようするに httpd.conf のなれの果てみたいになっている。大本の設定ファイルは以下だが、

```
# vim /etc/dovecot/dovecot.conf
```

このファイルの中で設定すべき項目は以下ぐらい。

```
# pop3s を指定することはできない。この指定で pop3 と pop3s の両方に対応してしまう。
protocols = pop3
# IPv4, IPv6 双方のすべてのネットワークからのアクセスを受け付ける。ていうか省略可能?
listen = *, ::
```

POP3s プロトコル以外は受け付けたくないので、 conf.d/10-master.conf を編集。

```
# vim /etc/dovecot/conf.d/10-master.conf

# IMAP は使わないのでコメントアウト
#service imap-login {
#  inet_listener imap {
#    #port = 143
#  }
#  inet_listener imaps {
#    #port = 993
#    #ssl = yes
#  }
#
# (snip...)
#}

service pop3-login {
  inet_listener pop3 {
    # POP3 は使わないので、 port = 0 を設定する
    port = 0
  }
  inet_listener pop3s {
    # こっちはどうやらデフォルト設定なので省略しても良いのかも。
    port = 995
    ssl = yes
  }
}
```

conf.d/10-director.conf も編集。

```
# vim /etc/dovecot/conf.d/10-director.conf

# IMAP は使わないのでコメントアウト
#service imap-login {
#  #executable = imap-login director
#}
```

Maildir 形式の設定は conf.d/10-mail.conf にて。

```
# vim /etc/dovecot/conf.d/10-mail.conf

mail_location = maildir:~/Maildir
```

認証周りの設定は conf.d/10-auth.conf にて。

```
# vim /etc/dovecot/conf.d/10-auth.conf

# 平文認証。通信路は SSL で保護されているので無問題。むしろ生のパスワードが渡されないと SQL で認証できない。
auth_mechanisms = plain

# 認証には SQL を使用するので、挿入するファイルを変更
#!include auth-system.conf.ext   # ←コメントアウト
!include auth-sql.conf.ext       # ←こっちのコメントアウトを外す
```

それから SSL 関連の設定は conf.d/10-ssl.conf にて行う。

```
# vim /etc/dovecot/conf.d/10-ssl.conf

ssl = yes
ssl_cert = </etc/httpd/conf/ssl/developer.harapeko.jp.crt
ssl_key = </etc/httpd/conf/ssl/developer.harapeko.jp.key
ssl_ca = </etc/httpd/conf/ssl/intermediate.crt
```

ここまでの設定に関しては、以下のサイトが参考になった。

* [dovecot-2.0.x ハマリ道 - defiantの日記](http:://d.hatena.ne.jp/defiant/20110421/1303362615)

そして、 SQL 認証に関する設定。これは conf.d/10-auth.conf が include しているファイル conf.d/auth-sql.conf.ext の中で、更に dovecot-sql.conf.ext ファイルを、変数が設定されたファイルとして参照しているので、そいつを作成してその中に記述する。

```
# vim /etc/dovecot/dovecot-sql.conf.ext

driver = mysql
connect = host=localhost dbname=mailusers user=mailusers password=パスワード
password_query = select userid as username, domain, plain_passwd as password from users where userid = '%n' and domain = '%d'
user_query = select home, uid, gid from users where userid = '%n' and domain = '%d'
```

以上で設定は完了。起動し、デフォルトで起動するようにする。

```
# /etc/init.d/dovecot start
# chkconfig --add dovecot
# chkconfig dovecot on
```

### Submission over TLS の設定

cyrus-SASL と、平文認証用のプラグインはすでにインストールされていた。 SQL 用の auxprop プラグインと MD5 用のプラグインをインストールする。

```
# yum -y install cyrus-sasl-sql cyrus-sasl-md5
```

設定ファイルは前回とは異なり、 /etc/sasl2/smtpd.conf 以外の 2つのファイルは削除されていた。安心して左記のファイルのみを編集する。

```
# vim /etc/sasl2/smtpd.conf

pwcheck_method: auxprop
auxprop_plugin: sql
mech_list: cram-md5 digest-md5 plain login
sql_engine: mysql
sql_hostnames: localhost
sql_user: mailusers
sql_passwd: パスワード
sql_database: mailusers
sql_select: select plain_passwd as password from users where userid = '%u'
```

こちらも TLS を介するので mech_list は plain のみでも良かったのだが、クライアント側をいちいち変更したくなかったので (それまでは cram-md5 で運用していた)、前回と同じ設定にしておいた。

余談: dovecot とちがって、パスワードの比較は何が何でも SQL で行うのではなく、 DB 上のパスワードをメカニズムに応じて auxprop プラグインに変換させたものを cyrus-SASL の中で比較するらしい (だから DB 上のパスワードは平文じゃないと駄目なんだな… ;_;)。 いっそのこと平文のみ受け付けて、受け取ったパスワードの方をカスタマイズ可能なハッシュ関数にかけてから比較する auxprop プラグインでも作った方が安全な気もするが…

さらに main.cf を編集。

```
# vim /etc/postfix/main.cf

# 以下の行を追加
smtpd_sasl_auth_enable = yes
smtpd_sasl_security_options = noanonymous   # noplaintext は外した。
smtpd_sasl_local_domain = $myhostname
smtpd_tls_cert_file = /etc/httpd/conf/ssl/developer.harapeko.jp.crt
smtpd_tls_key_file = /etc/httpd/conf/ssl/developer.harapeko.jp.key
smtpd_tls_CAfile = /etc/httpd/conf/ssl/intermediate.crt     # ルート CA 証明書の指定を追加
smtpd_use_tls = yes
smtpd_tls_auth_only = yes
smtpd_recipient_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_unauth_destination
```

Submission ポートを使用するので、 master.cf も編集。

```
# vim /etc/postfix/master.cf

# 以下の行のコメントアウトを外す
submission inet n       -       n       -       -       smtpd
  -o smtpd_tls_security_level=encrypt
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_client_restrictions=permit_sasl_authenticated,reject
  -o milter_macro_daemon_name=ORIGINATING
```

前回は 1行目と 3行目しかコメントアウトを外さなかったんだけど、全部外してる例も見かけたので全部外してみた。 SASL/TLS を介することが前提になってるんですね。多分こうしておいた方が安全なんだと思う (いいかげん)。

そして postfix を再起動…。

```
# /etc/init.d/postfix restart
```

## Web フォルダの移行

前回はこれで終わりだったんだけど (ドメイン移行以外は)、その後、[この辺](http:://blog.harapeko.jp/2010/11/05/apache-webdav-setting/)で Web フォルダなるものを作っていたので、そいつの設定とファイルのコピーも行う。

と思ったんだけど、ファイルは Apache の設定のときにコピーした vhosts 環境の中に含まれてたので、コピーは終わってた。なので設定だけですね。[この辺](http:://blog.harapeko.jp/2010/11/05/apache-webdav-setting/)参照、ってことで。

## ドメイン参照先変更

アプリケーションの移行および設定は一通り完了したので、最後にドメインの移行を済ませる。

やることは前回と同じ。

## 反省

移転先を決めてから旧サーバー停止まで 1ヶ月半は猶予があったのに、結局終わりの 2, 3日でかけこみになってしまった…。

BIND の設定で A レコードに間違えて旧サーバーの IP アドレスを指定してしまったために、旧サーバーで事前にわざわざ TTL を 600 程度の小さな値にしておいた意味が全くないままほぼ丸一日分ぐらい名前解決に失敗する状態を作ってしまいました。 5/31～6/1 の間にメールをお送りいただいた方はエラーが出て送信に失敗されていたかも知れません。本当に申し訳ない限りです…

そしてやっぱりメールの設定でいろいろとしくじっていたという… master.cf の設定を忘れるとか dovecot-sql.conf.ext ファイルを置く場所を間違えるとか、必要なプラグインがインストールされていなかったのになかなか気づかなかったとか…。

やっぱりサーバー移転はちゃんと時間に余裕を持って取り組まないと駄目ですね。 orz

あと DNS は設定をしくじってもリカバリがすぐできるように、新しい方の設定においても最初は TTL を短めな値にしておいて、ちゃんと移行できたことを確認してから TTL を再設定、とかした方が安全なのだと思われます (と、Twitter でもとある方に指摘されました…)。
