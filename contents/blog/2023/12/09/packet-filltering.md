[author: T.MURACHI]
# サーバー設定日記: 2023/12/9
## パケットフィルタリング
ConoHa はコントロールパネルの設定で IP フィルタリングできる機能があるんだけど、開放できるポートが限られていて ping が通らなくて困ったので、やっぱり自分で設定することにしました。

[前回 Ubuntu 鯖に設定したとき](/trac-wiki/ideanote/HowTo/SakuraVpsSetup3.html)とやり方はほぼ同じです。(設定内容は若干アップデートがあるかもですが)

まずは `iptables` コマンド、 `ip6tables` コマンドを呼びまくるシェルスクリプトを作って実行しちゃいます。

```console
$ sudo -i
# vim iptables.sh
```

```sh
#!/bin/sh

/sbin/iptables -F
/sbin/iptables -X
/sbin/iptables -P INPUT ACCEPT
/sbin/iptables -P OUTPUT ACCEPT
/sbin/iptables -P FORWARD ACCEPT

/sbin/iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
/sbin/iptables -A INPUT -p icmp -j ACCEPT
/sbin/iptables -A INPUT -i lo -j ACCEPT

/sbin/iptables -A INPUT -m tcp -p tcp -m state --state NEW --dport 53 -j ACCEPT
/sbin/iptables -A INPUT -m udp -p udp -m state --state NEW --dport 53 -j ACCEPT
/sbin/iptables -A INPUT -p tcp -m state --state NEW -m multiport --dports 22,25,587,993,80,443 -j ACCEPT
#/sbin/iptables -A INPUT -p tcp -m state --state NEW -m multiport --dports 22,25,587,993,80,443,6543 -j ACCEPT
/sbin/iptables -A INPUT -j REJECT --reject-with icmp-port-unreachable
/sbin/iptables -A FORWARD -j REJECT --reject-with icmp-port-unreachable

# SNMP blocking
/sbin/iptables -A OUTPUT -m udp -p udp -m multiport --dports 161,162 -j REJECT --reject-with icmp-port-unreachable
```

```console
# chmod u+x iptables.sh
# ./iptables.sh
# vim ip6tables.sh
```

```sh
#!/bin/sh

/sbin/ip6tables -F
/sbin/ip6tables -X
/sbin/ip6tables -P INPUT ACCEPT
/sbin/ip6tables -P OUTPUT ACCEPT
/sbin/ip6tables -P FORWARD ACCEPT

/sbin/ip6tables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
/sbin/ip6tables -A INPUT -p icmp -j ACCEPT
/sbin/ip6tables -A INPUT -i lo -j ACCEPT

/sbin/ip6tables -A INPUT -m tcp -p tcp -m state --state NEW --dport 53 -j ACCEPT
/sbin/ip6tables -A INPUT -m udp -p udp -m state --state NEW --dport 53 -j ACCEPT
/sbin/ip6tables -A INPUT -p tcp -m state --state NEW -m multiport --dports 22,25,587,993,80,443 -j ACCEPT
#/sbin/ip6tables -A INPUT -p tcp -m state --state NEW -m multiport --dports 22,25,587,993,80,443,6543 -j ACCEPT
/sbin/ip6tables -A INPUT -j REJECT --reject-with icmp6-port-unreachable
/sbin/ip6tables -A FORWARD -j REJECT --reject-with icmp6-port-unreachable

# SNMP blocking
/sbin/ip6tables -A OUTPUT -m udp -p udp -m multiport --dports 161,162 -j REJECT --reject-with icmp6-port-unreachable
```

```console
# chmod u+x ip6tables.sh
# ./ip6tables.sh
```

設定内容の確認はこの辺ですね。

```console
# iptables -L -n
# ip6tables -L -n
```

次に、この設定を永続化、つまりサーバーが再起動しても有効なままになるように、 `iptables-persistent` をインストールします。

```console
# apt update
# apt -y upgrade
# apt -y install iptables-persistent
```

設定を保存するか聞いてくるので yes を選んであげましょう。保存内容は `/etc/iptables/rules.v4` および `/etc/iptables/rules.v6` に保存されます。

あとで設定を変えたい場合は、最初に書いたシェルを修正して呼び直し (つまり `iptables` コマンドや `ip6tables` コマンドを呼びまくる)、それで変えた設定を、 `service netfilter-persistent save` コマンドで保存してあげれば ok 。
