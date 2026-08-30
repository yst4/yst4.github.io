+++
title = "学習ログ 26-08-27 joblibと機械学習"
date = 2026-08-27
# updated = 2026-08-11
description = ""
# if you write to post, please comment out the below draft line.
# draft = true
# path = ""

[taxonomies]
categories = ["learn"]
tags = ["learn-ML"]

[extra]
mermaid = false
math = false
+++

今日も引き続きAAAMLPの写経リファクタリングをやってるのですが、chapter 4に入って、プロジェクト構成のところを始めてます。
<!-- more -->

## :book: AAAMLP 3-4章
和書名は :book: kaggleGrandmasterに学ぶ機械学習実践アプローチ ですが、やや古い本で写経はリファクタリングをしながらやっております。

- [https://github.com/yst4/aaamlp_exp/](https://github.com/yst4/aaamlp_exp/)

写経をするうえでuvでプロジェクト管理をしながらgithubで管理をしてるんですけど、サブプロジェクトを使うと、いくつかややこしい所出てきますね。一応、レポジトリに英語の文章を残してあります。

４章に入ると、この本の真骨頂の一つだと思うけど、プロジェクト管理の基礎をやってるんですね。ここでjoblibというライブラリを使うようになったんですが、機械学習やデータサイエンスではよく使われる並列処理とモデルのバイナリ保存を活用できるものらしいです。

この章の最初のtrain.pyを実行したときに重いなぁ。と思ってて、Geminiにコードはシングルプロセッシングで行ってるみたいやって教えたら、joblibは並列処理できるから変えられそうやなと言ってたら、方法を教えてもらって試した。`run(fold=数字)`は本にあった見本コード`chap04/train.py`ですね。

```python
if __name__ == "__main__":
    # run(fold=0)
    # run(fold=1)
    # run(fold=2)
    # run(fold=3)
    # run(fold=4)
    joblib.Parallel(n_jobs=-1)(joblib.delayed(run)(fold) for fold in range(5))
```

これをmainのところに加えて、runを５つ同時に動かすようにした。シングルだけでやってる時より格段に時間が短くなって、この方法知ってないと色んな試行するのに差が出るんやろなぁ。と思ったな。そのくらい差がてきめんに違ったですね。この本でjoblib.Parallelをどこらへんから活用してるのかは知らないんですが。

写経しててリファクタリングしてて手間がかかるけどよいと思ってるのは、こういう知恵が実感できることやプロジェクト管理の知恵も実戦で学べるところなんですよね。レポジトリに落とすときに結構試行錯誤してるんですがね。こういったハンズオン本で写経するって意味を感じますね。考え方の筋が良いと感じたから、時代遅れになったこの本をリファクタリング兼ねてやってるんですがね。
