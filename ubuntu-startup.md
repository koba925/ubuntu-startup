# UbuntuをMacbook Airに入れてみる

ちょっと待てよ
最近、MacBookAirあんまり触ってないな
もしかしてなくなっても困らない？

・・・Ubuntu入れてみようか（ごくり

4GB/128GBの最弱MBAでも、まるごとUbuntuにするなら十分な容量のはず
欲張らなければ仮想マシンもそこそこ使えたりして？
仮想マシンがちょっと重かったとしてもDockerならいけるだろうし
いざとなればTimeMachineで元に戻るはずだしきっとたぶん

参考文献を入手

Ubuntuスタートアップバイブル

この本では、Ubuntu日本語Remixというのを使う
[Ubuntuの日本語環境 | Ubuntu Japanese Team](http://www.ubuntulinux.jp/japanese)

素の状態からやらないとトラブルの時に困ったりすることがあるけど
これはきっといい感じにしてくれてると信じて素直にこれを使おう
日本語関連で手間取るのはもったいないし

まずはライブUSBを作って起動させる
この本は（ほぼ当然ながら）PC前提だけどMacではどこまで同じでいけるんだろう

Ubuntu日本語Remixのisoをダウンロード
めっちゃ時間かかるので他の準備を進めたり本を読んだりMBAの事例を調べたり
デュアルブートにしてる例が多いな
ちょこちょこ何かあるみたいな感じもあるけどすんなり入ってることもあるみたい

[Rufus](https://rufus.ie/)をダウンロードして実行
ダウンロードしたisoを指定して書き込み
そんなややこしい設定はなくてすんなり完了

MBAに挿してOption押しながら再起動してEFI Bootを選択
Try ... とか読んでる間にライブセッションが起動してしまった
普通に動いてる
Wi-Fiの設定したらネットにもつながるのかな

> Wi-Fiアダプターが見つかりません

むむ
これ、実際にインストールするときには見つけてくれるんだろうか
突き進む

ライブセッションのデスクトップにもUbuntuをインストールするっていう
アイコンがあるのでここから実行
キーボードの選択 日本語-日本語(Macintosh)

なお出来心で「キーボードレイアウトを検出」したらヘブライ語になりました
すみません

アプリケーションは通常（最小にしてみたい気もしたけど）
アップデートのダウンロードはチェックされててグレーで触れない
グラフィックスと～サードパーティ製ソフトウェア、はインストール

ディスクを削除してUbuntuをインストール
お願い（祈

> GRUBのインストールに失敗しました

そうですか

> 'grub-efi-amd64-signed' パッケージを /target/ にインストールするのに失敗しました。GRUBブートローダなしでは、インストールしたシステムは起動しません。

OKじゃないけどOKな

> インストーラーがクラッシュしました
> インストーラーがクラッシュしました
> 申し訳ありません、インストーラーがクラッシュしました。このウィンドウを閉じた後で、バグ報告ツールを使って・・・

> 問題を報告することができません。

ネットにつながってないもんな

再起動してライブセッションに入らずにやってみよう
起動画面からMacOSがいなくなってる

そういえばここでWi-Fi選んでおけるんだな
ここで選んでおくとライブセッションでWi-Fi使えたりする？
関係ない気もするけど ネットワークブート用とかだろうな
でもちょっとやってみよ
はいだめでした

気を取り直してEFI Bootして「Install Ubuntu」
ほとんど同じ
Ubuntu 18.04を消しますか、同居しますか、って聞かれたくらい
消さずにそのまま使ってくれてもいいのよ（たぶん

・・・
同じだ

調べるといくつか出てくる

* rEFIt(rEFInd)を使う

これをつかっておけばよかったらしいが、Mac OSで動かすものらしい
手遅れ

* FAT32のパーティションを作る

作ってなかったっけ？ESPとかいう名前で
partedで見てみた

```console
1 1049kb 538MB 537MB fat32 EFI System Parition  boot, esp
2 538MB  121GB 121GB ext
```

ほら

* ネットにつながないでインストールする

つながってたの？

* がんばる

[Ubuntu + Mac: Pure EFI Boot - The Slightly Disgruntled Scientist](https://heeris.id.au/2014/ubuntu-plus-mac-pure-efi-boot/#fixing-efi)を見てがんばる

細かいところの意味はわかってないけど要するにEFIパーティションをFAT32からHFS+に変えてgrubをインストールしなおすということらしい

なおうるさい起動音を止められるのはMac OSからしかできないから
先に止めておけって書いてあるけど手遅れ

さてやってみよう

メニューが出たところでcを押してgrubのシェルに入る
lsでhomeの場所を確認 → ls (hd2,gpt2)/home
lsで/boot/grubの場所を確認 → ls (hd2,gpt2)/boot/grub
rootを設定 → set root=(hd2,gpt2)
UUIDを表示 → ls -l (hd2,gpt2)
linux /boot/vmlinuz<Tabキー> .efi.signed root=UUID=<上のUUID>
initrd /boot/initrd<Tabキー>
boot

おーブートした
ログイン
不完全な言語サポートって言ってるけど今は無視

とはいうもののここからaptを使うのでネットつながってないと
やっぱりWi-Fiアダプターがないって言われる
ハードウェア的にはあるからドライバがないのかな

```console
$ lspci -nn -d 14e4:
03:00.0 Network controller [0280]: Broadcom Inc. and subsidiaries BSM4360 801.11ac Wireless Network Adapter [14e4:43e0] (rev 03)
```

これは、有線でつなぐしかないか？
USBのアダプタでも買ってくるかなあ？

それっぽいドライバがUSBメモリに入っているという情報を発見

/media/takahiro/UBUNTU1804/pool/を見てみる
ほんとだ

/main/d/dkms/dkms_2.3ubuntu9.2_all.deb
/restricted/b/bcmwl-kernel-source_6.30.223.271+bdcom-0ubuntu4_amd64.deb

このへん

`sudo dpkg -i`してみると依存関係が不足しててインストールできない
だめかとおもったけど、探してみると依存関係のパッケージも入ってる
いっぱいあってわけわからなくなったけどとにかく依存関係を追いかけて必要なものを全部インストール
一発でできるコマンドとかありそうな気がするけどわからなかった

あと、b43-fwcutterとfakerootってやつも使うって記事があった気がするんだけどこれも入ってたので入れておく

結局全部入れることができた
どういうことだろう
ここまでしてくれたなら自動で入れてくれてもよさそうなものだけど
たぶん見落としてるだけで簡単にできるやり方があったにちがいない

そして設定を見てみると・・・Wi-Fi見えてた！つながった！

続き

```console
$ sudo add-apt-repository ppa:detly/mactel-utils
```

Releaseファイルがなくて更新が無効、って出たけどいいのかな

```console
$ sudo apt update
$ sudo apt install mactel-boot hfsprogs gdisk grub-efi-amd64
```

mactel-bootが見つからないって
うーんやっぱりだめかな

ソースらしき [Intel Mac Utilities : Jason Heeris](https://launchpad.net/~detly/+archive/ubuntu/mactel-utils) 見てみたら
bionicないじゃん
それじゃダメなのでは

apt-repositoryの情報は/etc/apt/sources.list.d/の下に保存されているらしい
detly-ubuntu-mactel-utils-bionic.listっていうファイルがあった
bionicと書いてあるところをxenialに書き換えたらupdateできた

・・・あとは祈るしかないな

apt installも成功

続きは書いてある通りでいけたんだけれども
全部終わって再起動したらgrubのシェルで止まってしまった
何かが足りない

もう一度、さっきやった手順でなんとかUbuntuにログインした後、
`update-grub`したら直った模様（若干自信なし）

```console
# update-grub
Sourcing file `/etc/default/grub'
Generating grub configuration file ...
Linux イメージを見つけました: /boot/vmlinuz-4.18.0-17-generic
Found initrd image: /boot/initrd.img-4.18.0-17-generic
Found Mac OS X on /dev/sda1
```

さて

> 不完全な言語サポートって言ってるけど今は無視

インストール中にネットにつながってなかったせいだろう
`sudo apt update`と`sudo apt upgrade`をすませば
解消するのでは

途中でナショナル配置とラテン配置の切り替えどうする？って聞かれた
日本語入力の切り替えのことかな？
Capslockにしてみたけど何も変わってない気がする
そのあたりは明日やろう

## UbuntuをMacbook Airに入れてみる (2)

気になる点はいくつか

Dockの最上部に"RELEASEのインストール"というアイコンがある
検索してみると、Ubuntuのインストールの残骸らしい
実行したら起動しなくなったという報告もあった
（自分は実行してしまったけどなんともなかった・・・ように見える）
Dockのアイコンは消してしまってよいらしい
実態はubiquityというパッケージらしいのでこれも消す

日本語入力がいい感じに設定できてない
入力ソースの選択が「日本語」「日本語(Mozc)」「日本語(Macintosh)」と
なにか重複感
インストールの時「日本語(Macintosh)」を選んだけど「日本語」のほうが
よかったのかな？わからない

Vagrantでデスクトップ環境を立ち上げたときとはちょっと違う感じ
Macだからなのか日本語Remixだからなのか
なによりまず日本語入力にするとカナ入力なのが困る
プロパティを見るとローマ字入力設定なのに
「日本語(Macintosh)」ってやつが悪さをしているっぽい、というか
「日本語(Macintosh)」はカナ入力キーボードってことなのかもしれない
よくわからない

ここから下は試行錯誤した後で書いてるので、順序が前後してるところが
あると思う

「日本語（Macintosh）」は「設定」「地域と言語」から消した

さてキーボード
Vagrantのときを思い出して
sudo dpkg-reconfigure keyboard-configuration を実行

Apple
Apple Aluminum(ANSI)
Apple Aluminum(ISO)
Apple Aluminum(JIS)
Apple laptop

とある
laptopよりもJISだろうな

次は
日本語 - 日本語（Macintosh）
かなあ
日本語（Mozc）ってのはない

ナショナル／ラテンの切り替えはなしでいいか
一時切り替えもなし

AltGrは「キーボード設定のデフォルト」にしておく
どのキーなのかはわからない

コンポーズキーもなし

Xサーバを強制終了するのにControl+Alt+Backspaceは使わない

いっぱい選択あったな
そしてリブート

うん
英数でローマ字入力、かなで直接入力になったよ！
って反対やん！
いやちがうか
英数でローマ字入力だけど、かなはトグルだ
まあとにかく設定設定

入力ソースのメニューで「ツール」「プロパティ」
「キー設定の選択」を「編集」
「入力キー」のところで2回ダブルクリックすると、「割り当てるキーの入力」と
いうのが出てきて、ここでキーを押すとそのキーが割り当てられる模様
で「かな」キーを押すと「Eisu」、「英数」キーを押すと「Hiragana」になると
いうことが判明
キーボードレイアウトの表示で見てみても、「かな」を押すと「英数」の位置が、
「英数」をおすと「かな」の位置が光るので、どこかで反対にされてしまってるようだ

わかってれば問題ない
「IMEを有効化」になっているところを「Eisu」にしたり
「IMEを無効化」になっているところを「Hiragana」にしたり
「Eisu」「Hiragana」に別の機能が割り当てられてる行を削除したり

とりあえずこれで普通に日本語が入力できるようになってように見える

ついでにほかのキーも試してみる
caps → 「Eisu」
Shift → 「Shift」
Ctrl → 「Ctrl」
Tab → 「Tab」
Esc → 「Escape」
Enter → 「Enter」
delete → 「Backspace」

caps以外は普通だな
capsはもしかしてナショナル／ラテンの切り替えで一度設定した名残？

## UbuntuをMacbook Airに入れてみる (3)

じゃvscodeでも入れようか

```console
$ snap find vscode
Name               Version   Publisher  Notes    Summary
ampareinvertcolor  1.0.0     juthawong  -        Simply Invert CSS Color - Made For Web Designer
code               a622c65b  vscode✓    classic  Code editing. Redefined.
code-insiders      6ac87465  vscode✓    classic  Code editing. Redefined.
$ snap install code
error: too early for operation, device not yet seeded or device model not acknowledged
```

なんかエラー出た
デバイスはまだシードされていません？
なんだろう
何か最初にしなきゃいけないことができてないっぽいな

upgradeしたら何か変わらないかな

```console
$ sudo apt upgrade snapd
snapd はすでに最新バージョン (2.38+18.04) です。
$ snap install code
error: too early for operation, device not yet seeded or device model not acknowledged
```

変わりません

アンインストールしてインストールしたら普通に動くようになるかなあ
でもsnapをアンインストールしたとたんsnapでインストールしたソフトが
消えちゃったりとか、そこまでいかなくても管理不能になったりしないだろうか
今何入ってるの

```console
$ snap list
Name               Version          Rev   Tracking  Publisher   Notes
core               16-2.38          6673  stable    canonical✓  core
gnome-3-26-1604    3.26.0.20190228  82    stable/…  canonical✓  -
gtk-common-themes  0.1-16-g2287c87  1198  stable/…  canonical✓  -
```

消えたりするとダメージあるな
むう

検索してみるといくつか引っかかるけど、あんまりすんなり解決していないようにも見える
調べてみろと書いてあったことを試していく

デバイスとかシードの情報はstate.jsonというファイルに書いてあるようだ

```console
takahiro@lorraine:/var/lib/snapd/seed$ sudo cat /var/lib/snapd/state.json | jq '.data.auth.device["brand","model"],.data.seeded,.data["seed-time"],.data.auth.device.serial'
"generic"
"generic-classic"
null
null
"8994a8e7-3d6b-4d2a-96c1-6191f45aaa2e"
```

確かにシードされていないようだな
でもどうやったらシードされるかはうまく検索できなかった

```console
$ snap changes
ID   Status  Spawn                     Ready  Summary
1    Doing   2 days ago, at 19:35 JST  -      Initialize system state
```

DoneじゃなくてDoingなのが気になる
まだInitialize中で終わってない、とも読める

ジャーナルを見る

```console
$ journalctl -u snapd --full --no-pager
-- Logs begin at Sat 2019-05-18 19:35:29 JST, end at Mon 2019-05-20 21:24:08 JST. --
 5月 18 19:35:33 lorraine systemd[1]: Starting Snappy daemon...
 5月 18 19:35:34 lorraine snapd[752]: AppArmor status: apparmor is enabled and all features are available
 5月 18 19:35:34 lorraine snapd[752]: helpers.go:145: error trying to compare the snap system key: system-key missing on disk
 5月 18 19:35:34 lorraine snapd[752]: daemon.go:379: started snapd/2.38+18.04 (series 16; classic) ubuntu/18.04 (amd64) linux/4.18.0-17-generic.
 5月 18 19:35:34 lorraine systemd[1]: Started Snappy daemon.
 5月 18 19:35:34 lorraine snapd[752]: stateengine.go:102: state ensure error: Get https://api.snapcraft.io/api/v1/snaps/sections: dial tcp: lookup api.snapcraft.io: no such host
 5月 18 19:36:27 lorraine snapd[752]: daemon.go:611: gracefully waiting for running hooks
 5月 18 19:36:27 lorraine snapd[752]: daemon.go:613: done waiting for running hooks
 5月 18 19:36:27 lorraine systemd[1]: snapd.service: Service hold-off time over, scheduling restart.
 5月 18 19:36:27 lorraine systemd[1]: snapd.service: Scheduled restart job, restart counter is at 1.
 5月 18 19:36:27 lorraine systemd[1]: Stopped Snappy daemon.
 5月 18 19:36:27 lorraine systemd[1]: Starting Snappy daemon...
 5月 18 19:36:28 lorraine snapd[1974]: AppArmor status: apparmor is enabled and all features are available
 5月 18 19:36:28 lorraine snapd[1974]: daemon.go:379: started snapd/2.38+18.04 (series 16; classic) ubuntu/18.04 (amd64) linux/4.18.0-17-generic.
 5月 18 19:36:28 lorraine systemd[1]: Started Snappy daemon.
 5月 18 19:36:28 lorraine snapd[1974]: stateengine.go:102: state ensure error: Get https://api.snapcraft.io/api/v1/snaps/sections: dial tcp: lookup api.snapcraft.io: no such host
 5月 18 21:55:43 lorraine snapd[1974]: main.go:147: Exiting on terminated signal.
 5月 18 21:55:43 lorraine systemd[1]: Stopping Snappy daemon...
 5月 18 21:55:43 lorraine systemd[1]: Stopped Snappy daemon.
-- Reboot --

:

-- Reboot --
 5月 19 22:01:41 lorraine systemd[1]: Starting Snappy daemon...
 5月 19 22:01:41 lorraine snapd[791]: AppArmor status: apparmor is enabled and all features are available
 5月 19 22:01:41 lorraine snapd[791]: helpers.go:717: cannot retrieve info for snap "gnome-calculator": cannot find installed snap "gnome-calculator" at revision 406: missing file /snap/gnome-calculator/406/meta/snap.yaml
 5月 19 22:01:41 lorraine snapd[791]: daemon.go:379: started snapd/2.38+18.04 (series 16; classic) ubuntu/18.04 (amd64) linux/4.18.0-20-generic.
 5月 19 22:01:42 lorraine systemd[1]: Started Snappy daemon.
 5月 19 22:01:42 lorraine snapd[791]: stateengine.go:102: state ensure error: Get https://api.snapcraft.io/api/v1/snaps/sections: dial tcp: lookup api.snapcraft.io: no such host
 5月 20 05:44:17 lorraine snapd[791]: api.go:1071: Installing snap "code" revision unset
```

api.snapcraft.ioにアクセスできてない？

```console
$ wget https://api.snapcraft.io/api/v1/snaps/sections
--2019-05-20 21:29:17--  https://api.snapcraft.io/api/v1/snaps/sections
api.snapcraft.io (api.snapcraft.io) をDNSに問いあわせています... 91.189.92.41, 91.189.92.19, 91.189.92.20, ...
api.snapcraft.io (api.snapcraft.io)|91.189.92.41|:443 に接続しています... 接続しました。
HTTP による接続要求を送信しました、応答を待っています... 200 OK
2019-05-20 21:29:18 (30.3 MB/s) - `sections' へ保存完了 [558/558]

$ cat sections
{"_embedded": {"clickindex:sections": [{"name": "featured"}, {"name": "games"}, {"name": "finance"}, {"name": "productivity"}, {"name": "utilities"}, {"name": "news-and-weather"}, {"name": "science"}, {"name": "health-and-fitness"}, {"name": "education"}, {"name": "personalisation"}, {"name": "devices-and-iot"}, {"name": "books-and-reference"}, {"name": "security"}, {"name": "music-and-audio"}, {"name": "social"}, {"name": "server-and-cloud"}, {"name": "development"}, {"name": "entertainment"}, {"name": "photo-and-video"}, {"name": "art-and-design"}]}}
```

できてはいるけど、もしかしたら起動直後でWi-Fiにつながってないタイミングってのはあるかもね？
サービスを再起動してみたらどうかな
まず現状を

```console
$ systemctl | grep snapd
snapd.autoimport.service      loaded inactive   dead      start Auto import assertions from block devices
snapd.seeded.service          loaded activating start     start Wait until snapd is fully seeded
snapd.service                 loaded active     running         Snappy daemon
snapd.socket                  loaded active     running         Socket activation for snappy daemon
```

ここにもシードを待ってる人が
死んでる人も

まずサービスを止める

```console
takahiro@lorraine:/var/lib/snapd/seed$ systemctl stop snapd
Warning: Stopping snapd.service, but it can still be activated by:
  snapd.socket
```

順番が違ったみたい

```console
takahiro@lorraine:/var/lib/snapd/seed$ systemctl stop snapd.socket
takahiro@lorraine:/var/lib/snapd/seed$ systemctl stop snapd
```

どこまで消えたかな

```console
takahiro@lorraine:/var/lib/snapd/seed$ systemctl | grep snapd
● snapd.seeded.service        loaded failed failed    Wait until snapd is fully seeded
```

あなたは残ってましたか

```console
takahiro@lorraine:/var/lib/snapd/seed$ systemctl stop snapd.seeded.service
takahiro@lorraine:/var/lib/snapd/seed$ systemctl | grep snapd
● snapd.seeded.service        loaded failed failed    Wait until snapd is fully seeded
```

消えない

```console
takahiro@lorraine:/var/lib/snapd/seed$ systemctl kill snapd.seeded.service
takahiro@lorraine:/var/lib/snapd/seed$ systemctl | grep snapd
● snapd.seeded.service        loaded failed failed    Wait until snapd is fully seeded
```

消えない
むしろ起動してやるべきなのか

```console
takahiro@lorraine:/var/lib/snapd/seed$ systemctl reload snapd.seeded.service
Failed to reload snapd.seeded.service: Job type reload is not applicable for unit snapd.seeded.service.
See system logs and 'systemctl status snapd.seeded.service' for details.

takahiro@lorraine:/var/lib/snapd/seed$ systemctl status snapd.seeded.service
● snapd.seeded.service - Wait until snapd is fully seeded
   Loaded: loaded (/lib/systemd/system/snapd.seeded.service; enabled; vendor preset: enabled)
   Active: failed (Result: signal) since Mon 2019-05-20 22:13:09 JST; 12min ago
  Process: 891 ExecStart=/usr/bin/snap wait system seed.loaded (code=killed, signal=TERM)
 Main PID: 891 (code=killed, signal=TERM)

 5月 19 22:01:42 lorraine systemd[1]: Starting Wait until snapd is fully seeded...
 5月 20 22:13:09 lorraine systemd[1]: snapd.seeded.service: Main process exited, code=killed, status=15/TERM
 5月 20 22:13:09 lorraine systemd[1]: snapd.seeded.service: Failed with result 'signal'.
 5月 20 22:13:09 lorraine systemd[1]: Stopped Wait until snapd is fully seeded.
takahiro@lorraine:/var/lib/snapd/seed$ systemctl start snapd.seeded.service
```

帰ってこない

```console
^Z
[1]+  停止                  systemctl start snapd.seeded.service
takahiro@lorraine:/var/lib/snapd/seed$ bg
[1]+ systemctl start snapd.seeded.service &

takahiro@lorraine:/var/lib/snapd/seed$ systemctl status snapd.seeded.service
● snapd.seeded.service - Wait until snapd is fully seeded
   Loaded: loaded (/lib/systemd/system/snapd.seeded.service; enabled; vendor preset: enabled)
   Active: activating (start) since Mon 2019-05-20 22:26:39 JST; 2min 50s ago
 Main PID: 5488 (snap)
    Tasks: 11 (limit: 4374)
   CGroup: /system.slice/snapd.seeded.service
           └─5488 /usr/bin/snap wait system seed.loaded

 5月 20 22:26:39 lorraine systemd[1]: Starting Wait until snapd is fully seeded...
```

あくまでもシードを待つ姿勢
こいつをどうすべきかわからないのでいったん再起動
やっぱり同じ

ここはもう勢いあまってsnapdをアンインストールしてインストールか
仮に起動しなくなったとしてもUbuntuインストールしなおすだけだし
ここはremoveするのではなくpurgeするところだろうな

```console
$ sudo apt purge snapd
以下のパッケージは「削除」されます:
  gnome-software-plugin-snap* snapd*
続行しますか? [Y/n] y
$ sudo apt install snapd
以下のパッケージが新たにインストールされます:
  snapd
アップグレード: 0 個、新規インストール: 1 個、削除: 0 個、保留: 0 個。
```

```console
$ snap list
No snaps are installed yet. Try 'snap install hello-world'.
```

やっべなんにもなくなった
removeから試すべきだったか

泥縄でインストール
これは、日本語Remixで入るのと同じものが入るんだろうか
少し心配

```console
$ snap install core
2019-05-20T22:44:45+09:00 INFO Waiting for restart...
core 16-2.38.1 from Canonical✓ installed
$ snap install gnome-3-26-1604
gnome-3-26-1604 3.26.0.20190228 from Canonical✓ installed
$ snap install gtk-common-themes
gtk-common-themes 0.1-16-g2287c87 from Canonical✓ installed
```

そのままvscodeまでインストールしてしまおう

```console
$ snap install code --classic
code a622c65b from Visual Studio Code (vscode✓) installed
$ code
```

起動した！

では再起動
無事立ち上がりますように・・・
立ち上がった！

ここまで
