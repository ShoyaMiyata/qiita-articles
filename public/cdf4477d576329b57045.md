---
title: 今さら聞けない、gemとBundlerの基本と使い方
tags:
  - Ruby
  - Rails
  - Gem
  - Bundler
private: false
updated_at: '2025-02-11T08:20:12+09:00'
id: cdf4477d576329b57045
organization_url_name: lmi-inc
slide: false
ignorePublish: false
---
# はじめに

今回は、Rubyに関する gem、Railsについて簡単に復習した後、bundlerについて説明していきます。というのも環境構築の際、```bundle install```に何度もお世話になり、何をしてくれているのか気になったのがきっかけです。
環境構築時やgemを追加する度に何をしているのか曖昧なままなのは気持ちが悪いので今回まとめてみました。

gemやRailsに関しては、基礎過ぎて振り返ることがないかもしれませんが、意外と「知ってそうで知らなかったこと」もあるかと思います。

内容に不備あれば、ご指摘いただけると幸いです。


# 目次
- [gem](#gem)
- [Rails](#rails)
- [bundler](#bundler)

## gem

**gemとはRubyのライブラリのこと**

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/4049bb25-055a-f348-6d4f-1df185005834.png)


gemはRubyの拡張機能のことで、欲しい機能があれば追加できます。
    さらに自分でgemを作成することも可能で、他のユーザーに向けて公開することが出来ます。

Ruby のライブラリは主に [RubyGems.org](#http://rubygems.org/) に gem として置かれています。直接ウェブサイトを閲覧したり、gem コマンドを使用してそれらを探すことができます。

:::note info
- **ライブラリ**：特定のプログラムを定型化して他のプログラムから引用できる状態にし、さらにこれを複数集めたファイル
- Rubyではそれらのほとんどは **gem** という形式で公開されています
- RubyGems は ライブラリの作成や公開、インストールを助けるシステム

:::

    gem search -r
    gem search -r rails
    gem search -r rails kaminari
    gem search -l 

- ```-r``` : remote RubyGems.orgリポジトリを検索することを意味する
- ```-l``` : local インストール済みのgemに検索をかけられる

## Rails
[Rails](#https://railsguides.jp/) は、「Ruby で書かれたウェブアプリケーションフレームワーク」ですが、実は**Railsもgem**です。当たり前だよって思う方が多いかもしれませんが、この事実に気づいたとき、「そうなんだ！」と驚いた記憶があります。

Railsの学習を開始するタイミングは、とりあえず何もわからないけど講師の方に言われた通り、コードをコピペして環境構築していたので、大前提の知識が入っていませんでした。（きっと教えてくださっていたと思いますが、当初は周辺知識がないので何のことだかわかっていなかったと思います。）

以下は[RubyGems.org](#https://rubygems.org/search?query=rails)で 公開されているrails gemです。「rails」の右側にはバージョンが記載されています。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/8f01ca56-687d-bc58-a7a7-8fe72e2379bc.png)

Rails は、railties、actionpack、activerecord などの複数の gem によって構成されています。このように、個別の機能が gem として提供されているため、柔軟に開発ができます。
詳細画面に行くと、インストールのために必要なGEMFILE用のコードと、インストール実行用のコマンド、そして、他のgemとの**依存関係**が書いてあります。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/861450f3-564b-974d-9439-5e039beaa49f.png)




## bundler

ようやく今回の本題に入ります。

Railsの詳細画面に他のgemとの依存関係があったように、gem同士はさまざまな依存関係にあり、管理するgemの数が増えるほど複雑になってきます。

例として、ページネーションの実装を助けてくれる```kaminari```の詳細画面を見てみると、なにやら色々と書いてあります。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/9db25d4c-0016-536a-6fc7-bd802f9a2693.png)

自分のGemfileを確認してみても、

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/51b7323b-4d7a-cc7c-7d6b-24cd7e7b7c3f.png)

かなりたくさんのgemが入っています。
となると、複数のgemの依存関係を管理するのって大変そうな気がしてきます。

**そこで登場するのが、bundlerです。**

:::note info
**bundlerは、gem の依存関係を管理するためのツール**で、もちろん**bundlerもgem**。bundlerを使うことで、複数のgemの依存関係を保ちながらgemの管理ができるということです。

:::

簡単に言えば、**「gemのバージョン管理をやってくれる便利gem」**

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/a6940e9e-0e7e-fe88-5aff-469e7f6c8672.png)

https://bestgems.org/ (人気gemランキングサイト)の中でbundlerは1位


![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/f7fcd3bc-ac21-f958-594e-22fe4803f445.png)

bundlerというgemによってrubygems.orgからインストールしたgemの依存関係を考慮して、うまいことRailsアプリにインストールしてくれていることはわかった。が、それはどうやっているのでしょうか。


調べてみると、2つのファイルを使っていました。
- Gemfile
- Gemfile.lock


そして、bundlerを利用してrails アプリにgemをインストールする方法は以下の2つあります。

    bundle install
    bundle update

ちなみにこの違いを理解していないとバグ発生の可能性あるそうで、、

---
:::note warn
bundle installと　bundle updateの違い
:::



```bundle install```

- Gemfile.lock が存在する場合、その内容に基づいて gem をインストールします。
- Gemfile.lock がない場合は、Gemfile に記述された最新バージョンをインストールし、新たに Gemfile.lock を作成します。
- Gemfile に新しい gem が追加された場合、その gem をインストールし、Gemfile.lock を更新します。

```bundle update```
- Gemfile.lock を無視し、Gemfile に記載された最新互換バージョンを取得します。
- Gemfile.lock が大きく更新されるため、予期せぬバグが発生する可能性があり、慎重に使用する必要があります。


```bundle update```コマンドは慎重に使い、基本的には```bundle install```だけを使う様にした方が良いかもしれません。

---

とにかくなるほど、```bundler```のおかげで、```bundle install```を実行することで2つのGemfileを見ながら適切なバージョンをインストールしてくれるのですね。


# おわりに
ここまで見るとgemが何ものなのかわかりましたね。また、railsに機能を追加したいときに、gem fileにコードを記入し、```bundle install```を実行する理由が何なのかをだいぶ理解できたと思います。
基礎の部分をおざなりにしないで、理解していくのが大事だと再認識させられました。

最後までご覧いただきありがとうございました。
