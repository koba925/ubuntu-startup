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

