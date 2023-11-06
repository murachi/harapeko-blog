[author: murachi]
# さくらの VPS セットアップメモ

## 申し込み

* [さくらの VPS](http:://vps.sakura.ad.jp/)

上記サイトより手順に沿って申し込む。ここで取得するのは以下の 2つ。

* さくらインターネット会員 ID
  * 「お申込受付完了のお知らせ」と題されたメール、および登録時の画面にて ID が伝えられる。パスワードは登録時に指定したものをそのまま利用する。
  * [会員メニュー](https:://secure.sakura.ad.jp/menu/)にて、契約情報の確認および変更を行うのに使用する。**ID は絶対に紛失しないよう保管すること**。
* VPS の IP アドレス、および root 初期パスワード
  * 「仮登録完了のお知らせ」と題されたメールにて、連絡される。**このメールは紛失・流出しないよう大切に保管すること**。

## VPS の起動

VPS は起動していない状態で提供される。そのままの状態で連絡された IP に ssh でつなぎに行ってもうんともすんとも言わない。

VPS の起動方法は以下の通り。

1. [VPS コントロールパネル](https:://secure.sakura.ad.jp/vpscontrol/)にアクセスする。
1. 取得した VPS の IP アドレスと root パスワードを入力する。
1. VPS ホーム画面にて、「仮想サーバ操作」の「起動」ボタンをクリックする。

## ソフトウェアの更新と ViM のインストール

ViM が入っていなかったのでインストールする。ついでにソフトウェアを最新版にアップデートしておく。

1. root でログインする。
1. ソフトウェアを最新版にアップデートする。
   ```
   # yum check-update
   # yum update
   ```
1. ViM をインストール
   ```
   # yum install vim-enhanced
   ```

## ユーザー登録と ssh 接続のためのセキュリティ設定

初期状態では、 ssh 接続にて root でパスワード認証によりログインする、という決して安全とは言えない状態になっている。これを、 root 以外のユーザーのみが公開鍵認証によりログインできるように設定する必要がある。

この設定方法については、よくまとまった記事があったので、こちらを参照のこと。

* [myfinder's blog: さくらのVPSを借りたら真っ先にやるべきssh設定](http:://blog.myfinder.jp/2010/09/vpsssh.html)

以下、簡単にメモする。

1. root でログインする。
1. ユーザーを作成する。
   ```
   # useradd -m -G wheel murachi
   # passwd murachi
   ```
1. sshd の設定を変更する。
   ```
   # vim /etc/ssh/sshd_config
   
   PermitRootLogin no          # root での接続を無効
   PasswordAuthentication no   # パスワード認証を無効
   UsePAM no                   # PAM を無効
   ```
1. 公開鍵認証用の鍵を作る。これはローカル PC 上で行う。
  * **公開鍵のパスワードと実際のユーザーのパスワードは同じにしないこと**。これ、同じにしたらあんまり意味がなくなってしまう。
  * Windows で Putty の場合は puttygen.exe ってので作れる。公開鍵と秘密鍵をファイルに出力し (秘密鍵は拡張子が .puk になる)、Putty からの接続時に秘密鍵ファイルを「接続/SSH/認証」画面の「認証のためのプライベートキーファイル」欄に指定する。
  * それ以外 (Windows で Cygwin 利用、または Mac OS X や Linux など) の場合、 ssh-keygen コマンドを使用する。 -t は rsa か dsa のどちらかにすること (rsa1 は弱いので不可)。
    ```
    $ ssh-keygen -t dsa
    $ ls -a .ssh
    ```
    これにより、 .ssh ディレクトリ下に秘密鍵ファイル (id_rsa または id_dsa) と公開鍵ファイル (id_rsa.pub または id_dsa.pub ファイル) が生成される。
1. 公開鍵認証ファイルをサーバー上に書き込む。
   ```
   # su - murachi
   $ mkdir .ssh
   $ chmod 700 .ssh
   $ vim .ssh/authorized_keys
   
   (公開鍵ファイルの内容をコピペする)
   
   $ chmod 600 .ssh/authorized_keys
   ```
1. sshd を再起動。
   ```
   $ exit  # root に戻る
   # /etc/init.d/sshd restart
   ```
1. 一旦ログアウトし、作ったユーザーでログインし直す。
   ```
   # exit
   logout
   $ ssh -l murachi xxx.xxx.xxx.xx
   ```
  * Windows で Putty の場合は、ホスト名を hoge@xxx.xxx.xxx.xx とかにして接続する。秘密鍵ファイルの指定を忘れずに!
1. sudo を設定する。
   ```
   $ su -
   # visudo
   
   %wheel  ALL=(ALL)       ALL     # コメントアウトされていたはずなので、コメントを外す
   
   # exit
   $ sudo su -     # sudo できるか確認
   ```
1. root パスワード無効化、 sudo ログの設定は、お好みで。

## シェルのロケール設定

bash を UTF-8 による日本語表示に対応させる。

```
$ vim ~/.bash_profile

# 以下の行を追加
export LANG=ja_JP.UTF-8

$ sudo su -
# vim .bash_profile

# root ユーザーでも日本語表示したい場合はこちらも同様に設定する。
export LANG=ja_JP.UTF-8
```

お好みで。

## BIND の設定

参考:

* [＠IT：BIND 9のセキュリティ対策](http:://www.atmarkit.co.jp/flinux/rensai/bind909/bind909a.html)

1. なんと BIND がインストールされていないので、インストールする。
   ```
   $ sudo yum install bind
   ```
1. chroot のためのディレクトリ構成を /var/named/chroot 以下に構築する。
   ```
   $ sudo su -
   # mkdir -p /var/named/chroot/
   # cd /var/named/chroot
   # chown root:named .
   # chmod 760 .
   # chmod g+s .
   # mkdir -p dev etc var/log var/named var/run/named
   # chmod o-rx * var/* var/run/named
   # chmod g+w var/log var/run/named
   ```
1. 設定ファイルをコピーし、修正する。また、ゾーンファイルを作成する。
   ```
   # cd etc
   # cp /etc/named.conf .
   # cp /etc/localtime .
   # vim named.conf
   
   (設定内容は省略)
   ```
  * ゾーンファイルの作成については、[会社サーバーを最初に立てたときのブログ記事](http:://blog.harapeko.jp/2008/05/07/buildup-server/)を参照。
  * named.root ファイルが必要な場合は、[このファイル](ftp:://ftp.rs.internic.net/domain/named.root)を wget してしまえばいいと思う。
    ```
    # cd /var/named/chroot/var/named
    # wget ftp://ftp.rs.internic.net/domain/named.root
    # chmod 640 named.root
    ```
1. デバイスファイルを作成する。
   ```
   # cd /var/named/chroot/dev
   # mknod null c 1 3
   # mknod random c 1 8
   # chmod 666 *
   ```
1. CentOS なので、 chroot が有効になるよう、 /etc/sysconfig/named ファイルを編集する。
   ```
   # vim /etc/sysconfig/named
   
   ROOTDIR="/var/named/chroot"
   ```
1. BIND を起動。
   ```
   # /etc/init.d/named start
   ```

上手く起動すればめでたしめでたし。でもドメイン取得していないうちは起動しても意味ないので、とりあえず止めておきましょう。今回はサーバー移行で移行前のサーバーをまだ使っていたいので、とりあえず止めておきました。

```
# /etc/init.d/named stop
```

## パケットフィルタリング設定

iptables は入っていて、動いていたが、肝心要の設定ファイル /etc/sysconfig/iptables が存在しなかった。移転前のサーバーからこのファイルをコピーし、それを適宜修正してから稼働させた。

1. /etc/sysconfig/iptables-config の内容を確認。
   ```
   $ sudo su -
   # vim /etc/sysconfig/iptables-config
   
   # 以下の行が移転前サーバーと異なっていたが、気にせずそのままにした。
   IPTABLES_MODULES=""
   ```
  * ちなみに、移転前サーバーではこの行は以下のようになっていた。
    ```
    IPTABLES_MODULES="ip_conntrack_netbios_ns"
    ```
  * これは iptables 用に読み込むモジュールの指定で、指定できるモジュールは CentOS の場合、以下のようにすることで調べることができる。
    ```
    # ls /lib/modules/`uname -r`/kernel/net/ipv4/netfilter/
    ```
  * ip_conntrack モジュール (接続情報トラッキング) や ip_conntrack_ftp モジュール (ip_nat_ftp と組み合わせて FTP の PASV モードに対応?) なんかはネットで情報が出てくるが、 ip_conntrack_netbios_ns については、読み込みに失敗したから外しました、といったような情報しか出てこなかった。 Windows マシンからの通信絡みなんだろうとは思うんだが…詳細きぼん… さくらの VPN にもこのモジュールは存在するので指定してもエラーにはならないが、何に使うものかも分からないで指定するのも無意味なので外しておくことにした。
1. /etc/sysconfig/iptables を移転前サーバーからコピーする。
  * まず、移転前サーバーにて。
    ```
    $ sudo cp /etc/sysconfig/iptables .
    $ sudo chown murachi:wheel iptables
    ```
  * 次に、移転先サーバーにて。
    ```
    $ rsync -ae ssh onaka.harapeko.jp:/home/murachi/iptables .
    $ sudo su -
    # chown root:root /home/murachi/iptables
    # mv /home/murachi/iptables /etc/sysconfig/
    ```
1. /etc/sysconfig/iptables の内容を適宜修正。
   ```
   # vim /etc/sysconfig/iptables
   
   # Firewall configuration written by system-config-securitylevel
   # Manual customization of this file is not recommended.
   *filter
   :INPUT ACCEPT [0:0]
   :FORWARD ACCEPT [0:0]
   :OUTPUT ACCEPT [0:0]
   :RH-Firewall-1-INPUT - [0:0]
   -A INPUT -j RH-Firewall-1-INPUT
   -A FORWARD -j RH-Firewall-1-INPUT
   -A RH-Firewall-1-INPUT -i lo -j ACCEPT
   -A RH-Firewall-1-INPUT -p icmp --icmp-type any -j ACCEPT
   -A RH-Firewall-1-INPUT -p 50 -j ACCEPT
   -A RH-Firewall-1-INPUT -p 51 -j ACCEPT
   -A RH-Firewall-1-INPUT -p udp --dport 5353 -d 224.0.0.251 -j ACCEPT
   #-A RH-Firewall-1-INPUT -p udp -m udp --dport 631 -j ACCEPT
   #-A RH-Firewall-1-INPUT -p tcp -m tcp --dport 631 -j ACCEPT
   -A RH-Firewall-1-INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
   # DNS
   -A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 53 -j ACCEPT
   -A RH-Firewall-1-INPUT -m state --state NEW -m udp -p udp --dport 53 -j ACCEPT
   # Service applications with TCP: ssh, SMTP, SMTP Submission port, POP3S, HTTP(S)
   -A RH-Firewall-1-INPUT -m state --state NEW -m multiport -p tcp --dports 22,25,587,995,80,443 -j ACCEPT
   # NTP
   # -A RH-Firewall-1-INPUT -m state --state NEW -m udp -p udp --dport 123 -j ACCEPT
   # other (reject)
   -A RH-Firewall-1-INPUT -j REJECT --reject-with icmp-host-prohibited
   COMMIT
   ```
  * ポート 631 (=Internet Printing Protocol) なんて使わないのでコメントアウト。
  * TCP でしか使わないようなサービス (ssh, SMTP (サブミッションポート含めて), POP3S, HTTP(S)) を multiport モジュールを使って 1行にまとめたらファイルがやたらとコンパクトになって良い感じ。
  * NTP も当面は公開しないのでコメントアウト。
  * セキュリティリスクにしかならない FTP と POP3 は未来永劫使う予定無いので思い切って削除。
1. iptables を再起動する。
   ```
   # /etc/init.d/iptables restart
   ```

## NTP 設定

NTP を使って時刻合わせをする。 NTP 自体はすでにインストール済みで、しかも NTPD が動いていたが、設定を修正した。

1. /etc/ntp.conf を修正する。
   ```
   $ sudo vim /etc/ntp.conf
   
   # 公開しないので…
   #restrict default kod nomodify notrap nopeer noquery
   #restrict -6 default kod nomodify notrap nopeer noquery
   restrict default ignore
   
   # デフォルトでさくらの NTP サーバーが設定されていた。 NICT も追加しておく。
   server ntp1.sakura.ad.jp
   server ntp.nict.jp iburst
   server ntp.nict.jp iburst
   server ntp.nict.jp iburst
   ```
1. NTPD を再起動。
   ```
   $ sudo /etc/init.d/ntpd restart
   ```

## Postfix 設定

メールサーバーは Postfix を使用する。詳細な設定は DNS を有効にし、 SSL を組み込んでからさらに行う。ここではとりあえずインストールと

1. デフォルトで入っている Sendmail を停止し、二度と起動しないようにする。
   ```
   $ sudo su -
   # /etc/init.d/sendmail stop
   # chkconfig --del sendmail
   ```
1. Postfix をインストール。
   ```
   # yum install postfix
   ```
1. /etc/postfix/main.cf を書き換える。
   ```
   # vim /etc/postfix/main.cf
   
   # 以下は既存の項目を書き換え。
   myhostname = developer.harapeko.jp
   myorigin = $mydomain
   inet_interfaces = all
   mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
   mynetworks_style = host
   home_mailbox = Maildir/
   
   # (こっちはコメントアウト。 X Window とか使わないし)
   #debugger_command =
   #        PATH=/bin:/usr/bin:/usr/local/bin:/usr/X11R6/bin
   #        xxgdb $daemon_directory/$process_name $process_id & sleep 5
   
   # (こっちのコメントアウトを外す。)
   debugger_command =
           PATH=/bin:/usr/bin:/usr/local/bin; export PATH; (echo cont;
           echo where) | gdb $daemon_directory/$process_name $process_id 2>&1
           >$config_directory/$process_name.$process_id.log & sleep 5
   
   # メールサイズを 10MB に制限
   message_size_limit = 10485760
   ```
1. Maildir/ 形式を利用するので、ホームディレクトリスケルトンに Maildir ディレクトリを作成する。
   ```
   # mkdir -p /etc/skel/Maildir/{new,cur,tmp}
   # chmod -R 700 /etc/skel/Maildir/
   ```
1. システムで使用するサーバー機能を Postfix に切り替える。
   ```
   # alternatives --config mta
   
   There are 2 programs which provide 'mta'.
   
     Selection    Command
   -----------------------------------------------
   *+ 1           /usr/sbin/sendmail.sendmail
      2           /usr/sbin/sendmail.postfix
   
   Enter to keep the current selection[+], or type selection number: 2
   ```
1. Postfix を起動し、問題なければサーバー起動時に自動起動するよう設定する。
   ```
   # /etc/init.d/postfix check
   # /etc/init.d/postfix start
   # chkconfig --add postfix
   # chkconfig postfix on
   ```

## Web サーバー設定

### Apache のインストールと設定

Apache をインストールし、設定する。

1. apache をインストールする。
   ```
   $ sudo su -
   # yum install httpd
   ```
1. /etc/httpd/conf/httpd.conf を編集。
   ```
   # vim /etc/httpd/conf/httpd.conf
   
   # サーバー名を明示
   ServerName www.harapeko.jp:80
   
   # ドキュメントルートを変更。特に意味はないが…
   #DocumentRoot "/var/www/html"
   DocumentRoot "/var/www/htdocs"
   
   # ここも忘れずに修正…
   #<Directory "/var/www/html">
   <Directory "/var/www/htdocs">
   
   # インデックス表示は行わない
   #    Options Indexes FollowSymLinks
       Options FollowSymLinks
   
   # .httpaccess を許容する
   #    AllowOverride None
       AllowOverride All
   
   # 日本語を最優先する
   #LanguagePriority en ca cs da de el eo es et fr he hr it ja ko ltz nl nn no pl pt pt-BR ru sv zh-CN zh-TW
   LanguagePriority ja en ca cs da de el eo es et fr he hr it ko ltz nl nn no pl pt pt-BR ru sv zh-CN zh-TW
   
   # デフォルトの文字セットは指定しない
   #AddDefaultCharset UTF-8
   AddDefaultCharset off
   
   # バーチャルホストを使用する
   NameVirtualHost *:80
   
   # 会社トップページ http://www.harapeko.jp/
   <VirtualHost *:80>
       ServerAdmin root@harapeko.jp
       DocumentRoot /var/www/htdocs
       ServerName www.harapeko.jp
       ErrorLog logs/www-error_log
       CustomLog logs/www-access_log common
   </VirtualHost>
   # 会社ブログ http://blog.harapeko.jp/
   <VirtualHost *:80>
       ServerAdmin root@harapeko.jp
       DocumentRoot /var/www/vhosts/blog/htdocs
       ServerName blog.harapeko.jp
       ErrorLog logs/blog-error_log
       CustomLog logs/blog-access_log common
   </VirtualHost>
   # 村山個人ブログ (http://harapeko.asablo.jp/blog/ ) 用仮設リソース置き場 http://daiyokujo.harapeko.jp/
   <VirtualHost *:80>
       ServerAdmin root@harapeko.jp
       DocumentRoot /var/www/vhosts/daiyokujo/htdocs
       ServerName daiyokujo.harapeko.jp
       ErrorLog logs/daiyokujo-error_log
       CustomLog logs/daiyokujo-access_log common
   </VirtualHost>
   # 開発用リソース (Trac 等) http://developer.harapeko.jp/
   <VirtualHost *:80>
       ServerAdmin root@harapeko.jp
       DocumentRoot /var/www/vhosts/developer/htdocs
       ServerName developer.harapeko.jp
       ErrorLog logs/developer-error_log
       CustomLog logs/developer-access_log common
   
       # Trac を mod_python で動かすための設定
       <IfModule mod_python.c>
       <Location /trac/original>
           SetHandler mod_python
           PythonDebug On
           PythonHandler trac.web.modpython_frontend
           PythonOption TracEnvParentDir /var/Developer/trac/original
           PythonOption TracUriRoot /trac/original
       </Location>
   
       <Location /trac/original/otoco/login>
           AuthType Digest
           AuthName realm
           AuthUserFile "/var/Developer/trac/original/otoco/.htdigest"
           Require valid-user
       </Location>
   
       <Location /trac/original/ideanote/login>
           AuthType Digest
           AuthName realm
           AuthUserFile "/var/Developer/trac/original/ideanote/.htdigest"
           Require valid-user
       </Location>
       </IfModule>
   </VirtualHost>
   
   # バーチャルホストに使うディレクトリの設定
   <Directory /var/www/vhosts/*/htdocs>
       Options FollowSymLinks
       AllowOverride All
       Order allow,deny
       Allow from all
   </Directory>
   ```
1. 移転前サーバーの htdocs 等をコピーする
   ```
   # exit  # 一旦 root を抜ける
   $ cd /var/www
   $ sudo mkdir htdocs vhosts
   $ sudo chown murachi:apache htdocs vhosts
   $ sudo chmod g+s htdocs vhosts
   $ rsync -ae ssh onaka.harapeko.jp:/var/www/htdocs .
   $ rsync -ae ssh onaka.harapeko.jp:/var/www/vhosts .
   ```
1. とりあえず起動してみる
   ```
   $ sudo /etc/init.d/httpd configtest
   $ sudo /etc/init.d/httpd start
   ```
  * クライアント PC が Windows の場合: %windir%\system32\drivers\etc\hosts に www.harapeko.jp のアドレスを追加してから http://www.harapeko.jp/index.php にアクセスすると、 PHP スクリプトのソースが表示されるのが確認できた。
1. デフォルトで起動するように設定する。
   ```
   $ sudo /sbin/chkconfig --add httpd
   $ sudo /sbin/chkconfig httpd on
   ```

### Web アプリに使用する各種言語のセットアップ

必要なのは以下の通り。

* PHP + mod_php ... 会社サイト (自作) と会社ブログ (WordPress) にて使用。
* Python + mod_python ... タスク管理 (Trac) にて使用。
* Perl + mod_perl ... ~~個人的な趣味~~将来的に使用する予定…。

まずは PHP をインストールする。

1. PHP をインストールする。
   ```
   $ sudo su -
   # yum install php           # cli とか common とかも一緒に入る
   # yum install php-mysql     # MySQL とか pdo 、あと何故か perl-DBI も一緒に入る
   # yum install php-mbstring  # マルチバイト (日本語) 処理に必要っぽい
   ```
  * 移転前サーバーではこれ以外に php-pgsql (PostgleSQL 関数) と php-ldap (LDAP 関数) が入っていたが、多分どっちも使わないと思うので入れなかった。
1. PHP の設定。
   ```
   # vim /etc/php.ini
   
   # (とはいえ、変更すべき箇所は特になし。 magic_quotes_* シリーズが Off になっているのを確認するぐらい?)
   ```
1. mod_php の設定。
   ```
   # vim /etc/httpd/conf.d/php.conf
   
   # (こちらも変更すべき箇所は特になし。)
   ```
1. Apache を再起動。
   ```
   # /etc/init.d/httpd restart
   ```
  * トップページが (PHP スクリプトのソースではなく) ちゃんと表示されることを確認。
    移転先サーバーによるものであることが判りやすいよう、サイトの内容を若干変更した (住所が引っ越し前のままだったのを修正する等)。

次に Python をインストール。

1. Python 自体はすでに入っているので、 mod_python と MySQL-python モジュールをインストール。
   ```
   # yum install mod_python MySQL-python
   ```
1. mod_python の設定。
   ```
   # vim /etc/httpd/conf.d/python.conf
   
   # (変更すべき箇所は特になし。)
   ```
1. Apache を再起動。
   ```
   # /etc/init.d/httpd restart
   ```
1. テストプログラムを動かしてみる。面倒くさいので .htaccess を使う方法で。
  1. /var/www/htdocs の下にディレクトリ py-test を作り、その下に .htaccess と Python スクリプトを作成する。
     ```
     # exit
     $ cd /var/www/htdocs
     $ mkdir py-test
     $ vim py-test/.htaccess
     
     AddHandler mod_python .py
     PythonHandler mptest        # スクリプトファイルを mptest.py にする場合
     PythonDebug On
     
     $ vim py-test/mptest.py
     
     from mod_python import apache
     
     def handler(req):
         req.write("Hello, World!")
         return apache.OK
     ```
  1. ブラウザから http://www.harapeko.jp/py-test/mptest.py にアクセスし、画面に "Hello, World\" とだけ表示されるのを確認。
  1. テストプログラムはもはや不要なので、消しておく。
     ```
     $ rm -rf py-test
     ```

最後に Perl をインストール。

1. Perl はフツーに 5.8 が入っていた。 5.10 にアップグレードしたい気もするがとりあえずあとにして、 mod_perl と DBD-MySQL をインストールする。
   ```
   $ sudo yum install mod_perl perl-DBD-MySQL
   ```
1. mod_perl の設定。
   ```
   $ sudo vim /etc/httpd/conf.d/perl.conf
   
   # テスト用にレジストリ形式で CGI エミュレーションするディレクトリを設定。通常は不要。お好みで。
   Alias /perl /var/www/perl
   <Directory /var/www/perl>
       SetHandler perl-script
       PerlResponseHandler ModPerl::Registry
       PerlOptions +ParseHeaders
       Options +ExecCGI
   </Directory>
   
   # ああ、こんなのあったなぁ…
   <VirtualHost *:80>
     ServerName test.harapeko.jp
     DocumentRoot /var/www/vhosts/test/htdocs
     ErrorLog logs/test-error_log
     CustomLog logs/test-access_log common
   
     PerlModule APR::Table
     PerlModule MpTest::AhoAho
     <Location /aho>
       SetHandler modperl
       PerlResponseHandler MpTest::AhoAho
     </Location>
   </VirtualHost>
   ```
1. [せっかく思い出しちゃったので](http:://blog.harapeko.jp/2008/12/29/mod-perl-aho/)、 mod_perl 版あほプログラムを移転前サーバーからコピー。
  * 移転前サーバーにて:
    ```
    $ cd
    $ sudo cp /etc/httpd/MpTest/AhoAho.pm .
    $ sudo chown murachi:apache AhoAho.pm
    $ chmod 640 AhoAho.pm
    ```
  * 移転先サーバーにて:
    ```
    $ cd
    $ rsync -ae ssh onaka.harapeko.jp:/home/murachi/AhoAho.pm .
    $ sudo mkdir /etc/httpd/MpTest
    $ sudo chown murachi:apache /etc/httpd/MpTest
    $ chmod 750 /etc/httpd/MpTest
    $ sudo chmod g+s /etc/httpd/MpTest
    $ cp AhoAho.pm /etc/httpd/MpTest/
    ```
1. Apache を再起動。
   ```
   $ sudo /etc/init.d/httpd restart
   ```
  * hosts に test.harapeko.jp を追加してから、 http://test.harapeko.jp/aho にアクセスすると、ちゃんとあほプログラムが動作した。

### MySQL を設定

MySQL のインストール、および設定、起動を行う。

1. MySQL __サーバー__をインストールする (クライアントは PHP をインストールした際にインストールされている)。
   ```
   $ sudo yum install mysql-server
   ```
1. MySQL の設定。
   ```
   $ sudo vim /etc/my.cnf
   
   [mysqld]
   
   # mysqld ディレクティブ内に以下の行を追加
   default-character-set=utf8
   character-set-server=utf8
   collation-server=utf8_general_ci
   init-connect=SET NAMES utf8
   
   # client, mysqldump 各ディレクティブはそもそも存在しなかった。以下全て追加。
   [client]
   default-character-set=utf8
   
   [mysqldump]
   default-character-set=utf8
   ```
  * やりたいことはだいたい解ると思うが、何をやるにしてもデフォルトの文字セットは UTF-8 にしたいですよ、ということ。
1. MySQL サーバーを起動する。
   ```
   $ sudo /etc/init.d/mysqld start
   ```
1. MySQL サーバーをデフォルトで起動するようにする。
   ```
   $ sudo /sbin/chkconfig --add mysqld
   $ sudo /sbin/chkconfig mysqld on
   ```
1. MySQL の root ユーザーのパスワードを設定。もちろんパスワードはひみちゅ。
   ```
   $ mysqladmin -u root -p password '新しいパスワード'
   $ mysql -u root -p mysql    # 新しいパスワードで接続できるか確認してみる
   ```
1. とりあえずブログが動くかどうか試してみたいので、ブログ用のテーブルだけ移行してみることにする。
  * 移転前サーバーにて:
    ```
    $ cd
    $ mkdir tempsql
    $ cd tempsql
    $ mysqldump -u harapeko_press -p harapeko_press > harapeko_press.sql
    ```
  * 移転先サーバーにて:
    ```
    $ cd
    $ mkdir tempsql
    $ rsync -ae ssh onaka.harapeko.jp:/home/murachi/tempsql/harapeko_press.sql tempsql/
    $ mysql -u root -p
    
    mysql> create database harapeko_press;
    mysql> grant all privileges on harapeko_press.* to harapeko_press@localhost identified by '移転前と同じパスワード';
    mysql> quit
    
    $ mysql -u harapeko_press -p harapeko_press <tempsql/harapeko_press.sql
    ```
  * hosts に blog.harapeko.jp を追加して http://blog.harapeko.jp/ にアクセスしたところ、全く遜色なくブログが表示された。

## 開発用リポジトリの設定

### Subversion のインストール

Subversion 自体は既に入っていた。念のため、 Apache モジュールのみインストールする。

```
$ sudo yum install mod_dav_svn
```

### Trac のインストール

基本的にやることは[ここに書いたこと](http:://blog.harapeko.jp/2008/12/18/developer-repository-structure/)と同じ。

1. python-setuptools をインストール。
   ```
   $ sudo yum install python-setuptools
   ```
1. yum にリポジトリを追加する。
   ```
   $ sudo vim /etc/yum.repos.d/CentOS-Base.repo
   
   # 以下全て追加
   [dag]
   name=Dag RPM Repository for Red Hat Enterprise Linux
   baseurl=http://apt.sw.be/redhat/el$releasever/en/$basearch/dag
   gpgcheck=1
   enabled=1
   includepkgs=clearsilver python-clearsilver trac
   gpgkey=http://dag.wieers.com/packages/RPM-GPG-KEY.dag.txt
   
   [kbs-CentOS-Extras]
   name=CentOS.Karan.Org-EL$releasever - Stable
   gpgcheck=1
   gpgkey=http://centos.karan.org/RPM-GPG-KEY-karan.org.txt
   enabled=1
   baseurl=http://centos.karan.org/el$releasever/extras/stable/$basearch/RPMS/
   includepkgs=python-docutils python-imaging
   
   $ sudo yum --enablerepo=dag --enablerepo=kbs-CentOS-Extras update
   ```
1. [python-genshi](http:://packages.sw.be/python-genshi/) と [python-sqlite2](http:://packages.sw.be/python-sqlite2/) をインストール。
   ```
   $ cd
   $ mkdir -p download/site-pkg
   $ cd download/site-pkg/
   $ wget http://packages.sw.be/python-genshi/python-genshi-0.6-1.el5.rf.noarch.rpm
   $ wget http://packages.sw.be/python-sqlite2/python-sqlite2-2.3.3-1.el5.rf.x86_64.rpm    # さくらの VPS は 64bits だよ!!
   $ sudo rpm -i python-genshi-0.6-1.el5.rf.noarch.rpm
   $ sudo rpm -i python-sqlite2-2.3.3-1.el5.rf.x86_64.rpm
   ```
1. yum を使って Trac をインストールし、すぐさまアンインストールする。
   ```
   $ sudo yum install trac
   $ sudo yum remove trac
   ```
1. [日本語版 Trac](http:://www.i-act.co.jp/project/products/products.html) をインストール。
   ```
   $ wget http://www.i-act.co.jp/project/products/downloads/Trac-0.12.ja1.zip
   $ unzip Trac-0.12.ja1.zip
   $ cd Trac-0.12.ja1
   $ sudo .setup.py install
   $ sudo cp -R trac /usr/lib/python2.4/site-packages/     # 何故か必要だった…。
   ```
1. 念のため、 Apache を再起動する。
   ```
   $ sudo /etc/init.d/httpd restart
   ```

### データを移行する

リポジトリディレクトリ、および Trac の DB データを移行する。

1. リポジトリディレクトリを移行する。
  * 移行前サーバーにて:
    ```
    $ sudo chown -R murachi:apache /var/Developer
    ```
  * 移行先サーバーにて:
    ```
    $ sudo mkdir /var/Developer
    $ sudo chown murachi:apache /var/Developer
    $ rsync -ae ssh onaka.harapeko.jp:/var/Developer /var/
    $ rm -rf /var/Developer/trac/trust/tuh-system/          # 取引先様の提供されるサーバーに移行した為、削除。
    $ rm -rf /var/Developer/svn/trust/tuh-system/           # 同上。
    $ sudo chown -R murachi:apache /var/Developer/trac      # Trac リポジトリのグループを全て apache にする。
                                                            # 何故か murachi になってたので…。
                                                            # svn リポジトリは複数人数体制になってポリシーが固まるまでとりあえず放置。
    ```
1. Trac の DB データを移行する。
  * 移行前サーバーにて:
    ```
    $ cd ~/tempsql/
    $ mysqldump -u trac_otoco -p trac_otoco >trac_otoco.sql
    $ mysqldump -u trac_ideanote -p trac_ideanote >trac_ideanote.sql
    ```
  * 移転先サーバーにて:
    ```
    $ cd ~/tempsql/
    $ rsync -ae ssh onaka.harapeko.jp:/home/murachi/tempsql/trac_otoco.sql .
    $ rsync -ae ssh onaka.harapeko.jp:/home/murachi/tempsql/trac_ideanote.sql .
    $ mysql -u root -p
    
    mysql> create database trac_otoco;
    mysql> create database trac_ideanote;
    mysql> grant all privileges on trac_otoco.* to trac_master@localhost identified by '新しいパスワード';
    mysql> grant all privileges on trac_ideanote.* to trac_master@localhost;
    mysql> quit
    
    $ mysql -u trac_master -p trac_otoco <trac_otoco.sql
    $ mysql -u trac_master -p trac_ideanote <trac_ideanote.sql
    $ vim /var/Developer/trac/original/otoco/conf/trac.ini
    
    # DB のユーザー名を変更したので、修正。
    database = mysql://trac_master:パスワード@localhost/trac_otoco
    
    $ vim /var/Developer/trac/original/ideanote/conf/trac.ini
    
    # こちらも同様に修正。
    database = mysql://trac_master:パスワード@localhost/trac_ideanote
    ```
1. Trac のバージョンが異なるので、 Trac リポジトリをアップグレードする。
   ```
   $ trac-admin /var/Developer/trac/original/otoco upgrade
   $ trac-admin /var/Developer/trac/original/otoco wiki upgrade
   $ trac-admin /var/Developer/trac/original/otoco repository resync '*'
   $ trac-admin /var/Developer/trac/original/ideanote upgrade
   $ trac-admin /var/Developer/trac/original/ideanote wiki upgrade
   $ trac-admin /var/Developer/trac/original/ideanote repository resync '*'
   ```

ここまでで、 hosts に developer.harapeko.jp を追加し、 http://developer.harapeko.jp/trac/original にアクセスすると、 Trac が問題なく移行できていることが確認できた。

### Trac のプラグインをインストール

[FootNoteMacro](http:://trac-hacks.org/wiki/FootNoteMacro) を使っていたので、それをインストールする。

1. wget ではうまくダウンロードできないので、手元のマシンで[ここ](http:://trac-hacks.org/wiki/FootNoteMacro)からダウンロードし、 rsync にて転送する。
   ```
   $ rsync -ae ssh footnotemacro-r9282.zip www6240u.sakura.ne.jp:/home/murachi/download/site-pkg
   ```
1. 展開し、インストールする。
   ```
   $ cd ~/download/site-pkg/
   $ unzip footnotemacro-r9282.zip
   $ cd footnotemacro/0.11/
   $ sudo python setup.py install
   ```
1. Apache を再起動する。
   ```
   $ sudo /etc/init.d/httpd restart
   ```

さらに、ソースコードのシンタックスカラーリングを行う [Pygments](http:://bitbucket.org/birkenfeld/pygments-main) をインストールする。

```
$ cd ~/download/site-pkg/
$ wget -O pygments-1.3.1.tar.bz2 http://bitbucket.org/birkenfeld/pygments-main/get/1.3.1.tar.bz2
$ tar -xjvf pygments-1.3.1.tar.bz2
$ cd pygments-main/
$ sudo ./setup.py install
$ sudo /etc/init.d/httpd restart
```

[このへん](http:://developer.harapeko.jp/trac/original/ideanote/wiki/HowTo/CTutorial)を見て、フットノートもカラーリングもうまくいっていることを確認。

## ドメイン参照先変更

サーバーアプリケーションの設定は概ね済んだので、ドメイン名の参照先アドレスを変更し、 BIND を起動する。

1. 弊社ではドメイン登録に [JPDirect](http:://jpdirect.jp/) を利用している。ここで行う操作は以下の通り。
  1. ページ右上の「汎用ユーザー専用」と書かれたログインボタンをクリックしてログインする。
  1. メニューの「ホスト情報変更」をクリックして、登録している NS.HARAPEKO.JP および ONAKA.HARAPEKO.JP の IP アドレスを、新しいアドレスに変更する。
1. 移転先サーバーにて、 BIND を起動し、デフォルトで起動するようにする。
   ```
   $ sudo /etc/init.d/named start
   $ sudo /sbin/chkconfig --add named
   $ sudo /sbin/chkconfig named on
   ```
1. しばらく待って、[会社概要ページ](http:://www.harapeko.jp/summary.php)の内容が変化するのを確認する。
  * 旧住所 (八千代台) で表示されていたのが、新住所 (鷺沼台) に変化する。
  * もちろん、事前に hosts を元に戻しておくこと。

## SSL 証明書を更新

### 登録メールアドレスのメールを一時的に読めるようにする

弊社では現在 SSL 認証局として [RapidSSL](http:://www.rapidssl.com/) を利用している。ここの右の方にある「Reissue SSL」をクリックしてみたところ、認証を求められ、必要事項を記入したところ、登録していたメールアドレス宛にメールが送られた。ところがそのメールアドレスはまさに harapeko.jp ドメインのもの。今のメールサーバーの設定ではメールを読みに行くことができない (配送はされるはず)。

そこで、とりあえず登録しているメールアドレス用のアカウントを作成し、 .forward を書いて別の使えるアドレス宛に転送することにした。

1. RapidSSL の場合、登録者のメールアドレス (登録時のもの) と承認者のメールアドレス (admin@example.jp 的なもの) の 2つのメールアドレスが受信可能である必要がある。まず、登録時のメール用アカウントを作成する。通常このアカウントでログインできる必要はないので、「-s /sbin/nologin」オプションを指定してあげるのがミソ。
   ```
   $ sudo su -
   # useradd -m -s /sbin/nologin toshiyuki.murayama
   # passwd toshiyuki.murayama
   ```
1. .forward ファイルを作成。メールボックスにメールを貯めておいて欲しいので、必ず最後の行にバックスラッシュ付きで自身のメールアドレスを指定するのを忘れずに。
   ```
   # sudo su - -s /bin/bash toshiyuki.murayama
   $ vim .forward
   
   携帯電話のメールアドレス
   一時的に転送先にするメールアドレス
   \toshiyuki.murayama あっと、マー君 harapeko.jp
   
   $ chmod g-w .forward
   ```
1. 次に、承認者のメールアドレスを作成する。
   ```
   $ exit      # toshiyuki.murayama アカウントから抜ける
   # useradd -m -s /sbin/nologin admin
   ```
1. .forward ファイルを作成。こちらは単に自分のメールアドレスの別名として振る舞えばいいので…。
   ```
   # sudo su - -s /bin/bash admin
   $ vim .forward
   
   toshiyuki.murayama おっと、マイケル? harapeko.jp
   
   $ chmod g-w .forward
   ```

### 秘密鍵ファイルと CSR ファイルを作成

コマンドラインを一気にどうぞ (exit しまくって元のユーザーに戻ったものとして)。ちなみにこの辺のやり方は認証局によって微妙に違うので、別の認証局を使う人はその認証局が示すやり方を参照して下さい。

```
$ cd
$ mkdir tempssl
$ cd tempssl
$ openssl genrsa -out developer.harapeko.jp.key 1024
$ openssl req -new -key developer.harapeko.jp.key -out developer.harapeko.jp.csr

Country Name (2 letter code) [GB]:JP
State or Province Name (full name) [Berkshire]:Chiba
Locality Name (eg, city) [Newbury]:Narashino
Organization Name (eg, company) [My Company Ltd]:Harapeko Inc.
Organizational Unit Name (eg, section) []:Engineering Section
Common Name (eg, your name or your server's hostname) []:developer.harapeko.jp
Email Address []:
A challenge password []:
An optional company name []:

$ cat developer.harapeko.jp.csr
```

これによって、秘密鍵ファイル (.key ファイル) と CSR ファイルが生成されます。最後に CSR ファイルを cat しているのは、あとで再発行を依頼する際にこの内容をコピペするためです。

### SSL 証明書再発行

いよいよ再発行。

1. [RapidSSL](http:://www.rapidssl.com/) のサイトにて、画面右の「Reissue SSL」をクリックする。
1. 別窓が開くので、登録しているドメイン名またはサブドメイン名、およびメールアドレスと、画面に表示されている数字を入力。
1. 登録しているメールアドレス宛にメールが送られてくるので、そこに書かれたアドレスをブラウザで開く。
1. 画面左の「Reissue Certificate」をクリックする。
1. CSR の入力を求められるので、入力欄に、先ほど cat したファイルの中身をそのままコピペし、送信する。
1. 承認者用のメールアドレス宛にメールが送られてくるので、そこに書かれたアドレスをブラウザで開く。
1. 内容を確認し、「承認する」ボタンをクリックする。
1. 登録しているメールアドレス宛にメールが送られ、その文末に SSL 証明書が記述されている。
   ```
   -----BEGIN CERTIFICATE-----
     :   :   :
   証明書の内容 (意味不明な文字列…)
     :   :   :
   -----END CERTIFICATE-----
   ```

### SSL 証明書を Apache に適用する

一応 [RapidSSL のサポートサイトによる説明](https:://knowledge.rapidssl.com/support/ssl-certificate-support/index?page=content&id=SO6252&actp=search&viewlocale=en_US&searchid=1286970866293)を参考にしています。ディレクトリ名とか細かい部分は任意でいいと思う。

1. SSL 証明書をファイルに保存する。
   ```
   $ cd ~/tempssl/
   $ vim developer.harapeko.jp.crt
   
   -----BEGIN CERTIFICATE-----
     :   :   :
   証明書の内容 (意味不明な文字列…)
     :   :   :
   -----END CERTIFICATE-----
   ```
1. Apache の設定ファイルがあるディレクトリに証明書ファイルと秘密鍵ファイルをコピーする。
   ```
   $ sudo mkdir /etc/httpd/conf/ssl
   $ sudo cp developer.harapeko.jp.crt /etc/httpd/conf/ssl/
   $ sudo cp developer.harapeko.jp.key /etc/httpd/conf/ssh/
   ```
1. そういえば mod_ssl をまだインストールしていなかった。インストールする。
   ```
   $ sudo yum install mod_ssl
   ```
1. mod_ssl の設定。
   ```
   $ sudo vim /etc/httpd/conf.d/ssl.conf
   
   <VirtualHost _default_:443>
   
   # バーチャルホストとしての設定はもちろん必要。
   DocumentRoot /var/www/vhosts/developer/htdocs
   ServerName developer.harapeko.jp
   ErrorLog logs/developer-ssl_error_log
   TransferLog logs/developer-ssl_access_log
   LogLevel warn
   
   # 以下の 2行を、実際のファイルの場所に書き換える。
   SSLCertificateFile /etc/httpd/conf/ssl/developer.harapeko.jp.crt
   SSLCertificateKeyFile /etc/httpd/conf/ssl/developer.harapeko.jp.key
   ```
1. Apache を再起動する。
   ```
   $ sudo /etc/init.d/httpd restart
   ```

https://developer.harapeko.jp/ にアクセスし、以下の 2点を確認する。

* 「準備中」と書かれたページが表示される。
* ブラウザのステータスバーに鍵マークが表示され、クリックすると Web サイトの識別情報が表示される。

なお、移転前サーバーではこの配下に受託業務用の Trac を運用していたが、現在ではその業務のお取引先様よりご提供頂いているサーバーに移転してしまったため、今回はその辺の設定まではしなかった。忘れると後々辛いので、備忘録としてそれ用に ssl.conf 内に記述していた設定内容を以下に掲載しておくことにする。

```
# trac location
<Location /trac/trust>
   SetHandler mod_python
   PythonDebug On
   PythonHandler trac.web.modpython_frontend
   PythonOption TracEnvParentDir /var/Developer/trac/trust
   PythonOption TracUriRoot /trac/trust
</Location>

<Location /trac/trust>
   AuthType Digest
   AuthName realm
   AuthUserFile "/var/Developer/trac/trust/.htdigest"
   Require valid-user
</Location>
<Location /trac/trust/tuh-system>
   AuthType Digest
   AuthName realm
   AuthUserFile "/var/Developer/trac/trust/tuh-system/.htdigest"
   Require valid-user
</Location>
```

## メールサーバーを再度設定する

SSL が使えるようになったので、 POP3S と Submission over SSL/TLS の設定をして、メールが利用できるようにする。

### メールユーザー DB を構築

弊社メールサーバーの動作用件は以下の通り。

* クライアントとの接続プロトコルは POP3S と Submission over SSL/TLS を用いる。 POP3 は使わない。
  * 25番ポート (SMTP) はメール配送用に開けておく必要がある。
* Linux アカウントのパスワードと同じパスワードを使わせたくない。
* 受信/送信の両方で同一のパスワードとして管理したい。

POP3S の認証は dovecot にて、 Submission over SSL/TLS の認証は cyrus-SASL にて行う。この場合、

* cyrus-SASL では /etc/shadow や PAM を参照する saslauthd ではなく、 auxprop を用いる。 auxprop がサポートしている参照先は以下のいずれかである。
  * sasldb (cyus-SASL 独自形式の DB; 解説をもっともよく見かける)
  * DBMS (MySQL/PostgleSQL/SQLite)
  * LDAP
* 上記のうち、 sasldb は dovecot が理解できず、双方で同一のパスワードを管理できなくなってしまうので、NG。
* 別にアプリケーション間で共通のパスワードを一元管理したいわけでもないので、わざわざ LDAP 使う必要もないかな。

というわけで、せっかく MySQL も入れてあるし、 MySQL でメールユーザー DB を構築することにしてみた。

なお、 dovecot はメール配送時のユーザー情報参照にもこの DB を利用することになる為、その内容はユーザー ID とパスワードの他、メールサーバーのドメイン名、ホームディレクトリ、UID、GID も必要になる。この辺の情報については dovecot をインストール後、 /usr/share/doc/dovecot-x.x.x/wiki/AuthDatabase.SQL.txt ファイルを参照のこと。

1. DB を作成。 DB 名、 DB ユーザー名共に mailusers とした。
   ```
   $ mysql -u root -p
   
   mysql> create database mailusers;
   mysql> mysql> grant all privileges on mailusers.* to mailusers@localhost identified by 'パスワード';
   mysql> use mailusers;
   mysql> create table users (
       ->   userid varchar(128) primary key,
       ->   domain varchar(128),
       ->   plain_passwd varchar(64),
       ->   home varchar(255) not null,
       ->   uid integer not null,
       ->   gid integer not null
       -> );
   mysql> quit
   ```
  * ドメイン名とパスワードは NULL 可とした。メール配送は受け付けるが転送するだけで認証することのないアカウント (root, admin など) もある為。
1. mailusers でのアクセス確認を兼ねて、テーブルにさしあたって必要なメール用のアカウントをいくつか登録してみる。
   ```
   $ mysql -u mailusers -p mailusers
   
   mysql> insert users values
       ->   ('toshiyuki.murayama', 'developer.harapeko.jp', 'パスワード', '/home/toshiyuki.murayama', 502, 502),
       ->   ('root', null, null, '/root', 0, 0),
       ->   ('admin', null, null, '/home/admin', 503, 503);
   mysql> quit
   ```
  * ホームディレクトリや UID, GID は事前に /etc/passwd を参照して確認しておくこと。
  * パスワードは、危険だが平文で保存する。ちなみに dovecot は MD5 ハッシュ、 cyrus-SASL は平文を要求する… cyrus も MD5 に対応してくれよ… orz
  * もちろんパスワードは Linux アカウントのパスワードと同じである必要はない。むしろセキュリティリスクを抑える為にも、別々のパスワードを設定すべきである。

### POP3S の設定

1. dovecot をインストールする。
   ```
   $ sudo yum install dovecot
   ```
1. dovecot の設定。
   ```
   $ sudo vim /etc/dovecot.conf
   
   # POP3S にのみ対応する。
   protocols = pop3s
   
   # 接続対象アドレス範囲。 IPv6 で指定しているが、暗黙に IPv4 アドレスも含むことになる。
   listen = [::]
   
   # SSL 証明書ファイル、および鍵ファイルの場所を指定。
   ssl_cert_file = /etc/httpd/conf/ssl/developer.harapeko.jp.crt
   ssl_key_file = /etc/httpd/conf/ssl/developer.harapeko.jp.key
   
   # Maildir 形式を使用する。
   mail_location = maildir:~/Maildir
   
   auth default {
     # 平文認証、ログイン認証、MD5 ダイジェスト認証、CRAM-MD5 に対応する。
     mechanisms = plain login digest-md5 cram-md5
   
     # 認証に PAM は使わないので、コメントアウト
     #passdb pam {
     # (中略)
     #}
   
     # 認証に DBMS を利用する
     passdb sql {
       # DBMS への接続方法を記した設定ファイルのパス。あとで作成
       args = /etc/dovecot-sql.conf
     }
   
     # ユーザー情報も DB を参照するようにする。コメントアウト
     #userdb passwd {
     # (中略)
     #}
   
     # ユーザー情報を DBMS にて参照する。
     userdb sql {
       args = /etc/dovecot-sql.conf
     }
   }
   ```
1. DBMS への接続方法を示す設定ファイルを作成する。
   ```
   $ sudo vim /etc/dovecot-sql.conf
   
   # MySQL を使用する。
   driver = mysql
   # 接続情報 (ホスト名、DB 名、DB ユーザーの ID/パスワード)
   connect = host=localhost dbname=mailusers user=mailusers password=パスワード
   # 認証の際のクエリー SQL - パスワードは MD5 ハッシュを要求するので、 MD5() 関数を噛ませる
   password_query = select concat(userid, '@', domain) as user, md5(plain_passwd) as password from users where userid = '%n'
   # ユーザー情報を参照する際のクエリー SQL
   user_query = select home, uid, gid from users where userid = '%u'
   
   $ sudo chmod 600 /etc/dovecot-sql.conf  # DB ユーザーのパスワードを知られてはマズイので、権限を変更しておく
   ```
1. dovecot を起動し、デフォルトで起動するようにする。
   ```
   $ sudo /etc/init.d/dovecot start
   $ sudo /sbin/chkconfig --add dovecot
   $ sudo /sbin/chkconfig dovecot on
   ```

### Submission over SSL/TLS の設定

1. cyrus-SASL と、それ用の MD5 認証用プラグインと SQL auxprop プラグインをインストールする。
   ```
   $ sudo yum install cyrus-sasl cyrus-sasl-md5 cyrus-sasl-sql
   ```
1. cyrus-SASL の設定。設定方法は[ここ](http:://www.postfix.org/SASL_README.html#auxprop_sql)を参考にした。
   ```
   $ vim /etc/sasl2/smtpd.conf
   
   pwcheck_method: auxprop
   auxprop_plugin: sql
   mech_list: cram-md5 digest-md5 plain login
   sql_engine: mysql
   sql_hostnames: localhost
   sql_user: mailusers
   sql_passwd: パスワード
   sql_database: mailusers
   sql_select: select plain_passwd as password from users where userid = '%u'
   
   $ sudo chmod 600 /etc/sasl2/smtpd.conf  # DB ユーザーのパスワードを知られてはマズイので、権限を変更しておく
   $ rm -f /usr/lib64/sasl2/smtpd.conf /usr/lib64/sasl/smtpd.conf  # この辺のファイルが残っていると悪さをするので消しておく
   ```
  * **最後のファイル削除は超重要**。これで何時間嵌ったことか…。orz
  * (*2011/8/17 追記*) cyrus-SASL をアップデートすると、上記で削除したファイルが復活してしまうことがあるようなので注意。 yum update したらメールが送信できなくなったとかいう場合は多分これが原因。…もしかしたらこの辺のファイルも上記の内容に書き換えた方が安全かも。
1. Postfix の設定に、 SASL 関連の設定と、サブミッションポートの設定を追加する。 main.cf の設定パラメータについては[こちら](http:://www.postfix-jp.info/trans-2.2/jhtml/postconf.5.html)を参照。
   ```
   $ sudo vim /etc/postfix/main.cf
   
   # SMTP の SASL による認証を有効にする
   smtpd_sasl_auth_enable = yes
   # 匿名での利用、および平文認証を禁止する
   smtpd_sasl_security_options = noanonymous, noplaintext
   # realm の名前。ホスト名にしておく
   smtpd_sasl_local_domain = $myhostname
   # SSL 証明書および秘密鍵ファイル
   smtpd_tls_cert_file = /etc/httpd/conf/ssl/developer.harapeko.jp.crt
   smtpd_tls_key_file = /etc/httpd/conf/ssl/developer.harapeko.jp.key
   # STARTTLS サポートをクライアントに通知する
   smtpd_use_tls = yes
   # TLS 暗号化されていない接続を受け付けない
   smtpd_tls_auth_only = yes
   # SASL 認証に成功した場合のみ、 SMTP によるメールの配送 (RCPT TO コマンド) を受け付けるようにする
   # (この順番で書くことにより、サーバー上で sendmail コマンドを用いる場合 (Web アプリからのメール発送等)
   # には SASL 認証が不要になる)
   smtpd_recipient_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_unauth_destination
   
   $ sudo vim /etc/postfix/master.cf
   
   # サブミッションポートを有効にする
   submission inet n       -       n       -       -       smtpd
     -o smtpd_sasl_auth_enable=yes
   ```
1. Postfix を再起動する。
   ```
   $ sudo /etc/init.d/postfix restart
   ```

### メールアカウントの追加

このシステムにおけるメールアカウントの追加方法を記しておく。以下はメールアドレスが "hoge (あっとまーく) harapeko.jp" になる場合。

1. メールアカウントと同名の Linux アカウントを作成する。
  * Linux 上での操作にも使用する場合 (-G オプションは任意)。
    ```
    $ sudo /usr/sbin/useradd -m -G users,apache hoge
    $ passwd hoge
    ```
  * メールアカウントとしてのみ使用する場合。
    ```
    $ sudo /usr/sbin/useradd -m -s /sbin/nologin hoge
    ```
1. ホームディレクトリ、UID、GID を確認する。
   ```
   $ cat /etc/passwd | grep hoge
   hoge:x:506:506::/home/hoge:/sbin/nologin
   ```
  * 左から、アカウント ID、パスワード (shadow を使っているので "x")、UID、GID、クラス (アプリケーション用のユーザーではないので空欄)、ホームディレクトリ、ログインシェル。
1. メールユーザー DB にアカウント情報を登録する。
   ```
   $ mysql -u mailusers -p mailusers
   
   mysql> insert users values ('hoge', 'developer.harapeko.jp', 'password', '/home/hoge', 506, 506);
   mysql> quit
   ```
1. 以下の内容で手元のメール UA を設定する。
  * 受信プロトコル (サーバー種別): POP3s
    * サーバー名 (ホスト名): developer.harapeko.jp
    * ポート: デフォルト (995)
    * ユーザー ID: hoge
    * パスワード: password
  * 送信プロトコル (サーバー種別): SMTP
    * STARTTLS (または "SMTP over SSL/TLS") を使用する (というチェックボックスを選択するか、もしくはサーバー種別として選択する)。
    * サーバー名 (ホスト名): developer.harapeko.jp
    * ポート: サブミッションポート (587)
    * SMTP は認証が必要
      * ユーザー ID: hoge
      * パスワード: password
      * 認証方式: CRAM-MD5 または Digest-MD5 ("MD5 チャレンジ認証" など)

今のところ、 [EdMax フリー版](http:://www.edcom.jp/edmaxtop.html)と Max OS X Mail にて動作確認済み。

## おわりに

### 便利なコマンド

* パッケージが導入済みかどうかを調べる方法。
  ```
  $ rpm -qa phrase*
  ```
  例えば cyrus-sasl 関連のパッケージについて調べる場合。
  ```
  $ rpm -qa cyrus-sasl*
  cyrus-sasl-plain-2.1.22-5.el5_4.3
  cyrus-sasl-md5-2.1.22-5.el5_4.3
  cyrus-sasl-lib-2.1.22-5.el5_4.3
  cyrus-sasl-plain-2.1.22-5.el5_4.3
  cyrus-sasl-2.1.22-5.el5_4.3
  cyrus-sasl-2.1.22-5.el5_4.3
  cyrus-sasl-sql-2.1.22-5.el5_4.3
  cyrus-sasl-md5-2.1.22-5.el5_4.3
  cyrus-sasl-lib-2.1.22-5.el5_4.3
  cyrus-sasl-sql-2.1.22-5.el5_4.3
  ```
* パッケージの依存関係を調べる方法。
  * 何を必要としているか:
    ```
    $ rpm -q --whatprovides package
    ```
  * 何に必要とされているか:
    ```
    $ rpm -q --whatrequires package
    ```
  * yum で依存関係のツリーを表示する。
    ```
    $ yum deplist package | less
    ```
* 最近出力されたログ内容を調べるには tail コマンドが便利。
  ```
  $ sudo tail /var/log/messages
  ```
  * sudo しないと閲覧すらできないログが多い。
* dovecot による認証の様子を詳しくログに出力する方法。
  ```
  $ sudo vim /etc/dovecot.conf
  
  info_log_path = /var/log/dovecot-info.log
  log_timestamp = "%b %d %H:%M:%S "
  auth_verbose = yes
  auth_debug = yes
  auth_debug_passwords = yes
  
  $ sudo /etc/init.d/dovecot restart
  $ sudo tail /var/log/dovecot-info.log   # ログ閲覧時
  ```
  * 認証がうまくいかない原因を調査したい場合とかに便利。

### 反省点

* 時間かかりすぎた orz 。
  * やった作業の日付もメモっておくべきだったかな。
* 必要なメールアカウント用の Linux アカウントは最初に Postfix を設定する際に作っておくべきだった。いくつかのメールが届かずに捨てられてしまったかも…。
* 移転前サーバーのデータはあとでちゃんと消しておかないとなぁ…。


以上。
