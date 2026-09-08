+++
title = "学習ログ 26-09-08 AAAMLP, ISLR2"
date = 2026-09-08
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

今日も引き続き`AAAMLP`の写経リファクタリングをやってるのですが、今日は`AAAMLP`5章途中と、R言語をインストールして`ISMR2`をやってるところですね。
<!-- more -->

## :book: AAAMLP 5章
和書名は :book: `kaggleGrandmasterに学ぶ機械学習実践アプローチ` ですが、やや古い本で写経はリファクタリングをしながらやっております。

- [https://github.com/yst4/aaamlp_exp/](https://github.com/yst4/aaamlp_exp/)

順調に写経はしてるんですけど、ただ写経してるだけじゃつまんないし血にも肉にもなりにくいんで、色々手を入れながらGeminiと対話してやってますね。可読性を考えたときにはいくつか注意してるんですけどね。例えば、`xgboost`のパラメータをコマンドラインの引数で変更させられるようにしてたりします。せっかく`argparse`Library使うのを教えてくれてるんだからね。同じように紹介されてた`joblib`も結構使ってますよ。簡単に並列化で高速になるんだもん。

あとうまく`itertools`使ってるのに無駄にリストにして`for`文に持ってきたり。イテレータについて言うと、わざわざリストに直すのはメモリも速度でもデメリットになるだけしかないのでね。チョットそういう洗練されてないところはあるんですが、巷の記事でイテレータをうまく扱ってるものは希少なんでこういったことが残りやすいんでしょうね。

``` python,linenos,
def feature_engineering(df, cat_cols):
    """
    This function is used for feature engineering
    Args:
        df: the pandas dataframe with train/test data
        cat_cols: list of categorical columns
    Results:
        dataframe with new features
    """

    combi = itertools.combinations(cat_cols,2)

    for c1,c2 in combi:
        df[c1 + " " + c2] = df[c1].astype(str) + "_" + df[c2].astype(str)

    return df
```
元の関数は示さないけど、11行目と13行目に当たる部分の`itetools`と`for`文を組み合わせるならこういう使い方になりますね。`range`や`zip`、`enumerate`感覚でいいんですよ。

個人的には`pandas`つかうより`polars`使うほうがパイプラインを組んでいく考えで`rust`や`Haskell`に通じる発想でできるんで読みやすく扱いやすそうだなと思ってますね。いずれはスイッチすると思います。知らん世界だから多くの情報がある`pandas`を優先してるだけなので。`AAAMLP`を`polars`でやるってのも悪くないんですが。

### `df[:,"foo"] = ...` なんて

```
df.loc[:,"foo"] = ... 
# →
df["foo"] = ...
```
と変えることは多いですね。可読性も上がりますしね。これも実は注意が必要な場面もあるんですが、これは意図をGeminiに聞いてみると

> `df.loc[:, col] = ...` が使われる主な理由は、`df[col]` が元の DataFrame の参照（ビュー）かコピーかが曖昧なときに発生する警告を防ぐためです。

と言ってるんですね。曖昧な扱いのときは注意が必要ってことですね。

### AUCの考え方
評価基準になるAUCなんですけど、数字で気になることがあるんですよね。本が出た当時と利用Libraryも内部変更もあるし作り変えも影響あるんで、数値の読み方って注意必要なんですよね。そこをGeminiに聞いてるとね。

> データサイエンスの現場では、以下のように判断するのが一般的です。
>
> - AUC 差が 0.01 以上: 明確な手法の差（特徴量の追加やアルゴリズム変更の効果あり）
>
> - AUC 差が 0.001 〜 0.005 程度: 単なる乱数・ライブラリのバージョン・CPUの浮動小数点計算の差（実質「同等」とみなす）

だいたいこういう感覚で見るようなんで、それを考えると十分想定内だから気持ち悪さはなくなったかな。

## :book: ISLR2 とR言語
統計的なこともすこし頭の中整理しないとなと思ってね。定評がある教科書 `An Introduction to Statistical Learning` が教育敵配慮？で無料公開されてるので使わない手がないなというので、活用してます。当初は`ISLP`を読んでたけど、`Python`写経をやってると時代の変化があってね。ならばすこし枯れてると行っていいRのほうがええやん。となりました。

- :book: https://www.statlearning.com/

### R言語
R言語といっても大昔にすこし触った経験しかなくて、その頃から`scheme`の親戚（S式ではないけど思想的にlispなんですよ！）とオブジェクト指向は`C++`/`Java`のようなクソ仕様（嫌いなんで。）じゃなくて`Common Lisp`のオブジェクト指向に当たる`CLOS`のような柔軟なOOPという感覚だったんですよね。

スタイルの変化をGeminiに聞いてみるとPipelineを意識して、どちらかというと`Hakell`のような雰囲気に変わったなぁ。と思ったかな。`tidyverse`というライブラリ群がスタイルに影響を与えたようで、モダン`R`はかなりコードの見た目変わったなと感じました。2010年代後の流れだとか。

僕が一番だめだと思ってるのは手続き臭いコードなんで。`Python`の`numpy`/`pandas`でも言えることですが、手続き臭いforや分岐の入り乱れたコードって読みづらく遅いんですよね。

そのへんは`C++`や`Java`の悪癖を引きずってるようなイメージなんですよね。そっちの世界では`for`分岐入り乱れのほうが速いから普通なんですけどね。どうやら`Javascript`や`Typescript`も`C++`系のそういう発想のほうがよいみたいですね。

話は脱線したけど、手続き臭さよりベクター思考とかパイプライン思考のほうが似合ってる言語ですね。`R`の歴史は`xlispstat`や`S`/`S Plus`言語という大きな潮流の末裔なんですしね。

言語については色んなもの扱ってたから新規に学ぶのはさほど負担とは思ってないです。
