+++
title = "学習ログ 26-08-18"
date = 2026-08-18
# updated = 2026-08-11
description = ""
# if you write to post, please comment out the below draft line.
#draft = true
# path = ""

[taxonomies]
categories = ["learn"]
tags = ["kaggle"]

[extra]
mermaid = false
math = false
+++
今日の学習ログ　内容としては、`AAALMP`や`戦略的データサイエンス入門`を買ったよって話になるけど、すこしお金の勉強もあって`ウォール街のランダムウォーカー 14 版`をも読んでるって話になる。
<!-- more -->
## :book: AAALMP 和書 (Kaggle Grandmasterに学ぶ 機械学習 実践アプローチ) について
AAALMPって古い本なんですが、俗称がついてて4GM本というらしいです。今から5年前の本なのかな、やや古くってサンプルプログラムを動かそうと思うとライブラリの仕様変更にぶち当たるところがあったりして、そのへんは対話型AIと相談して解決させたりして読めなないことはない本です。

- :book: [abhishek thakurさんがAAAMLP本用に公開してるgithub](https://gist.github.com/rxaviers/7360908) ここに`AAAMPL.pdf`がある。
- :book: [和書のマイナビブックの公式紹介ページ](https://book.mynavi.jp/ec/products/detail/id=123641)

内容が古いといろいろ言われるかもしれないけど、この方のProject管理方法は優れてるなぁ。と読んでて感心してた。多くの本を読んでてもkaggleなどのデータサイエンス系のProject管理についてまで丁寧に書かれてるのはあまりないんじゃないかな？色んな本を見てないので断言はできませんが。

### :point_up: 責務の分離が徹底的にされている

こんな感じで基本的なProject管理が行なわれてるんですが、config.pyでファイルパス指定だとかmodel.pyでモデルの辞書作成していたりてる。
```
├── input
│ ├── train.csv
│ └── test.csv
├── src
│ ├── create_folds.py
│ ├── train.py
│ ├── inference.py
│ ├── models.py
│ ├── config.py
│ └── model_dispatcher.py
├── models
│ ├── model_rf.bin
│ └── model_et.bin
├── notebooks
│ ├── exploration.ipynb
│ └── check_data.ipynb
├── README.md
└── LICENSE
```
何がいいかと言うと、これでterminalのcli(コマンドライン)を前提で作られてて、シェルスクリプトでフラグを変えてモデルや試行回数を変更させて結果を確認できるというところなんですよね。色んなモデルを作ったりアンサンブルをさせたり。。。膨大なtestをするときにmodels.pyの記述を変えてシェルスクリプトをすこし変更するだけといった、乱雑になりがちな実験が見通しが良くて扱える点にあるんですよね。

僕の場合uvでProject管理を考えたときに一番問題なるのは、inputsなど読み込みのファイルパスなんですよね。例えば、この本の例なら
```
# config.py
TRAINING_FILE = "../input/mnist_train_folds.csv"
MODEL_OUTPUT = "../models/"
```
となってるんだけど、Anacondaを前提でやるとこうなるのだと思うんですが、uv でProject管理をする場合変わるんですよね。

#### :warning: uvプロジェクトでの扱い方注意なところ

たぶんkaggleにアップロードする際にも問題は出るだろうなと言うことで、試してはいないけど、geminiに相談したらこういった方法をすすめてきましたね。未確認と書いておきますが示しておきます。
```
# config.py
import os
from pathlib import Path

# Kaggle環境かどうかの自動判定
IS_KAGGLE = "KAGGLE_KERNEL_RUN_TYPE" in os.environ

if IS_KAGGLE:
    # Kaggle用のパス
    DATA_DIR = Path("/kaggle/input/competition-name")
    MODEL_OUTPUT = Path("/kaggle/working")
else:
    # ローカル（uv環境）用のパス：config.pyの親ディレクトリ（プロジェクトルート）基準
    ROOT_DIR = Path(__file__).resolve().parent.parent
    DATA_DIR = ROOT_DIR / "input"
    MODEL_OUTPUT = ROOT_DIR / "models"

# その他の共通設定
TRAINING_FILE = DATA_DIR / "train_folds.csv"
MODEL_NAME = os.getenv("MODEL", "rf")  # 環境変数やデフォルト値の設定
```
uvでは inputsのパスが、`../input/*.csv` から `./input/*.csv`になるだけなんですが、上記ではpythonのPathライブラリでより丁寧に作っていますね。kaggleの場合は、アップロードしたときに`$KAGGLE_KERNEL_RUN_TYPE`で認識されるようで、それをIS_KAGGLEに割り当てて分岐させてますね。こうしておくとローカルで動かすときは

`uv run python src/train.py`

でパスは崩れない。

これだったら、独自のヘルパー関数を用意しても `./src/utils.py`ならば他のsrcのpyファイルと相対的に同じディレクトリなので、`from utils import ...`で呼び出せるので、責務の分離は明確にできますね。

色んな試行錯誤しても乱雑さを最小限にする工夫がAAAMLPでかなりされてるのを見て、４つのグランドマスターを取った人の知恵なんだなぁ。と感じますね。とかく構造が美しいです。

###  :point_up: 「実験の再現性」と「並列試行」に最適化されている

このProject管理ですごくよい点ですよね。
この本ではシェルスクリプトを
```
#!/bin/sh
# filename: run.sh
python train.py --fold 0
python train.py --fold 1
python train.py --fold 2
python train.py --fold 3
python train.py --fold 4
```
と書いてるけど、uvでのProjectなら
```
#!/bin/sh
# filename: $PROJECT/run.sh
uv run python src/train.py --fold 0
uv run python src/train.py --fold 1
uv run python src/train.py --fold 2
uv run python src/train.py --fold 3
uv run python src/train.py --fold 4
```
と書いておけばいいです。

色んな試行錯誤する段階になって示されてるこの手法のありがたみ出てきますよ。

## :book: 戦略的データサイエンス入門を買った。
オライリーから出てる戦略的データサイエンス入門をかいました。不幸にも紙本が在庫切れ状態だったので中古本買っておきました。新品同様の本が手に入れられてよかった。

当初はデータサイエンス設計マニュアルを買おうと思ったんだけど、状況を考えて優先を変えました。

- :book: [戦略的データサイエンス入門　O`REILY JAPAN公式ページ](https://www.oreilly.co.jp/books/9784873116853/)
- :book: [データサイエンス設計マニュアル　O`REILY JAPAN公式ページ](https://www.oreilly.co.jp/books/9784873118918/)

この２つの本はデータサイエンスとしてのセンスを養うための知恵みたいなものを知るにはいいなぁ。と判断したからです。多くの本はライブラリをどう扱うか？か理論的な説明なのですが、実際に扱う時の注意となると難しいんですよね。

そういうのはもっと現場目線での注意点がある本が一番いいってこと。考え方の本というのは骨格になるだけじゃなくてoutdateになりにくいというのはあります。

購入まで中身は確認できないことと設計マニュアルをとの違いなんて分かりづらいのですけど、目次からももっと現場よりでビジネスパーソンを意識してる感じあります。設計マニュアルは大学院に進むにはとか仕事を手に入れようとか、これからやる人を意識してるように見えるので、読んでおいてもいいなと思っていずれ手に入れると思う。本ばかり買っても積ん読にしかならないので、いまは手一杯ですね。

副読本という位置づけで扱ってますね。副読本と言っても内容は濃いめな印象を持ちましたよ。

## :book: ウォール街のランダムウォーカー 13版
これは別なんですけどね。別に株操作の機械学習は意識してないので。:smiley: この本と敗者のゲームは読んどかないとな。と以前から思ってて数年前に後者は読んだんですがね。
いまは最初の方だから、　ファンダメンタル派と砂上の楼閣派の話、バブルの歴史みたいな中身ですね。ファンダメンタル派、本質的な価値を見出すという考えと、価値なんて心理的な要素みたいな違いですね。堅物とマーケターやせどり業者みたいな違いのような感覚ですね。

そして、バブルの歴史は、オランダの１７世紀の[チューリップ・バブル](https://ja.wikipedia.org/wiki/%E3%83%81%E3%83%A5%E3%83%BC%E3%83%AA%E3%83%83%E3%83%97%E3%83%BB%E3%83%90%E3%83%96%E3%83%AB)、イギリスの１８世紀の[南海泡沫事件](https://ja.wikipedia.org/wiki/%E5%8D%97%E6%B5%B7%E6%B3%A1%E6%B2%AB%E4%BA%8B%E4%BB%B6)のバブルって所。いつの時代でも実態のない資産価値に踊らされるみたいな話が出てて、人の愚かさの歴史の回り方みたいなことかな。あのアイザック・ニュートンですら南海会社のバブルでかなり被害を受けたというのも書かれてて、「天体の動きは計算できるが、人々の狂気は計算できない」という言葉を残したらしい。

## さいごに
なかなかPython機械学習プログラミング[PyTorch&scikit-learn編] 　Pythonによる時系列予測 (Compass Data Science)を触れるの遠い。チョット読んでるんですけどね。
