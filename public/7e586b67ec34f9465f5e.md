---
title: 「git switch 」でエラー発生！stashで未コミットの変更を守る
tags:
  - Git
  - GitHub
  - AdventCalendar2024
private: false
updated_at: '2025-05-28T10:44:06+09:00'
id: 7e586b67ec34f9465f5e
organization_url_name: lmi-inc
slide: false
ignorePublish: false
---
## はじめに


こんにちは！リンクアンドモチベーションの宮田です！昨日ぶりです。
枠が空いていたのでいただきました🙇‍♂️

本記事は[リンクアンドモチベーション Advent Calendar 2024](https://qiita.com/advent-calendar/2024/lmi)の3日目を担当しています。🎄

```
git switch main
error: Your local changes to the following files would be overwritten
by checkout:
    
Please commit your changes or stash them before you switch branches.
Aborting
```
このエラーに出会ったのがこの記事を書くきっかけです。
このエラーはなんだ？ということで、調べてみると、

**現在のブランチに未コミットの変更がある状態で別のブランチに切り替えようとした際に表示されるエラーで、Gitが「現在の変更が失われる可能性があるので処理を中止した」と警告している状況だとわかりました。**

では、この状況で、変更を保持したまま、別のブランチにチェックアウトしたい場合はどうしたらいいのでしょうか。

この記事が、僕と同じエラーに出会った一人でも多くの方のお役に立つことを願っております。

## 【前提】ブランチ切替時の注意点

変更を保持する方法を知る前に、ブランチ切り替え時に知っておきたいことを確認しておきます。

まずはこの記事を確認すると良さそうです。

https://qiita.com/aono1234/items/e869902a0d465d415f5b

今回僕が出会ったのエラーは
「**移動先のブランチよりも進んだ変更（コミット済）がある＋未コミットの変更がある**」状態で、ブランチを移動しようとしたため起こったものでした。

変更があってもエラーが発生せずに移動できる場合もあります。（上の記事にあるもう一つのパターン）

ただここで注意が必要です。

例えば、あるブランチで作業している時に、緊急で別の作業が入ってしまったというとき、**作業中のブランチ上にまだコミットしていない変更内容などが含まれている状態でブランチを切り替えると、その変更は切り替えた先のブランチに適用されてしまいます。**

:::note warn
**コミットしてない変更は移動先のブランチに反映される**
:::

だからと言って、作業途中の中途半端な状態でコミットをしてしまうと、コミットログが汚くなってしまいますし、どこまでやったか、作業のログがわかりにくくなってしまいます。

それでは困りますね。

とにかく、エラー発生の有無に関わらず、ブランチを移動するときに現在のブランチの状態を保持し、他のブランチに影響させないようにするための手段が欲しいです！

## stash（スタッシュ）

そんな時に使えるのが、変更を一時的に退避させて保管する「**stash（スタッシュ）**」です。スタッシュは「こっそり隠す」という意味があり、作業内容を一時的にコミットせずに保管する機能です。

以下でstashの使い方を確認していきます。

## ``git stash save "○○" -u``
スタッシュを作成したいときはこのコマンドを使用します。
``git stash save``の後に続けて、スタッシュの名前を指定します。名前は省略できますが、その代わりに自動でコミットIDやコミットメッセージを使った名前が付加されます。

``-u``： これはまだステージングされていない変更も含めて``stash``するというオプションです。

```terminal
git stash save "作業途中" -u

Saved working directory and index state On branchD: 作業途中
```
この時点で未コミット、未ステージングの変更は全てスタッシュの中に保管されVScodeの画面上からは消えます。

## ``git stash list``
作成したスタッシュの一覧化して確認できるコマンドです。
```
git stash list

stash@{1}: On branchD: 作業途中
stash@{2}: On main: 保存
```

こんな感じで出てきます。

## ``git stash apply / git stash pop``
作成したスタッシュを適用する場合は、このコマンドです。

```
git stash apply / git stash pop   # 直前に作成したスタッシュを適用
git stash apply "stash@{n}" # 指定した番号のスタッシュが適用される
```
``git stash apply``をすると、``git stash save``したときの状況を復元することが出来ます。

## ``git stash drop``
作成したスタッシュを削除するコマンドです。
一度スタッシュを作成すると適用後も残り続けます。
```
stash@{0}: On main: Change 20
stash@{1}: On main: Change 19
stash@{2}: On main: Change 18
...
stash@{18}: On main: Change 2
stash@{19}: On main: Change 1
```

直前のスタッシュを削除したいときは ``git stash drop``を使います。
```
git stash drop
git stash list

#直前のスタッシュが削除される
stash@{1}: On main: Change 19
stash@{2}: On main: Change 18
...
stash@{18}: On main: Change 2
stash@{19}: On main: Change 1
```
指定したいときは、``apply``と同様です
```
git stash drop "stash@{n}" # 指定した番号のスタッシュが削除される
```
## ``git stash clear``
もう全部使いません！一個一個消してたら日が暮れるよの場合は ``git stash clear``で一括削除できます。

```
git stash clear

git stash list
# なにも表示されない
```
使うときには慎重になった方がよさそうですね。

これらの``stash``コマンドを使って適切にブランチごとの変更を管理していきたいです。


## git pullのエラーでも使える！！ 

ちなみにこの

``git stash``　→　``git stash apply``

実はリモートリポジトリから``pull``したいときにも使えます。例えば、自分のローカルでの変更を``push``しようとしたとき、リモートが既に更新されていて、ローカルが遅れた状態になっていると、「**一度最新の状態を``pull``しなさいという**」エラーがでます。

```terminal
 git push
 
 ! [rejected]        branchc -> branchc (non-fast-forward)
error: failed to push some refs to 'https://github.com/ShoyaMiyata/GitGit.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. If you want to integrate the remote changes,
hint: use 'git pull' before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

そこで``git pull``を試みようとすると、つぎは
「**今のローカルの変更が上書きされちゃうよ。リモートの変更をマージする前に``commit``するか``stash``してね**」というエラーが出ます。
```terminal
git pull       
From https://github.com/ShoyaMiyata/GitGit
 * [new branch]      branchc    -> origin/branchc
 * [new branch]      brancha    -> origin/brancha
error: Your local changes to the following files would be overwritten by merge:
        ABC.rb
Please commit your changes or stash them before you merge.
Aborting
```
そこで、スタッシュの登場です。


1. **ローカルの変更を``stash save``で一時的に避難させておいて、**
1. **``pull``でリモートの状況と同期させた後に**
1. **``stash apply``で変更を復活させ、リモートよりもローカルの方が進んでいる状況を作り出して、**
1. **``git push``する**

という流れができます

## おまけ ``gitignore``
stashで変更履歴の保持について理解してきましたが、ここで変更履歴を残さない方法を紹介します。

- 自分の環境だけの変更なのに、いちいち変更を追跡されてめんどくさい。
- 一生コミットすることないのに、数字だけ生き残っていらっしゃる
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/3cd2cd78-9d31-9bce-91d6-fff6f690b44f.png)
☝こんな具合に


そんな時には、``gitignore``が使えます。変更を**無視**して、変更履歴に表示する追跡対象から除外してくれる機能です。

#### 実装手順

1.  ``.gitignore``ファイルを作成
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/da55424d-a655-197a-f470-b92bc408d39e.png)
1.  ``.gitignore``ファイルをステージングしてコミット
1.  ``.gitignore``ファイルに無視させたいファイル名やパスを記述
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/10390d70-39ee-058d-d4d8-630924399495.png)
1.  無視させたいファイルを確認
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/8952a90c-bfe1-c39d-7cf5-39eeaaf1d8b0.png)
色が薄くなっています。この状態だと``log.txt``に変更を加えても変更履歴が追跡されることはありません。
ちなみに``.gitignore``ファイルを作成する前の変更履歴は残ってしまうので、別途対応が必要です。


## 最後に
Gitのブランチ切り替え時に未コミット変更でエラーが発生する原因と解決策を解説しました。解決法としてgit stashで変更を一時退避し、作業を保護する方法を紹介しました。

内容に不備などありましたら、ご指摘ください。
最後までお読みいただきありがとうございました！




<br>

ぜひこちらの記事も併せてご覧になってください

https://qiita.com/Shoya-Miyata/items/4331528b5f03956365c5


## 参考文献

[図解！　Git & GitHubのツボとコツがゼッタイにわかる本［第2版］](https://www.amazon.co.jp/dp/4798073474/ref=sspa_dk_detail_0?psc=1&pd_rd_i=4798073474&pd_rd_w=selu0&content-id=amzn1.sym.f293be60-50b7-49bc-95e8-931faf86ed1e&pf_rd_p=f293be60-50b7-49bc-95e8-931faf86ed1e&pf_rd_r=4EAVGW7M9NDK1AJASCV0&pd_rd_wg=YYJpf&pd_rd_r=b9c0cada-3d32-4c24-87fe-c3fb930028d0&s=books&sp_csd=d2lkZ2V0TmFtZT1zcF9kZXRhaWw
)


[はじめてでもできる GitとGitHubの教科書](https://www.amazon.co.jp/%E3%81%AF%E3%81%98%E3%82%81%E3%81%A6%E3%81%A7%E3%82%82%E3%81%A7%E3%81%8D%E3%82%8B-Git%E3%81%A8GitHub%E3%81%AE%E6%95%99%E7%A7%91%E6%9B%B8-%E3%81%9F%E3%81%AB%E3%81%90%E3%81%A1-%E3%81%BE%E3%81%93%E3%81%A8/dp/481561539X/ref=sr_1_1?__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&crid=17PMQMOIULXNW&dib=eyJ2IjoiMSJ9.vbMce37mJ91OdM9KfQjxALFJlNRWh1718Db4DUS5ZrYeM8r_7gS0rdYgLa-WPbmISdjT8znocE7lcn8_3aa-pALTJ_Y4OgbybQzzsEScxJsOk2Ul3jtNxdDTZ5_JQE-BSiVh-FiNNw0AtLsSjJw1Eqs7m5DxJaFKxreDXB8-jNw6XGC3V_nh1IebQ1Q4c8j7NZ285T3-NKGu2bZo4IBbw1zzzNB-8gm0iAOvQokew7Q.WuL2YPaFEEchTHC2x_5dmcPefJVssYChtgd-DlXfRSg&dib_tag=se&keywords=git+%E6%95%99%E7%A7%91%E6%9B%B8&qid=1732954685&s=books&sprefix=git+%E6%95%99%E7%A7%91%E6%9B%B8%2Cstripbooks%2C274&sr=1-1
)



