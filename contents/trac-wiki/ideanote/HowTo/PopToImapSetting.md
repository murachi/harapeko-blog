[author: murachi]
# POP3 を IMAP4 に移行する

これまでメールの蓄積管理はローカルマシン (ThinkPad) 上の EdMax に任せていたが、以下の事由により、サーバーサイドでメールボックスを管理したくなった。

* 客先常駐の仕事でネットにはアクセスできるが ThinkPad は持ち込めない。しかし客先業務に直接関係しないメールのやりとりで、客先で割り当てられたメールアドレスを使いたくない (というより仲介業者の営業さんが使いたがらない)。
* そも ThinkPad は使い始めて早 5年、未だに XP だったり、サードパーティー製のバッテリーパックに換装していたりと年季が入っていて、メール保管用マシンとして使い続けるのはいささか危なっかしい、というよりそろそろリプレースしたいと思っている。
* そうでなくてもデスクトップマシンで直接メールを閲覧できない現状は何かと不便だし、このまま Windows をホスト OS として使い続けるとも限らないと思い始めている。などなど…。

一方で、メールボックスの管理をサーバーサイドに移行した場合、以下のような難点を抱えることにもなる。

* spam を放置するとサーバーのディスクスペースを圧迫するため、利用するアカウントについてはこまめにメールの整理をする必要がある。
* メールは平文で保存されるため、セキュリティ上のリスクがある。
* サーバーを移転する際にはメールボックスもアカウントごとにコピーする必要があり、若干面倒くさい。
  * 社員を抱えることになった場合や窓口を増やした場合など、手作業ではまかないきれなくなった場合には、事故防止策含めて何らかの対策が必要かも…。

## 関連記事

* [さくらの VPS セットアップメモ](wiki::HowTo/SakuraVpsSetup)
* [さくらの VPS セットアップメモ ～2G プラン乗り換え + Scientific Linux 編～](wiki::HowTo/SakuraVpsSetup2)

## dovecot 設定

まず dovecot.conf にて、 protocol 変数を imap に変更。

```
$ sudo su -
# vim /etc/dovecot/dovecot.conf

protocols = imap

```

次に conf.d/10-master.conf を編集。

```
# vim /etc/dovecot/conf.d/10-master.conf

# 以下、コメントアウトを外す。
service imap-login {
  inet_listener imap {
    # SSL/TLS だけを許容したいので、 imap は port を潰しておく。
    #port = 143
    port = 0
  }
  inet_listener imaps {
    port = 993
    ssl = yes
  }

  # 接続ごとにプロセスを生成する場合は1。規定値。
  # 0にすると 1つのプロセスで全接続を受け付けるので高速になるが、 1 の方がセキュアらしい。
  #service_count = 1

  # 待ち受けプロセスをあらかじめいくつか起動させておきたい場合は値を設定する。
  #process_min_avail = 0

  # service_count = 0 にするのであれば増やした方がいいんだそうな。
  #vsz_limit = 64M
}

# もはや POP3 は使わないので、以下はコメントアウト。
#service pop3-login {
#  inet_listener pop3 {
#    #port = 110
#    port = 0
#  }
#  inet_listener pop3s {
#    port = 995
#    ssl = yes
#  }
#}
```

さらに conf.d/10-director.conf も編集。ていうか元々使わない方をコメントアウトとかする必要無かった気もする。

```
# vim /etc/dovecot/conf.d/10-director.conf

# コメントアウトしていたのを外す。
service imap-login {
  #executable = imap-login director
}
```

設定の変更はこんだけ。あとは dovecot を再起動すれば ok。

```
# /etc/init.d/dovecot restart
```

## ポートを開ける

IMAPS のポートは 993。これを開ける必要があります。というわけで iptables の設定。

```
# vim /etc/sysconfig/iptables

# 995 を 993 に変更。
-A RH-Firewall-1-INPUT -m state --state NEW -m multiport -p tcp --dports 22,25,587,993,80,443 -j ACCEPT

```

iptables も再起動しましょう。

```
# /etc/init.d/iptables restart
```

## 動作確認

まず POP3S ではメールを受信できないことを確認する。従来使っていたメーラー上のアカウント設定でメール受信を試みて、何らかエラーが出て失敗すれば ok。

次に、メーラー上で IMAPS を利用したアカウント設定を新規に作成し、メールの受信を試みる。受信できたら、別の端末のメーラーで、もしくは別のメーラーをインストールした上で、同じアカウントの設定を行い、こちらでも同じメールが受信できることを確認する。
