---
title: GitHubでコミットを取り消す方法【図解で簡単解説】
tags:
  - Git
  - GitHub
  - VSCode
  - AdventCalendar2024
private: false
updated_at: '2024-12-03T09:26:15+09:00'
id: 4331528b5f03956365c5
organization_url_name: lmi-inc
slide: false
ignorePublish: false
---
## はじめに
こんにちは！リンクアンドモチベーションの宮田です！

本記事は[リンクアンドモチベーション Advent Calendar 2024](https://qiita.com/advent-calendar/2024/lmi)の2日目を担当しています。🎄

アドベントカレンダー初参戦ということでワクワクしながら書きましたが、至らない点もあるかと思います。精一杯書きましたので、どうぞ最後までお付き合いください！

<br>
4月に入社してからエンジニアとしての道を歩み始め、手厚い研修を経ました。とはいえ、まだまだわからないことだらけです。特に、GitやGitHubに関しては、

https://qiita.com/Shoya-Miyata/items/c37c4e36475e82696db4

以前、記事を書いて、「わかったつもり」だったのに、初めて自分でチケットを実装し、プルリクエストを出す段階で壁にぶつかりました。
<br>

**先輩「このコミットは別のPRで出してほしい！」**

**僕（別PRか。プッシュしちゃったけどどうしよう？？(´･ω･)）**
<br>
どう操作すればいいのかわからず、戸惑ってしまいました。これをきっかけに、自分の知識の足りなさに改めて気づかされました。
<br>

そこで今回は、
1. **GitとGitHubの基本を復習**
1. **VScodeでのGit操作がGitHub画面にどう反映されるかを確認**
1. **GitHubにプッシュしたコミットを取り消す方法とその種類を解説**

という3本立て（+α）で、僕自身の学びを共有していきます！

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/1e6d50ac-c4bd-9ae6-caf5-ab037ad0b937.png)

コミットの取り消し操作自体はそんな頻繁に行うものではないかもしれませんが、GitやGitHubについての理解を深める大変良い機会になりました！

この記事が、同じような悩みを持つ方の助けになり、「**GitHub、なんかよくわからん……**」という感覚を少しでも和らげられるきっかけになれば嬉しいです。

それでは、GitHubと仲良くなれるように頑張っていきましょう！！🐈🙌

## 目次
1. [はじめに](#はじめに)
2. [GitとGitHubの基本を復習](#gitとgithubの基本を復習)
3. [VScodeでのGit操作がGitHub画面にどう反映されるかを確認](#vscodeでのgit操作がgithub画面にどう反映されるかを確認)
4. [GitHubにプッシュしたコミットを取り消す方法とその種類を解説](#githubにプッシュしたコミットを取り消す方法とその種類を解説)
   - 4.1. [強制プッシュ](#強制プッシュ)
   - 4.2. [新しいコミットで取り消す](#新しいコミットで取り消す)
5. [チェリーピックという方法🍒](#チェリーピックという方法)
6. [最後に](#最後に)
7. [参考文献](#参考文献)



## **GitとGitHubの基本を復習**
復習といってもかなり簡単な説明になります

**Git**
Gitはソースコードの変更履歴を管理するための**バージョン管理システム**です。コードの変更を記録して、過去のバージョンに戻したり、複数人が並行して作業する際の変更内容を統合したりすることができます。Gitはローカルで使用でき、個人での作業にも対応しています
<br>
**GitHub**
GitHubは、Gitを利用したリモートリポジトリサービスの一つです。Gitのローカルリポジトリは自分のPCにしか存在しませんが、GitHubを使うことで、インターネット上にリモートリポジトリを作成し、複数の開発者が同じプロジェクトで作業できるようになります。チームで開発を行う際に、各開発者が自分のPCでコードを編集した後、GitHubに変更を**プッシュ（push）** することで、全員が同じコードを共有できるようになります。
<br>

以下、自分で作成したお手製GitHub図解です。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/429d3a74-672c-388b-fe54-c7da406fc993.png)

ローカルリポジトリとリモートリポジトリとの間でやり取りを表したのが以下の図です。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/aea239ac-f9f8-7149-5f61-dffb31a5bb96.png)

別のローカルリポジトリもいるパターンです。
結局図で理解するのが一番良さそうです。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/7a77cc87-6781-7401-3630-e50d5defaae6.png)




## VScodeでのGit操作がGitHub画面にどう反映されるかを確認

今回は説明用に**GitGitリポジトリ**を作ってみました。
GitHubで確認できるのはもちろん、リモートリポジトリです。

ブランチを切って、VScodeから**push**すると、GitHub上にPR(Pull Request)が作成されます。今回はわかりやすいように「PUSH」というコミットメッセージにしてあります。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/593dde91-ec5f-f707-4aa6-c483c8c85bac.png)

このPRをクリックすると、ConversationやCommitsで**このブランチからのコミット履歴**が確認できます。（※コメントが雑なのは許してください）

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/9abbac64-6474-f18f-7f70-11b7c2f317ad.png)

**コミット履歴**といっても今は一つしかなく、わかりにくいと思うので、一度PRを作成したブランチから、追加で変更を加えて``commit``→``push``してみます。
次のコミットメッセージはとりあえず「**CHANGE**」にしてみました。すると、以下の画面のように新規のコミットが追加されます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/604ee89d-f879-4886-d6c1-b1c8b9df0200.png)

:::note info
**一つのブランチからの``commit``→``push``は一つのPRにまとめられます。**
:::


では、試しにもう一つ別のブランチ(Branch b)を作ってそこからPRを作成してみます。
すると、以下の画面のようになります。
（わかりやすいようにPR名を編集してあります）

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/1d1f7e61-02cf-388d-fa4b-311d0042f914.png)

最初に作成したブランチと今回作成したブランチで別のPRになっています。

:::note info
**PRはブランチ単位で作成される**
:::


## GitHubにプッシュしたコミットを取り消す方法とその種類を解説

それでは本題です。プッシュ済みのコミットをどうやったらGitHub上から取り消すことが出来るのか。

取り消し方は2種類あります。

:::note info
1. **強制プッシュ**
1. **新しいコミットで取り消す**
:::


順番に説明していきます。

## 1. 強制プッシュ
まずは強制プッシュから説明します。今回僕が実際に使用した方法です。

ローカルリポジトリの状態が遅れている時、通常はプルをして、リモートリポジトリの最新の状態を取り込んでからでないとプッシュできません。
ただ、他のメンバーがまだプルをしていない状態や、１人で使っているリポジトリなどで、間違えてプッシュをした内容を書き換えても問題がない場合に、**強制プッシュ**という方法があります。

では説明していきます。まずはGitHubの画面を確認してみましょう。
**Commits**にそのブランチでプッシュされたコミットが古い順にならんでいます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/3dcd0109-284d-1019-b162-df431d8701b0.png)

例えば、取り消したいコミットがあったとして、まずは、**コマンドライン上で、コミットをリセット（なかったことに）** します。

```
git reset HEAD^ 　　　　　　　　　　#HEAD^は直近のコミットを指す
```
上のコマンドを使用しコマンドライン上でリセットすると、**ローカルリポジトリでそのコミットがなかったことになります。**　

下の図のようなイメージです。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/edc3d768-31b2-f36a-da87-8adb785f7836.png)

見てわかるように、**リモートリポジトリとの間に差（履歴が遅れている状態）が生まれることになります。**（まだこの段階ではローカルリポジトリでリセットした状態はリモートリポジトリには反映されません。）

``push``は**ローカルリポジトリの変更をリモートリポジトリへ反映させる（履歴を進める）コマンド**なので、
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/96948594-9593-26c3-b2f0-52bc870d8f57.png)

``git push``すると、



```terminal
git push       

To https://github.com/ShoyaMiyata/GitGit.git

 ! [rejected]        branchA -> branchA (non-fast-forward)
error: failed to push some refs to 'https://github.com/ShoyaMiyata/GitGit.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. If you want to integrate the remote changes,
hint: use 'git pull' before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```
「**リモートリポジトリの状態よりも遅れてるから、まずはリモートリポジトリの状態を受け入れてちょうだい**」となるわけです。でも今回はローカルリポジトリで意図的にコミットを取り消して作った状況であり、リモートリポジトリにも反映させたいので、

```
git push -f  
```
``git push``に``-f(--force)``オプションを付けることで強制的にリモートリポジトリに変更を反映させることができます。

つまり、**「うっせぇだまれです。黙って従うです。」** ということができます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/758eb115-a46f-fe75-45d1-ddf204bf7cc0.png)

実際に実行して確認してみると、

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/dad5587e-d8ef-f450-0354-db6dbec017bf.png)

晴れて、プッシュしたコミットを取り消すことができました。
今回はあくまで**PR段階で、まだ誰にもプルされていない状態**だったので強制プッシュで問題なく対応できました。

:::note warn
他の開発者と協力して作業している場合、その影響をしっかり理解してから行うことが大事になります。
:::

## 2. 新しいコミットで取り消す

先ほどの強制プッシュとは異なり、今度は新しいコミットで前のコミットをなかったことにする方法です。

今回使うのが **``git revert``** というコマンドです。

強制プッシュで使用した``git reset`` は指定したコミットまで戻すことで、作業をリセットしましたが、今回の``git revert`` は**既存のコミットを取り消す新しいコミットを作成する**ことで、履歴を安全に修正できます。

①直近のコミットを取り消す場合は
```
git revert HEAD^
```
②遡って取り消したい場合は、``git log``でコミットログを確認して、なかったことにしたいコミットのハッシュを取得して、
```
git log

commit 573eff4904eefc0177cdbcda1ffcf3a6266addbf  # 例
Author: Shoya Miyata
Date:   Sat Nov 30 11:10:29 2024 +0900
```
```
git revert 573eff4904eefc0177cdbcda1ffcf3a6266addbf
```
します。その状態で
```
git push
```
すると、GitHubの画面上では

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/3cb78444-c190-fdfb-a3ec-9e085a388330.png)

頭に**Revert**と付いたコミットが追加で作成されています。
これならば、リモートリポジトリの履歴を保持できるので、他の人が前のコミットをプルしていた場合でも、**Revert**コミットをプルしてもらうことでミスを修正できます。

:::note info
#### プッシュ済みのコミットの取り消し方

- ミスって不要なコミットしちゃったから消したい💦
    -  →**強制プッシュ**
- この前出したコミットやっぱなしってことにしたいな。やっぱりあれなしで🙇
    - →**Revertコミット**
:::
こんなイメージで使い分けが出来そうです。


一応今回は、別PRとして改めてコミットを出しなおすために、強制プッシュでコミットを取り消し、別のブランチで再度コミットをプッシュすることで対応できました。（もっとスマートな方法あればぜひ教えてください！）


## チェリーピックという方法🍒
先ほど別PRを出すために、コミットを取り消してから、再度同じ内容をプッシュするという方法を取りましたが、
チェリーピックという方法を使えば、**特定のコミットを選んで現在のブランチに適用することができます。** この「チェリーピック (cherry-pick)」という言葉は、さくらんぼ狩りのように、自分が欲しいものだけを選び取る行為から由来しているそうです🍒

たとえば、BranchAのコミットを別PRとして出したいとします。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/d3674a38-d6e3-c6f4-bfb7-3420bb2c7510.png)


今回はBranchCで受け入れていきましょう。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/c24495d6-1c81-1036-9a48-baae09143bd7.png)

そしたら、ターミナル上でBranchCに移動して``git log --all``します。
```
git log --all
```
``--all``オプションを付けることで、現在チェックアウトしていないブランチのコミット履歴も表示できます。(通常は現在のブランチのログのみが表示される)

```
 git log --all
 
commit 295fa2b3737094248028ee4176940d1ba58d9ffb (refs/stash)
Merge: 7067c60 01a1306
Author: Shoya Miyata 
Date:   Sun Dec 1 14:17:57 2024 +0900

    WIP on branchc: 7067c60 クラス作成

commit c200db35fe319da09bb657dc34774fb33cfc2347 (branch-A)
Author: Shoya Miyata 
Date:   Sat Nov 30 20:29:23 2024 +0900

    別PRで出して
```
対象のログを確認できたので、``git cherry-pick``をしていきましょう。
```
git cherry-pick <適用したいコミットのハッシュ>

git cherry-pick c200db35fe319da09bb657dc34774fb33cfc234　# 今回はこのハッシュ
```
問題なくコミットを受け入れることが出来たら、``git push``をします。
BranchCを確認してみると、

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/10695cf9-8484-93c5-9496-37a9fd01e5db.png)

**無事にBranchAのコミットをBranchCに反映させられました。**
この時、もともとのBranchAにはコミットが残っています。（必要ない場合は先ほど解説した強制プッシュを使うなどして、取り消す対応が必要です。）

今回はミスのコミットを別PRへ移すために``cherry-pick``を使いましたが、他のブランチのコミットを一部反映させたいというときにも使えそうです。

:::note warn
**ブランチ移動の際にエラーが発生して困った**場合は、こちらの記事もぜひご覧ください。
:::

https://qiita.com/Shoya-Miyata/items/7e586b67ec34f9465f5e

## 最後に

調べてインプットしたことを記事としてまとめる作業はかなり骨が折れますが、何度も、何度も自分で読み返して、間違いがないかを確認したり、どうやって構成したら伝わりやすいかを考え続けるうちに、最初は「わからない」がきっかけで調べ始めたことも気づけば、「自分のもの」になった感覚があります。

**良質なインプットのためのアウトプット
良質なアウトプットのためのインプット**

このサイクルをこれからも続けていきたいです！
最後までお読みいただきありがとうございました！

## 参考文献

[図解！　Git & GitHubのツボとコツがゼッタイにわかる本［第2版］](https://www.amazon.co.jp/dp/4798073474/ref=sspa_dk_detail_0?psc=1&pd_rd_i=4798073474&pd_rd_w=selu0&content-id=amzn1.sym.f293be60-50b7-49bc-95e8-931faf86ed1e&pf_rd_p=f293be60-50b7-49bc-95e8-931faf86ed1e&pf_rd_r=4EAVGW7M9NDK1AJASCV0&pd_rd_wg=YYJpf&pd_rd_r=b9c0cada-3d32-4c24-87fe-c3fb930028d0&s=books&sp_csd=d2lkZ2V0TmFtZT1zcF9kZXRhaWw
)


[はじめてでもできる GitとGitHubの教科書](https://www.amazon.co.jp/%E3%81%AF%E3%81%98%E3%82%81%E3%81%A6%E3%81%A7%E3%82%82%E3%81%A7%E3%81%8D%E3%82%8B-Git%E3%81%A8GitHub%E3%81%AE%E6%95%99%E7%A7%91%E6%9B%B8-%E3%81%9F%E3%81%AB%E3%81%90%E3%81%A1-%E3%81%BE%E3%81%93%E3%81%A8/dp/481561539X/ref=sr_1_1?__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&crid=17PMQMOIULXNW&dib=eyJ2IjoiMSJ9.vbMce37mJ91OdM9KfQjxALFJlNRWh1718Db4DUS5ZrYeM8r_7gS0rdYgLa-WPbmISdjT8znocE7lcn8_3aa-pALTJ_Y4OgbybQzzsEScxJsOk2Ul3jtNxdDTZ5_JQE-BSiVh-FiNNw0AtLsSjJw1Eqs7m5DxJaFKxreDXB8-jNw6XGC3V_nh1IebQ1Q4c8j7NZ285T3-NKGu2bZo4IBbw1zzzNB-8gm0iAOvQokew7Q.WuL2YPaFEEchTHC2x_5dmcPefJVssYChtgd-DlXfRSg&dib_tag=se&keywords=git+%E6%95%99%E7%A7%91%E6%9B%B8&qid=1732954685&s=books&sprefix=git+%E6%95%99%E7%A7%91%E6%9B%B8%2Cstripbooks%2C274&sr=1-1
)
