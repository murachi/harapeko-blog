[$author: T.MURACHI]
# Ubuntu 22.04 でスクリーンセーバーを設定する
## 設定方法
1. xscreensaver とか必要なものをインストールする。
   ```sh
   $ sudo apt update
   $ sudo apt -y upgrade  # 必要に応じて
   $ sudo apt install xscreensaver xscreensaver-gl-extra xscreensaver-data-extra
   ```
1. xscreensaver デーモンが自動起動するよう設定する。
   具体的には、 `/etc/xdg/autostart` ディレクトリ下に `screensaver.desktop` のようなファイルを作成する。
   ```sh
   $ sudo vim /etc/xdg/autostart/screensaver.desktop
   ```
   内容は以下の通り。 `Exec` に指定しているコマンドのオプション `-no-splash` は、デーモン起動時に通常表示されるスプラッシュイメージを表示しないようにするためのもの。
   ```ini
   [Desktop Entry]
   Name=Screensaver
   Type=Application
   Exec=xscreensaver -no-splash
   ```
1. あとはアプリの検索で `xscreensaver` と入力しかけると表示される「スクリーンセーバー」をクリックするか、 `xscreensaver-demo &` コマンドを実行してあげれば、 xscreensaver の設定ダイアログが表示されるので、そこでよしなに好きなスクリーンセーバーを選んでモロモロ設定してあげればおｋ。

## 参考リンク
- [Ubuntuにスクリーンセーバーをインストールまたは変更する方法は？](https://cloudo3.com/ja/how-to/ubuntu%E3%81%AB%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%BB%E3%83%BC%E3%83%8F%E3%82%99%E3%83%BC%E3%82%92%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%81%BE%E3%81%9F%E3%81%AF%E5%A4%89%E6%9B%B4%E3%81%99%E3%82%8B%E6%96%B9%E6%B3%95%E3%81%AF%EF%BC%9F/77705025)
  - `gnome-screensaver` を削除する手順が書かれているが、 22.04 ではデフォルトでインストールされないようなので不要っぽい。
- [How do I set Xscreensaver to autostart? - Ask Ubuntu](https://askubuntu.com/questions/128481/how-do-i-set-xscreensaver-to-autostart)
  - Ubuntu 起動時にデーモンが自動起動するようにするための設定方法がここに記載されていました。
