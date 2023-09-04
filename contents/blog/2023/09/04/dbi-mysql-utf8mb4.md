[$author: T.MURACHI]
# MariaDB 10.6 以降の環境で Perl から UTF-8 文字列を扱う
## 結論
`mysql_enable_utf8mb4` を使え。

## 経緯
Trac が死屍累々なので (鯖移行時にデータアップグレードした Ideanote プロジェクト以外、何故かアップグレードできずアクセスできない状態)、 Wiki に書いた内容だけでも救済すべく、 Perl でスクリプトを書いて DB から直接データをぶっこ抜くことにした。

## スクリプト

こんな感じになった。

```perl
#!/usr/bin/perl
use strict;
use warnings;
use utf8;
use DBI;
use YAML qw/Dump/;
use Encode qw/encode/;

my $db_name = shift @ARGV or die "引数に DB 名を指定してくださいー";

my $dbh = DBI->connect("dbi:mysql:database=$db_name;host=localhost", 'trac_master', '3Bd98Sbc9',
        { mysql_enable_utf8mb4 => 1 })
        or die "$db_name に接続できませんー";

my $sth = $dbh->prepare(<<ENDLINE);
select A.*
from wiki as A,
    (select name, max(version) as version
     from wiki
     group by name) as B
where A.name = B.name
    and A.version = B.version
order by A.name
ENDLINE
$sth->execute;

my @wiki_data;
push @wiki_data, $_ while ($_ = $sth->fetchrow_hashref);
$_->{text} =~ s/\r//g for @wiki_data;

print encode('UTF-8', Dump \@wiki_data);

__END__
```

当初、 `DBI->connect()` の引数に渡すオプションで `mysql_enable_utf8` を渡そうとしていて、以下のようなエラーメッセージに見舞われていました。

```
Character set 'utf8' is not a compiled character set and is not specified in the '/usr/share/mysql/charsets/Index.xml' file
DBI connect('database=trac_ideanote;host=localhost','trac_master',...) failed: Can't initialize character set utf8 (path: /usr/share/mysql/charsets/) at ./loadwiki.pl line 11.
```

これは使っている DB が MariaDB 10.6 だからで、キャラクターセットとして `utf8` が廃止されちゃったからです。 (この情報自体はネット上でもよく見かける)

で、これの解消方法が当初よくわからず、 `SET NAMES 'utf8'` してあげればいいのかしらと以下を試してみたり、

```perl
my $dbh = DBI->connect("dbi:mysql:database=$db_name;host=localhost", 'trac_master', '3Bd98Sbc9',
        { mysql_enable_utf8 => 1, on_connect_do => [ "set names 'utf8'", "set character set 'utf8'" ] })
        or die "$db_name に接続できませんー";
```

`SET NAMES` に渡すキャラクターセットを `utf8mb3` や `utf8_bin` に変えてみたりを試していたのですが解消されず。 `mysql_enable_utf8` の指定自体をやめればいいのかと外してみたら今度は日本語の部分が全部 `?` に置き換わった状態で抽出されるという不具合に見舞われる始末。

で、原点に当たれとばかりに [DBD::mysql のドキュメント](https://metacpan.org/pod/DBD::mysql)を見てみたところ、 `mysql_enable_utf8mb4` というオプションを発見。これに差し替えてみたらあっさりデータが取れるようになったという次第。

鯖移行時に Ideanote の Trac データアップグレードに際してテーブルのキャラクターセットを `utf8_bin` に変更したりしていたことから色々と自分の中で勘違いがあったっぽいんだけど、結局 MariaDB 自体のデフォルトキャラクターセットが `utf8mb4` だったというオチのようで、最初っからそうしとけというだけの話だったのでした。

以上、備忘録として。

!# つーかもう、 MySQL 使うのやめようかな… ('A!`)=3
