---
title: Gitをコマンドラインで操作したい
tags:
  - Linux
  - Git
  - GitHub
  - GitCli
private: false
updated_at: '2024-11-27T16:59:47+09:00'
id: c37c4e36475e82696db4
organization_url_name: lmi-inc
slide: false
ignorePublish: false
---
# はじめに
研修中に先輩から「本配属後はまずGitをCLIで操作できるようにしよう。CLIの方が操作が速く、Gitの動きを理解するのにも重要だよ」とアドバイスを受けました。そこで、Railsチュートリアルを進める中で、Gitコマンドの操作を実践する機会が多かったため、今回そのまとめをしてみました。

https://railstutorial.jp/chapters/beginning?version=7.0#cha-beginning


なお、この記事ではコマンド操作とGUIの比較も一部で行っています。使用するリポジトリは、Railsチュートリアルで作成したものです

# Gitコマンド
```
    git clone リポジトリのURL
    git status
    git add -A
    git commit -m "メッセージ"
    git branch ブランチ名
    git branch -d ブランチ名
    git switch -c ブランチ名
    git switch ブランチ名
    git merge ブランチ名
    git push
    git pull
    git log
```
それでは順番に紹介していきます。

## git clone 
リモートリポジトリのコードをローカル環境にコピーするコマンドです。

    git clone リモートリポジトリのURL


**この作業**
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/ad2c2691-77de-f10b-78f6-95fc7149739c.png)





## git status
    
```git status```を使用することで現在のリポジトリの状態を確認します。変更内容や追加されたファイルが表示されます。


    git status 
    
    On branch main
    Your branch is up to date with 'origin/main'.

    Changes to be committed:
      (use "git restore --staged <file>..." to unstage)
            modified:   app/models/application_record.rb


## git add -A

    
```git add-A```を実行するとGit管理下のすべての変更ファイルをステージングエリアに追加します。

**GUIだとこの操作**
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/755995cd-ccea-41c6-e677-e97ada80f2cc.png)

## git commit -m "〇〇"
ステージングされた変更をコミットします。

**この作業**
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/6fc926fc-abe9-f154-47e4-2cb79cef3906.png)



# ブランチ操作

画面左下のこれがブランチ名
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/8d0c9c08-3376-2510-b0d7-354632b7cfec.png)
これをクリックするとブランチ一覧が表示されますが、コマンド操作で行う方法も紹介します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/b345ca3c-a3fd-3aa1-18ed-288f9b8cd2b6.png)


## git branch 
```git branch```でローカルブランチの一覧を表示することができます。

```*```は現在使用中のブランチを表す
ブランチの利用は1人で作業するときでも有用です。ブランチで行った変更はmainブランチに影響しないため、試行錯誤を繰り返してコードがめちゃくちゃになってしまってもmainブランチに戻ればいつでも元の状態に戻せます。

※```-a```オプションをつけると削除したブランチも見ることができます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/a0f46c1b-2710-83f0-8dd7-d29275e1b397.png)


## git branch <作成するブランチ名>

```git branch <作成するブランチ名>```で任意の名前でブランチを作成することができます。
実際はどんな作業をするのかがわかる名称にするのが良いです。


## git switch <ブランチ名>

ブランチを切り替えます。

    git switch basic-login 
    app/models/application_record.rb
    Switched to branch 'basic-login'
    
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/e15cffe6-0565-eca2-89dd-bbdc5ec4aab1.png)

ブランチを切ったらそこで作業をしていきます。


## git switch -c <ブランチ名>
上で紹介した```git branch <作成するブランチ名>```と```git switch <ブランチ名>```の2つを同時にしてくれるコマンドです。
ブランチの作成と切り替えを同時に行います。

## git branch -d(D)

ブランチを削除します。```-d```はマージ済みのブランチ、```-D```は未マージのブランチを削除します。


    git branch -d #マージ後
    git branch -D #マージ前のブランチ





## git merge <ブランチ名>
現在のブランチに指定したブランチをマージします。まず、```git swtich main```でメインブランチに移動してから実行します。

あとは、

    git merge <ブランチ名>
    
でマージできます。

**これ**
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/e56501fb-2f28-dc60-ce9c-76f441e926a3.png)


![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/2de9a5f1-53f9-37ac-b4a8-fcc818ade124.png)


## git push　
1. ステージング ```git add```
2. コミット ```git commit```


が済んでいるものを```git push```
するとプッシュでき、GitHub上にも変更が反映されます。

## git pull
```git pull```することで最新のリモートリポジトリの状態をローカルリポジトリに反映させることができます。

**これ**
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/1fc05539-c12b-86ec-0b1f-5f59f31d9412.png)


## git log

過去の操作履歴を確認することができます。
実際に見てみてください。
以下のような感じで出てきます。

    sample_app % git log
    commit 4ee2e9b7e0781a7ad8aa18098c09b22892a67ce6 (HEAD -> main, origin/main, password-reset, newbranch)
    Author: User Name <email@example.com>
    Date:   Fri Nov 1 20:25:17 2024 +0900
    
        Add pasword reset
    
    commit df0768c764cd2f0f86ad86e8353b96a58bec1ed2
    Author: User Name <email@example.com>
    Date:   Fri Nov 1 14:24:56 2024 +0900
    
        change render-build.sh


### おまけ
間違えてコミットしてしまった場合の取り消し方　

〜追記〜　⇧めっちゃお世話になりました

    git reset HEAD^
直近のコミットを削除したい場合はこれで対応が可能です。

以下、```git reset```オプションです。

- ```--softオプション```：ワークディレクトリの内容はそのままでコミットだけを取り消したい場合に使用。
- ```--hardオプション```：コミット取り消した上でワークディレクトリの内容も書き換えたい場合に使用。
- ```HEAD^``` : 直前のコミット
- ```HEAD~n``` ：直前からn個のコミット分（git logでコミット履歴を確認しましょう）
- ```HEAD~{n}``` ：n個前のコミット（git logでコミット履歴を確認しましょう）



## 最後に
今日紹介した中でもよく使ったコマンドをよく使う順番で（ランキングではなく流れ）まとめておきます。


``` 
    git clone リポジトリのURL
    git status
    git switch -c ブランチ名
    git add -A
    git commit -m "メッセージ"
    git switch main
    git branch 　　　　　　　　　#ブランチ名の確認
    git merge ブランチ名
    git push
```

説明に誤りがあればご指摘ください。
最後まで読んでいただき、ありがとうございました。
