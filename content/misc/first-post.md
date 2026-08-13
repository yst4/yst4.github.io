+++
title = "first post"
date = 2026-08-10
hidden=true
[taxonomies]
tags = []

[extra]
date_format = "%m-%d"
hidden= true
mermaid = true
math = true
+++
これはtest投稿です。学習ログなど残しておくために用意しています。
<!-- more -->

:smile: :warning:

## :orange_square: 数式

$$x = {-b \pm \sqrt{b^2-4ac} \over 2a}\$$
## marmaid記法

{% mermaid() %}
```mermaid
gantt
dateFormat YYYY-MM-DD
title ガントチャートのサンプル

section A　section
  完了したタスク :done, des1, 2022-07-06,2022-07-08
  アクティブなタスク :active, des2, 2022-07-09, 3d
  未来のタスク : des3, after des2, 5d
  別な未来のタスク : des4, after des3, 5d
section B section
  完了したタスク :done, des1, 2022-07-06,2022-07-08
  アクティブなタスク :active, des2, 2022-07-09, 3d
  未来のタスク : des3, after des2, 20d
  別な未来のタスク : des4, after des3, 30d
```
{% end %}

## admonition

{% admonition(type="tip", title="tip") %}
The `tip` admonition.
{% end %}

## ソースコード
{% admonition(type="example", title="example") %}
rust 公式のチュートリアルの数字あてゲーム
{% end %}

```rust,linenos,name=guessing_game.rs
use std::io;
use std::cmp::Ordering;
use rand::Rng;
fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..101);


    println!("The secret number is: {}", secret_number);    //秘密の数字は次の通り: {}

    loop{
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = guess.trim().parse()
            .expect("Please type a number!");                 //数値を入力してください！

        println!("You guessed: {}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
```

[![alt text](/icons/open-in-kaggle.svg "open in kaggle")](/)
![alt text](/icons/colab-badge.svg "open in colab")
