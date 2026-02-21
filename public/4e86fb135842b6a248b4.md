---
title: Cursor CLIを使ってエイリアスを秒で登録する
tags:
  - Zsh
  - Terminal
  - macOS
  - alias
  - cursor
private: false
updated_at: '2025-05-22T10:55:00+09:00'
id: 4e86fb135842b6a248b4
organization_url_name: lmi-inc
slide: false
ignorePublish: false
---
# はじめに

エイリアスをもう少し楽に登録したいを実現できたので記事にまとめてみました。

![zs.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/0465a1b7-27a6-49e4-82c8-c2c8574a59cc.gif)

#### Before

:::note alert
エイリアス登録って便利だとは聞くけど、正直なところ：

- `cat ~/.zshrc` や `vi ~/.bashrc` って毎回打つのがめんどくさい  
- `vi` エディタ、操作しにくい  
- 登録するのに少し壁があるので、「まあいっか」で後回しにしがち


:::

こんな感じで自分にとって、設定ファイルを編集すること自体がちょっとしたハードルでしたが、

#### After 
:::note info
- **Cursor上で `.zshrc` を編集できるように**
- エイリアス登録に使うコマンドすら**エイリアス化**して快適に登録
- ターミナル操作が **直感的** & **ストレスフリー** 
:::

## 目次

- [Cursor CLIツールを使えるようにする](#1-cursor-cliツールを使えるようにする)
- [エイリアス登録に必要なコマンドをエイリアス登録する](#2-エイリアス登録に必要なコマンドをエイリアス登録する)
- [注意点](#3-注意点)
- [おわりに](#おわりに)
---

## 1. Cursor CLIツールを使えるようにする

まず、`cursor` コマンドをターミナルから使えるようにする必要があります。  
以下の記事で手順を確認できます。


https://qiita.com/tacarzen/items/03f118a3a0fd37134052



## 2. エイリアス登録に必要なコマンドをエイリアス登録する

`.zshrc` を開いて編集して、反映して...という作業を一発でできるようにするためのエイリアスを登録します。

```bash
# Cursorでファイルを開く
alias cur="open -a Cursor"

# Cursorで.zshrcを開く
alias z="cur ~/.zshrc"

# 設定の反映
alias s="source ~/.zshrc"
```

### 登録方法

ターミナルで `.zshrc` を開く（初回だけ）

```bash
open -a Cursor ~/.zshrc
```
上記で ``.zshrc`` を開いたら、以下の alias をコピペして保存します：

```bash
alias z="open -a Cursor ~/.zshrc"
alias s="source ~/.zshrc"
alias cur="open -a Cursor"
```

反映する
```bash
source ~/.zshrc
```

これで次回から、
```bash
z  # .zshrc を Cursor で開く
s  # 設定をすぐ反映
```
という高速エイリアス登録ができるようになります


## 3. 注意点

### バックアップを取る

編集しやすくなったのはいいですが、`.zshrc` には環境構築やツール連携に関わる大事な設定が入っているので、**編集ミスや上書きのリスクを減らすために、初期状態のコピーを取っておくのが良さそうです。**

```bash
# 例えば
cp ~/.zshrc ~/.zshrc.bak
```

### コマンド名の衝突に注意

すでに使っているエイリアスとかぶらないように、**他のエイリアスと被っていないか確認**しておきましょう。

```bash
alias  # 現在のエイリアス一覧を表示
```


## おわりに
「Be Lazy」を体現するための快適なツールになりそうです！


エイリアスが登録されるまでの流れはぜひ以下の記事をお読み下さい


https://qiita.com/Shoya-Miyata/items/097311457c6d2562ae2c
