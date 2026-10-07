+++
title = "WSLCのセッション保存先を別ドライブにする"
date = 2026-10-07
# updated = 2026-08-11
#description = ""
# if you write to post, please comment out the below draft line.
# draft = true
path = "/blog/how-to-use/wslc-save-on-d-drive"

[taxonomies]
categories = ["how-to-use"]
tags = ["wslc", "wsl", "docker"]

[extra]
mermaid = false
math = false
+++
WSLCというのはdockerと互換性があるコンテナを作成して実行できるものですが、ぼく個人では今まで必要を感じてなかったです。しかし、wslが3.0が登場して、あらたにdocker互換のwslcが標準で付属するようになったので試してた。そこでコンテナの保存をCドライブからDに変えた。今はまだ情報もないのでメモを公開しておく。
<!-- more -->

## あらすじ
WSLはWindowsでubuntuなどlinuxを仮想OSとして利用するのに必要な仕組みです。多くの人達がdockerを使ってるのは知ってたけど、メモリもストレージも食うしやらなかった。[ :link: 今回標準付属の互換のWSL Containersが含まれるようになった](https://learn.microsoft.com/ja-jp/windows/wsl/wsl-container?tabs=csharp)ので触るようにしてみた。これはグループで開発するときとか使い捨てOSを試そうとするにはいいんだなと思ったですね。でも、ストレージは結構食うのはOS丸ごと抱えることからもわかるようにデフォルトのままでは使いたくないな。

でも、調べても余り情報がない。。。正式付属になって1週間ばかりなんで当然なんですけど。一応解決したんで書いときます。

## 試してみる。
これは　基本的に先にリンクつけておいた。microsoftのドキュメントで見ればいいですけど。使い方はpower shell上で行う。書いてるように動作は確認しました。でも、気になったのは保存先がCドライブなんですよね。大きなファイルも多くなりがちなのも分かってるだけにDドラブでやりたいと思いました。

## 情報の確認
確認の仕方はヘルプでだいたい分かるんだけど次の通り。
```
> wslc system info
クライアント:
WSL バージョン: 3.0.1.0
カーネル バージョン: 6.18.40.1-1
Direct3D バージョン: 1.611.1-81528511
DXCore バージョン: 10.0.26100.1-240331-1435.ge-release
Windows バージョン: 10.0.26300.9457
設定ファイル: C:\Users\****\AppData\Local\wslc\settings.yaml

サーバー:
セッション マネージャーのバージョン: 3.0.1
セッション: 1
ID   作成者 PID   表示名
1    24216     wslc-cli-****
> 
```
設定ファイルはこうやって保存先がでますが、便利なものでつぎのようにすれば設定ファイルをエディタで開くようにできます。
```
wslc settings
```

### stragePathの変更

これで中身を見て書き換えられます。注目したのは
```
session:
 (...)
 # storagePath: default
```
これです。ここを書き換えてやるんです。僕の場合は `D:\VHDX` にしたいんですね。実はwslのubuntuのvhdxファイルはここに移しているのでセッションもこちらにおいておくことにしてあります。

```
 # storagePath: default
 storagePath: "D:\\VHDX"
```

このように設定を書いておけばいいです。こうすると、セッションの保存先が `D:\VHDX\wslc\sessions\...` というところに保存されるようになります。

これでおしまいと思いましたが、ここにトラップがあります。

### :warning: 設定が反映されないトラップとその回避
設定を書き換えただけでは、バックグラウンドで既存のセッション（Cドライブ側）が保持され続けるため、Dドライブ側への新規作成に切り替わりませんよ。
wslを終わらせないといけないんですね。あと確実にwslcのプロセスもkillしておく必要もあります。僕自身はつぎのようにpowershellで入力して解決したけど、再起動でもよいと思います。

```
> wsl --shutdown
> Stop-Process -Name "wslc*" -Force -ErrorAction SilentlyContinue
```

こうして確実に終わらせました。それでwslcをお試ししてみます。
```
> wslc run --rm alpine hostname
```
wsl: session.storagePath で構成されたセッション ストレージを 'D:\VHDX\wslc\sessions\wslc-cli-****' に作成しています。後でこの設定を変更または削除すると、ここに格納されているデータは既定のセッションでは使用されなくなるので、削除して領域を解放することができます。

イメージ 'alpine' が見つかりません。プルしています 
(...)
```
このような表示が出れば、セッションのセーブ先がDドライブに移ったことがわかります。最後にディレクトリ `%LOCALAPPDATA%\wslc\sessions\<session>\storage.vhdx` という残骸がありますから消しておけばいい。ドライブ変更前にwslcを動かしていたら残ります。

これでOKです。
## おまけ：動作確認例（Jupyter DataScience Notebook）
ポートバインド（-p）とホストディレクトリのマウント（-v）を指定してjupyterを起動。詳しくは[:link: jupyter/datascience-notebook (docker hub)](https://hub.docker.com/r/jupyter/datascience-notebook)
```

> wslc run --rm -p 8888:8888 -v ~/project:/home/jovyan/work jupyter/datascience-notebook
```

かなぁ。これでよく使われてるデータサイエンス向けのdocker image?が動かせます。

```
> wslc run -it -v ~/project:/home/jovyan/work jupyter/datascience-notebook bash
```
これでシェルを動かしてみて。ipythonとか etc/os-releaseとか見てみた。ただ使い捨てOSのまんまなので、データセーブ先は `~/project`を適当に変えてください。
