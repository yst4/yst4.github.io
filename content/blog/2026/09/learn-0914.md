+++
title = "学習ログ 26-09-14 AAAMLPとボイラープレート"
date = 2026-09-14
# updated = 2026-08-11
description = ""
# if you write to post, please comment out the below draft line.
# draft = true
# path = ""

[taxonomies]
categories = ["learn"]
tags = ["learn-ML","python"]

[extra]
mermaid = false
math = false
+++
AAAMLPも６章まで終わらせたけど、ボイラープレートがだるすぎたんで。対策チョット示しておこう。
<!-- more -->
OOP的というのか手続き型プログラミングで一番苦手なのはボイラープレートなんだよな。

## AAAMLP ６章終わった
６章の特徴量エンジニアリングのところまで終わったけど、ここボイラープレートの山って感じでだるいなぁー。だった。それでいくつか回避策があるんで。生成AIに書かせるというのではなくてね。

### 1. エディタの正規表現と置換で対処

簡単に言えば `^(\w+)$` の応用ですね。例えば、
```
mean
max
min 
std
var
```
という範囲で置換表現するんですね。そこで上記のものが登場する。
検索窓で`^(\w+)$`と書けば、`mean`,`max`,`min`を拾ってくれます。
つぎに変換窓で`feature_dict['$1'] = np.$1(x)`と書いてやって正規表現と範囲指定で変換すれば
```
feature_dict['mean'] = np.mean(x)
feature_dict['max'] = np.max(x)
feature_dict['min'] = np.min(x)
feature_dict['std'] = np.std(x)
```

と変換してくれます。いちいち同じことを繰り返して打ってると、このパターンは集中力を削ぐのでタイポしやすいんですよね。その対策でもあります。これはzedでやってるけど、vscodeでも同じでしょう。この正規表現についてはvimでも表現がすこし変わるけど問題ないしこういうやり方もある。(vim系を使ってる一は示さなくても自分で調べるでしょ。😁)

```
percentile 10
percentile 20
percentile 30
```
といったものだったら、`^(\w+) (\d+)$`ととすれば選べます。変換では`feature_dict['$1_$2'] = np.$1(x,$2)`こうすればいい。ちょっとした正規表現を覚えておけば、スニペット云々で迷わなくなると思うよ。正規表現は結構便利なので。

### 2. 辞書を使って、名前空間を扱ってみること

これは記述が少ない方法で、ミスを減らせる便利さがあります。
```
stats = ['mean', 'max', 'min', 'std', 'var', 'ptp']

feature_dict2 = {name: getattr(np, name)(x) for name in stats}
```
こうしておけば、1. と等価なものになります。 percentileのものはすこし応用になりますけど割愛します。

### 3. 2とはチョット違った方法
これもGeminiに教えてもらったんだけど、こういう方法あるんですね。
```
funcs = [np.mean, np.max, np.min, np.std, np.var, np.ptp]

feature_dict3 = {f.__name__: f(x) for f in funcs}
```

### まとめ
辞書形式の内包表記でできたらいいけど、関数名の名前空間は別だったよな？ということでできない思ってたんですが、Geminiができるよって見せてくれたので残しておきます。正規表現の力を借りるというのはスニペット貼り付けに近い感覚かもしれないけど、2,3のほうがプログラムとして洗練されてるなぁ。と感じますね。
