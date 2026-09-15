+++
title = "学習ログ 26-09-15 pandasのpipeとreset_index"
date = 2026-09-15
# updated = 2026-09-15
description = ""
# if you write to post, please comment out the below draft line.
#draft = true
# path = ""

[taxonomies]
categories = ["learn"]
tags = ["learn-ML", "Python", "pandas"]

[extra]
mermaid = false
math = false
+++
pandasの扱いにくい元凶なんだとおもうけど、reset_indexとpipeのこと
<!-- more -->
## AAAMLPの６章
AAAMLP (和書名は :book: `kaggleGrandmasterに学ぶ機械学習実践アプローチ`)  の６章を終わらせてたんですが、ボイラープレートへの対策は昨日の日記に書いたんですが、それのおまけです。generate_featuresという関数を写経してて、group_byとreset_indexの話をGeminiの関係を聞いてたんだけど、これはたしかにバグの温床だな。とpandasのindexうんぬんはpolarsのことを調べるときによく言われてたけど、これ何かって感じでした。一度リセットしないとその後のプログラムで違う計算をしてしまう。この問題対策なんですよね。group_by使えば、かならずreset_indexをセットで考えないとあかんな。となった。

これは使用上のスコープの制限が曖昧だからなんだろうな。それで、明示的にwith構文でも作っておきたいなぁ。と言ってれば、pandasにはpipeがあってというのをGeminiが言ってきて、一応リファクタリングしたプログラムを見せてくれた。僕はpipeなんてあるの知らなかったけど、これならRのdplyrにちかい操作になるんやなぁ。polarsのパイプラインを取ってつけたような仕様なんです。reset_indexを呼び忘れをしてバグるくらいだったら安全だなという程度の認識ですね。

あとから加えた仕様っぽくって、ラムダ関数を使って一級オブジェクトを示してるけど、書き方が冗長なのは仕方ないのかもね。対話AIって知識の宝庫ではあるから、こうやって知らんことを思わぬ引き出しで教えてくれることがあるのが便利なところですよね。僕は彼らにすべてを任そうという気はないんですけどね。関数型言語を扱ってたら、馴染みやすい考え方だけどね。特にラムダ関数はね。

今回AAAMLPを写経するうえで、ライブラリの古い仕様の問題を解決させるのに助かってますね。5章の最後のtensorflowを使ったものに関しては、修正箇所が多かったんで完全丸投げしてしまいましたが、多すぎたら心が折れますわ。😁

R言語はプログラミングしようと思ったら、遅延評価が基本なんでreset_indexとは違った問題はあるんですけどね。評価をその場所で行うforceを使うか使わないかに気を配らないといけないので。例えば、 [:book: Advanced R chapter 10 Function Factories](https://adv-r.hadley.nz/function-factories.html) に簡単な例が示されてます。この例では関数を返す関数を使って、状態を保持するということをやるのですが、その状態の評価のタイミングがずれたら値がバグるって問題の指摘なんです。こういう作り方はいつ使うの？と言われると、状態を保持する関数（クロージャ）をループ等で動的に大量生成するときやメモ化といったキャッシュ処理技術では使いますね。

```python
def generate_features(df):
    dt = df['date'].dt
    features = {
        'year': dt.year,
        'month': dt.month,
        'dayofweek': dt.dayofweek,
        'weekofyear': dt.isocalendar().week,
        'weekend': dt.dayofweek >= 5,
    }
    # df.assign(**features) は非破壊的（新しいDataFrameを返す）ため、代入が必要です
    df = df.assign(**features)

    aggs = {}

    aggs['month'] = ['unique',  'mean']
    aggs['weekofyear'] = ['unique',  'mean']
    aggs['num1'] = ['sum', 'max',' min', 'mean']
    aggs['customer_id'] = ['size',  'unique']

    agg_df = df.groupby('customer_id').agg(aggs)
    agg_df = agg_df.reset_index()
    return agg_df

### another solution of generate_futures  by Gemini
def generate_features2(df):
    aggs = {
        'month': ['nunique', 'mean'],
        'weekofyear': ['nunique', 'mean'],
        'num1': ['sum', 'max', 'min', 'mean'],
        'customer_id': ['size', 'nunique']
    }

    return (
        df
        .assign(
            year=lambda x: x['date'].dt.year,
            month=lambda x: x['date'].dt.month,
            dayofweek=lambda x: x['date'].dt.dayofweek,
            weekofyear=lambda x: x['date'].dt.isocalendar().week,
            weekend=lambda x: x['date'].dt.dayofweek >= 5,
        )
        .groupby('customer_id', as_index=False)
        .agg(aggs)
        .pipe(lambda d: d.set_axis([
            f'{c0}_{c1}' if c1 else c0
            for c0, c1 in d.columns
        ], axis=1))
    )
```
