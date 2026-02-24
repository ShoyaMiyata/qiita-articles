---
title: 「PATHを通す」ってどういう意味？コマンドが動く仕組みから手順までを解説
tags:
  - Bash
  - 初心者
  - Linuxコマンド
  - 'path,'
  - '環境変数,'
private: false
updated_at: '2025-05-22T09:11:08+09:00'
id: 097311457c6d2562ae2c
organization_url_name: lmi-inc
slide: false
ignorePublish: false
---
# はじめに

よく聞く言葉、**「PATHを通す」** について調べたので記事としてまとめていこうと思います。

本記事では、 **PATHを通す方法とその前提としてコマンドとは何者か？** について整理します。

## 目次

- [はじめに](#はじめに)
- [コマンドってそもそも何？](#コマンドってそもそも何)
- [PATHとは？](#pathとは)
- [コマンドの「種類」は4種類ある](#コマンドの種類は4種類ある)
  - [エイリアス（alias）](#エイリアスalias)
  - [外部コマンド](#外部コマンド)
- [実行ファイルにも種類がある](#実行ファイルにも種類がある)
- [PATHを通すとは？](#pathを通すとは)
- [.bashrc / .zshrc って何？](#bashrc--zshrc-って何)
- [exportって何？](#exportって何)
- [sourceってなに？](#sourceってなに)
- [環境変数と設定ファイルの読み込み](#環境変数と設定ファイルの読み込み)
- [おわりに](#おわりに)


## コマンドってそもそも何？

ターミナルで `cd`や `ls`  、`rails ~` 、 `docker ~`と入力すると、なぜ動作するのでしょうか？

それは、**シェルがその名前に対応する「実行ファイル」を探して見つけてくれているから** です。


## PATHとは？

`PATH` は、**シェルがコマンドを探すための「ディレクトリ一覧」** を表す環境変数です。

`PATH`の中身を確認するとこんな感じで出てきます。

```bash
echo $PATH　# $は変数の中身を見るための記号です

/home/shoya/.rbenv/shims:/home/shoya/.rbenv/bin:/home/shoya/.rbenv/shims:/home/shoya/.rbenv/bin:/home/shoya/hello:/home/shoya/.rbenv/shims:/home/shoya/.rbenv/bin:/home/shoya/hello:/home/shoya/.rbenv/shims:/home/shoya/.rbenv/bin:/home/shoya/hello:/home/shoya/.rbenv/shims:/home/shoya/.rbenv/bin:/home/shoya/hello:/home/shoya/.rbenv/shims:/home/shoya/.rbenv/bin:/home/shoya/hello:/home/shoya/.rbenv/shims:/home/shoya/.rbenv/bin:/home/shoya/.rbenv/shims:/home/shoya/.rbenv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/usr/lib/wsl/lib:/mnt/c/WINDOWS/system32:/mnt/c/WINDOWS:/mnt/c/WINDOWS/System32/Wbem:/mnt/c/WINDOWS/System32/WindowsPowerShell/v1.0/:/mnt/c/WINDOWS/System32/OpenSSH/:/mnt/c/Program Files/Git/cmd:/mnt/c/Ruby32-x64/bin:/mnt/c/Program Files/PostgreSQL/13/bin:/mnt/c/Program Files/nodejs/:/mnt/c/Users/miyas/AppData/Local/Programs/cursor/resources/app/bin:/mnt/c/Ruby32-x64/bin:/mnt/c/Users/miyas/AppData/Local/Programs/Python/Python312/Scripts/:/mnt/c/Users/miyas/AppData/Local/Programs/Python/Python312/:/mnt/c/Users/miyas/AppData/Local/Programs/Python/Launcher/:/mnt/c/Users/miyas/AppData/Local/Microsoft/WindowsApps:/mnt/c/Users/miyas/AppData/Local/Programs/Microsoft VS Code/bin:/mnt/c/Users/miyas/AppData/Local/GitHubDesktop/bin:/mnt/c/Program Files/PostgreSQL/14/bin:/mnt/c/Program Files/PostgreSQL/16/bin:/mnt/c/Users/miyas/AppData/Roaming/npm:/mnt/c/Users/miyas/AppData/Local/Programs/cursor/resources/app/bin:/mnt/c/Users/miyas/AppData/Roaming/Cursor/User/globalStorage/github.copilot-chat/debugCommand:/snap/bin
```

PATHの中には、コロン(:)で区切られたディレクトリのリストが表示されていて、左から順にコマンドを探していきます。

`:`で区切られているのですが、見ずらいので改行してみました。

```bash
$ echo $PATH | tr ':' '\n'

/home/shoya/.rbenv/shims
/home/shoya/.rbenv/bin
/usr/local/sbin
/usr/local/bin
/usr/sbin
/usr/bin
/sbin
/bin
/usr/games
/usr/local/games
/usr/lib/wsl/lib
/mnt/c/WINDOWS/system32
/mnt/c/WINDOWS
/mnt/c/WINDOWS/System32/Wbem
/mnt/c/WINDOWS/System32/WindowsPowerShell/v1.0/
/mnt/c/WINDOWS/System32/OpenSSH/
/mnt/c/Program Files/Git/cmd
/mnt/c/Ruby32-x64/bin
/mnt/c/Program Files/PostgreSQL/13/bin
/mnt/c/Program Files/nodejs/
/mnt/c/Users/miyas/AppData/Local/Programs/cursor/resources/app/bin
/mnt/c/Users/miyas/AppData/Local/Programs/Python/Python312/Scripts/
/mnt/c/Users/miyas/AppData/Local/Programs/Python/Python312/
/mnt/c/Users/miyas/AppData/Local/Programs/Python/Launcher/
/mnt/c/Users/miyas/AppData/Local/Microsoft/WindowsApps
/mnt/c/Users/miyas/AppData/Local/Programs/Microsoft VS Code/bin
/mnt/c/Users/miyas/AppData/Local/GitHubDesktop/bin
/mnt/c/Program Files/PostgreSQL/14/bin
/mnt/c/Program Files/PostgreSQL/16/bin
/mnt/c/Users/miyas/AppData/Roaming/npm
/mnt/c/Users/miyas/AppData/Roaming/Cursor/User/globalStorage/github.copilot-chat/debugCommand
/snap/bin
```

だいぶ見やすくなりました。確かにPATHの中にたくさんのディレクトリがリストアップされていたようです。このリストの中から上から順にコマンドがあるか探しているみたいです。


例えば、ターミナルで`ls`コマンドを打つと以下のような流れで探索します。

:::note

1. `/home/shoya/.rbenv/shims/ls` ← ここにファイルある？
2. `/home/shoya/.rbenv/bin` ←ここは？を続けていって、
3. `/usr/bin/ls` ← ここにあるみたい。これを使おう！
4. `/bin/ls` ← もう見つかったから見に行かない
:::

その証拠に自分で適当に考えた架空のコマンドを打ってみると、少し反応までラグがありました。しばらく探しに行ってくれていたみたいです。


![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/bb347542-3ea9-4de0-95a4-3ff8c0ea921a.png)



なんかいろんなお返事が返ってきました。

じゃあ全部のコマンドがこのようにPATHの中を探しに行っているかというと、NOです。

コマンドには種類があるみたいです。以下見ていきましょう。



## コマンドの「種類」は4種類ある

シェルにおいて、コマンドの種類には以下の4種類があります

| 種類 | 説明 | 例 |
| --- | --- | --- |
| **ビルトイン** | シェルが内部的に持っている命令で、外部ファイルを呼び出す必要がない。 | `cd`, `exit` |
| **エイリアス** | 別名をつけて実際のコマンドを短縮化 | `alias gs='git status'` |
| **関数** | 複数の処理をまとめたシェル関数 | `myfunc() { echo "hi"; }` |
| **外部コマンド** | `PATH`上にある実行ファイル | `ls`, `git`, `node` ,`rails`  |


もう少し詳しく確認しておきましょう

## **エイリアス（alias）**

**あるコマンドに別名をつけたもの**。コマンド入力を別の形に変換してから実行されます。

よく使うコマンドを短縮するのに便利で、

**シェル起動時に `.bashrc` や `.zshrc` によって定義されます。←ここ重要な部分だと思います。**

## **外部コマンド**

**シェルが `PATH` 上で探して見つけた実行ファイル（実行可能なファイル）**

実際に `ls` や `git` などの **実行ファイル本体**
`which` や `command -v` でパスが返ってくるものは外部コマンドです

```jsx
which rails
/home/shoya/.rbenv/shims/rails

which git
/usr/bin/git

which ls
/usr/bin/ls

which cd
結果なし！
```

コマンドの種類を確認するには `type` コマンドが便利です。

```bash
type cd
# cd is a shell builtin

type ls
# ls is /bin/ls

type gs
# gs is aliased to `git status`
```


## 実行ファイルにも種類がある

先ほど実行ファイルについて軽く触れましたが、もう少し詳しくみてみます。実行ファイルとはコンピュータが直接実行できるファイルのことで、大きく分けて2種類あるようです。

:::note info
1. **バイナリ実行ファイル**
1.  **スクリプト実行ファイル**
:::


**1. バイナリ実行ファイル**

- C言語などでビルドされた**機械語の実行ファイル**（例：/usr/bin/ls）
- CPUが直接理解して実行できる形式

**2. スクリプト実行ファイル**

- **シェルスクリプトやPython, Ruby などのスクリプト言語で書いた「人が読めるテキストファイル」**
- 1行目にシバン（shebang、#!/bin/bashなど）があり、シェルが解釈して実行

<br>

**自分で作るなら基本は「スクリプトファイル」**

**「バイナリ実行ファイル」はC言語などでコンパイルして作られたもので、基本は外部アプリやOS組み込みのもの**って感じですね。

スクリプトファイルにシバンをつけてあげることで、実行ファイルの中身をコンピュータが読むときに

- どの言語で実行するべきかを明確に指定できる
- 「バイナリかテキストか」という判断による実行エラーを防ぐことができる

```bash

#!/bin/bash 　

echo "#!はシバンといって、今回はこのechoコマンドを「bashで実行するんだよ」と教えてくれてます"
```

それではそろそろPATHを通す方法を見ていきます。

## PATHを通すとは？

ここまで、PATHとコマンド、実行ファイルについて確認してきましたが、PATHを通すとはどういうことなのでしょうか？実際に手を動かして確認してみました。

たとえば、`~/scripts/hello` というスクリプトファイル(実行ファイル)を自作してコマンドにしたい場合

### ① スクリプトを作る

```bash
# ~/scripts/helloファイルの中身

#!/bin/bash 
echo "こんにちは！"
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/4378e2d5-67f7-4154-8cd5-9c229c6dce3a.png)


### ② 実行権限をつける

```bash
chmod +x ~/scripts/hello
# 読み込み権限(r)・書き込み権限(w)・実行権限(x)があった中のxですね。
```

### ③ PATHに登録する

```bash

export PATH="$HOME/scripts:$PATH"

# ~/.bashrc または ~/.zshrcの中に記入します。
# 直接ターミナルに打ち込むこともできますが、一回きりになってしまいます。理由はのちほど。
```

### ④sourceで変更を反映させる

```jsx
source ~/.bashrc
```

これでどこにいても `hello` と打つだけで「こんにちは！」とあいさつできます。良かったですね。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/1320fede-a214-46a6-bd9d-ccb6ea91e0fc.png)


無事挨拶できました🙌

**では途中で出てきた`.bashrc` / `.zshrc`や`source`とは何なのでしょうか？**

---

## `.bashrc` / `.zshrc` って何？

- `.bashrc`（bash）や `.zshrc`（zsh）は、**シェル起動時に読み込まれる設定ファイル**
- エイリアスや環境変数（PATH）を**毎回自動で設定する**ために使います

```bash
# .bashrc の中身にこれを書くと、パスを登録できます。
export PATH="$HOME/scripts:$PATH"
```

## `export`って何？

**変数を環境変数として使えるようにするためのコマンドです。**

```bash
export PATH="$HOME/scripts:$PATH"
```

これで`$HOME/scripts`（`/home/shoya/scripts`） を**PATH**の先頭に追加して、PATHをシェル全体に知らせています。

`cursor ~/.bashrc`だけで別の記事を書く

その後、変更を反映するには：

```bash
source ~/.bashrc
```

## `source`って何？

`source` は、「スクリプトの中身を今のシェルに読み込む」という意味です。

とすると、`.bashrc` に書かれた設定をすぐに反映できます。

`source`しないと現在のターミナル上では反映されません。

しかし別のターミナルをこのタイミングで開くと反映されています。どういうことでしょうか？

## 環境変数と設定ファイルの読み込み

ちょっとターミナルを擬人化するとわかりやすかったので最後に紹介していきます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/de4a9413-824c-49a7-a3db-ea58b49969da.png)


- ターミナルを開くと、**新しいターミナルくん**が誕生する
- ターミナル君は生まれた瞬間に ノート（`.bashrc`または `.zshrc`）を読んで、**中身をぜんぶ暗記**する
- でもその後に `.bashrc` を書き換えても、**ターミナルくんは気づかない**
- だから、「もう一回ノート読んで！」とお願いするコマンドが`source ~/.bashrc`
- 新しく開いたターミナルくんは「最初に`.bashrc`や `.zshrc`を読み込む」から、**すでに最新の状態を知って生まれてくる。**

sourceではなく、ターミナルにexportコマンドを直接打ち込んだ場合の環境変数のスコープはこんな感じです

```bash
# ターミナルくんAで export した場合

[ターミナルくんA]
├─ export NAME="DECOPIN"  # ホワイトボードに書いた
├─ bashくん（子）      # NAME を知ってる！
├─ pythonくん（子）    # 知ってる！
├─ rubyくん(子)        # 知ってる！
└─ lsくん（子）        # 知ってる！

[ターミナルくんB]（別のタブ）
└─ NAME ？            # 知らない
```

しかし、最初にターミナルは最初に`bashrc`または `.zshrc`というノートを読み込んでいるので、

```bash
# .bashrcに書いておけば、
# .bashrcの中身
export NAME="DECOPIN"

# ターミナルくんA
$ echo $NAME
DECOPIN

├─ bashくん（子）      # NAME を知ってる！
├─ pythonくん（子）    # NAME を知ってる！
├─ rubyくん(子)        # NAME を知ってる！
└─ lsくん（子）        # 知ってる！

# ターミナルくんB(別タブで生まれた)
$ echo $NAME
DECOPIN

├─ bashくん（子）      # NAME を知ってる！
├─ pythonくん（子）    # NAME を知ってる！
├─ rubyくん(子)        # NAME を知ってる！
└─ lsくん（子）        # 知ってる！

# ノート（.bashrc）に書いておけば、
# みんな同じ初期知識をもって生まれてくる！
```

## おわりに

だいぶ長くなりましたが最後までお読みいただきありがとうございました。

なかなか普段使うコマンドを意識することはないですが、なんで動いているのか？を意識してみるとかなり学びが深いなと思いました。

最後にまとめの表を置いておきます。

| 概念 | 意味 |
| --- | --- |
| PATH | シェルがコマンドを探す場所の一覧 |
| コマンドの種類 | ビルトイン / エイリアス / 関数 / 外部コマンド |
| `$` | 変数の中身を参照 |
| `export` | 環境変数を登録する |
| `source` | 設定ファイルを今のシェルに読み込む |
| `.bashrc` / `.zshrc` | シェルの起動時に読み込まれる設定ファイル。PATHやエイリアスを永続化するのに使う |
