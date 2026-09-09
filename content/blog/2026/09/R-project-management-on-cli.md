+++
title = "Rのプロジェクト管理"
date = 2026-09-10
# updated = 2026-08-11
description = ""
# if you write to post, please comment out the below draft line.
# draft = true
path = "/blog/how-to-use/R-project-management-on-cli"

[taxonomies]
categories = ["how-to-use"]
tags = ["R", "Bash", "Neovim", "Zed"]

[extra]
mermaid = false
math = false
+++
R言語のプロジェクト管理ってどうしてるんかなぁ？思って調べても未整備だったからいくつか調べてシェルスクリプト作っておいた。
<!-- more -->
## Rのプロジェクト管理はない
意外とないんだね。というのが感想。Rstudioにはプロジェクトを作るのはあるようなんですが、私の環境はRstudio使ってないのでね。基本的にZED, nvimとradianという構成にしてて、jupyterでも利用できるようにしていますね。ここではRをZEDやnvimで使いやすくするlsp (lintr, languageserver, airを使う)の設定はここでは振れないですが、基本的にCLIでしターミナルを使ってシェルスクリプトを動かしてプロジェクトを作っちゃいましょうってやってます。

## Rのプロジェクトの構成
今作ってあるものを見本として見せておきます。
```
$ tree ~/projects/R/test/ -L 2
/home/yasuto/projects/R/test/
├── config
├── docs
├── input
│   └── foo.csv
├── main.R
├── notebooks
├── output
└── src

7 directories, 2 files
```
こんな感じになっててプロジェクトディレクトリのルートでは
```
$ pwd
~/projects/R/test
$ ls -la
total 48
drwxr-xr-x 9 yasuto yasuto 4096 Sep  9 22:40 .
drwxr-xr-x 4 yasuto yasuto 4096 Sep  9 22:40 ..
-rw-r--r-- 1 yasuto yasuto  444 Sep  9 22:40 .Rprofile
-rwx------ 1 yasuto yasuto    0 Sep  9 22:40 .env
drwxr-xr-x 7 yasuto yasuto 4096 Sep  9 22:40 .git
-rw-r--r-- 1 yasuto yasuto  368 Sep  9 22:40 .gitignore
drwxr-xr-x 2 yasuto yasuto 4096 Sep  9 22:40 config
drwxr-xr-x 2 yasuto yasuto 4096 Sep  9 22:40 docs
drwxr-xr-x 2 yasuto yasuto 4096 Sep  9 22:44 input
-rwxr-xr-x 1 yasuto yasuto 1263 Sep  9 22:40 main.R
drwxr-xr-x 2 yasuto yasuto 4096 Sep  9 22:40 notebooks
drwxr-xr-x 2 yasuto yasuto 4096 Sep  9 22:40 output
drwxr-xr-x 2 yasuto yasuto 4096 Sep  9 22:40 src
```
このような状態にしてて、`.gitignore`や`.env`というのを用意してあります。`.env`と言うのはここでは空ファイルですが、パーミッションで他人に見られないようにしています。(パーミッションを700にしてますが、600でもOK；7は読み書き閲覧実行OKで、6は実行できないです。)にしてあります。この .envはapiキーを書くところです。先にこうやって予約してるのは、うっかりのセキュリティミスを注意したいからですね。それと.gitignoreでも.envは除外するように書いてありますので、うっかりgitレポジトリにapiキーを保存するという恐ろしいことを避けるためです。

前置きが長くなりましたが、スクリプトをお見せします。少し長いですけど、色々考えたら長くなっちゃったです。まだ修正はするかもしれません。

## `mkrproj` Rのプロジェクト作成
これは僕のアイデアとGeminiの支援で作っています。だからシェルスクリプトに日本語が溢れてます。
```
cat << 'EOF' > ~/bin/mkrproj
#!/bin/bash

# プロジェクト名の指定（引数がない場合はカレントディレクトリを対象にする）
PROJECT_NAME=${1:-.}

if [ "$PROJECT_NAME" != "." ]; then
    echo "Creating project: $PROJECT_NAME"
    mkdir -p "$PROJECT_NAME"
    cd "$PROJECT_NAME" || exit
fi

# 1. フォルダ構造の作成
echo "Creating directory structure..."
mkdir -p src notebooks input output docs config

# 2. .gitignore の自動生成
echo "Generating .gitignore..."
cat << 'IGEOF' > .gitignore
# --- R programming ---
.Rhistory
.RData
.Ruserdata
.Renviron
.Rproj.user/

# --- Data & Outputs ---
input/*
!input/.gitkeep
output/*
!output/.gitkeep

# --- IDE & Editors ---
.vscode/
.idea/
.zed/
.DS_Store
Thumbs.db

# --- Python (if using reticulate/quarto) ---
.venv/
__pycache__/

# --- Environment Variables & Secrets ---
.env
.Renviron
IGEOF

# 3. 追跡用の .gitkeep 配置と秘匿ファイルの権限設定
touch input/.gitkeep output/.gitkeep src/.gitkeep notebooks/.gitkeep
touch .env && chmod 600 .env

# 4. Gitの初期化（未初期化の場合のみ）
if [ ! -d ".git" ]; then
    git init
    echo "Initialized empty Git repository."
fi

# 5. 初期Rスクリプト（main.R）の自動生成
echo "Generating main.R template..."
cat << 'MAINEOF' > main.R
#!/usr/bin/env Rscript

# ==============================================================================
# Project: R Analysis Template
# Description: Main entry point for data analysis
# ==============================================================================

# 1. ライブラリの読み込み
if (!require("pacman")) utils::install.packages("pacman")
pacman::p_load(
  tidyverse,  # データ操作 & グラフ描画
  here,       # プロジェクトルートからの相対パス解決
  dotenv      # .env からの環境変数ロード
)

# 2. 環境変数のロード
if (file.exists(".env")) {
  dotenv::load_dot_env(".env")
}

# 3. パスの設定
INPUT_DIR  <- here::here("input")
OUTPUT_DIR <- here::here("output")

# --- 分析コード記述エリア ---
message("🚀 R session initialized successfully!")
MAINEOF

chmod +x main.R

# 6. .Rprofile の自動生成
echo "Generating .Rprofile..."
cat << 'RPROFEOF' > .Rprofile
library(utils)

if (interactive() && file.exists("main.R")) {
  message("📦 Automatically sourcing main.R...")
  source("main.R")
}
RPROFEOF

echo "✨ R Project structure successfully initialized!"
EOF
```
となってて、僕は`~/bin/mkrproj`という名前で保存をして実行可能にしてあります。
```
$  (コピペする)
$ chmod 700 ~/bin/mkrproj
```
で使えるようにできます。ただし`$PATH`に `~/bin`が含まれている必要があります。

使い方ですが、プロジェクト作成先でここでは`test`プロジェクトをつくろうとすると
```
$ cd (移動先ディレクトリ)
$ mkrproj test
```
これで必要なものがすべて揃ってます。プロジェクトルートの扱いの関係で`.Rprofile`を用意して`main.R`を作ってあります。ここでtidyverseを読むようにしてあるけど利用するものは各自書き換えてみてください。`here`と`dotenv`LibraryをLibrary自動管理のpacmanで読み込むようにしてるのは、.envにあるapiキーを読み込むことと、データを読み込むときに処理が楽にするためです。

例えば、`input`にデータを置いたときを示してみます。ここでは`input/foo.csv`としてあります。
```
r$>  data <- read_csv(here::here("input", "foo.csv")
```
###相対パス問題の解消 (`here` パッケージ)
このテンプレートでは `here` パッケージを採用しています。エディタのカレントディレクトリ設定に依存せず、常にプロジェクトルート基準で安全にパスを指定できます。

例えば input/foo.csv を読み込む場合の実行例です。
```
r$> data <- read_csv(here::here("input", "foo.csv"))
                                                                                                                        Rows: 2 Columns: 3
── Column specification ────────────────────────────────────────────────────────────────────────────────────────────────
Delimiter: ","
dbl (3): id, prob, result

ℹ Use `spec()` to retrieve the full column specification for this data.
ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

r$> data
# A tibble: 2 × 3
     id  prob result
  <dbl> <dbl>  <dbl>
1    10   0.3      5
2    24   0.7     10

r$> data |>
    select("id") |>
    summary()
       id
 Min.   :10.0
 1st Qu.:13.5
 Median :17.0
 Mean   :17.0
 3rd Qu.:20.5
 Max.   :24.0

r$>
```
で読めるようにしてあります。

## 拡張するなら
このスクリプトではrenvやプリロードのLibraryを直接main.Rに書いてるわけで、uvやcargoみたいにここは責務を分けて、Rproject.tomlあたりに、設定を自由にかけるようにしたらレポジトリをどこに持っていっても再現しやすくできるかなぁ。今回はその機能まで入れてないけど、比較的簡単に実現はできるんで、やりたい人がいれば生成AIにでもこのスクリプトを相談すれば改造してくれますよ。

どういう形が一番いいかわからんからまだ様子を見るようにしてます。
