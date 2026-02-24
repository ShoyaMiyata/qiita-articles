---
title: 【図解】Auth0はIdP？SP？認証の仕組みとSSOをざっくり理解する
tags:
  - JWT
  - session
  - SAML
  - SSO
  - Auth0
private: false
updated_at: '2025-12-22T12:32:23+09:00'
id: c0570678ce21f7e51816
organization_url_name: lmi-inc
slide: false
ignorePublish: false
---
## はじめに

こんにちは！
株式会社リンクアンドモチベーションの宮田です。

エンジニアとして2年目になり、普段はRuby on Railsを用いたバックエンドの保守開発を担当しています。

本日、私事で大変恐縮ですが誕生日を迎えましたので、自分へのプレゼントとして認証について詳しくなろうと思います💪

というのも最近、業務の中でお客様とSSO（シングルサインオン）連携の調整を行う機会が増え、**Auth0（オースゼロ）** の設定に触れることが多くなってきました。

これまではマニュアルや手順に沿って設定を行っていたのですが、お客様との技術的なやり取りの中で「もう少し踏み込んだ仕組みまで把握しておかないと、適切かつ迅速な対応が難しいな」と感じました。

そこで、アドベントカレンダーの機会を活用して、Auth0を通じた認証の仕組みを自分なりに整理してみることにしました。

今回の記事では、

- **Auth0が「IdP」なのか「SP」なのかという役割の整理**
- **複数のサービス間でSSOが実現される構造**
など、調査を通じて「なるほど、そういうことだったのか」と納得した部分を中心にまとめています。

>認証基盤にはさまざまなサービスがありますが、今回はAuth0と、外部IdPとしてOktaを例に挙げて説明していきます。


なお、全体の把握と見た目のわかりやすさを優先するために**フローの図解を簡略化して表現しています。** 厳密なシーケンス図とは異なる部分があるかもしれませんが、同じように「中身の構造を整理したい」という方の助けになれば幸いです。


## 目次

- [1. はじめに](#1-はじめに)
- [2. Auth0はIdPなの？SPなの？](#2-auth0はidpなのspなの)
- [3. パターン1：Auth0自身がユーザーを管理する場合（IdP）](#3-パターン1auth0自身がユーザーを管理する場合idp)
    - [① ログインの開始](#①-ログインの開始)
    - [② 認証情報の入力](#②-認証情報の入力)
    - [③ 認証成功とセッションの確立](#③-認証成功とセッションの確立)
    - [④ 2回目からのログインが楽な理由：JWTとセッション](#④-2回目からのログインが楽な理由jwtとセッション)
- [4. パターン2：外部IdP（Okta等）と連携する基本フロー](#4-パターン2外部idpokta等と連携する基本フロー)
    - [① 外部IdPへのリダイレクト](#①-外部idpへのリダイレクト)
    - [② Oktaでの認証と検証](#②-oktaでの認証と検証)
    - [③ 3層の「鍵」が揃う](#③-3層の鍵が揃う)
- [5. 連携のための事前設定：「名刺交換」と「荷物の送り方」](#5-連携のための事前設定名刺交換と荷物の送り方)
    - [① 「名刺交換」（メタデータの設定）](#①-名刺交換メタデータの設定)
    - [② 「荷物の送り方」（Binding）](#②-荷物の送り方binding)
- [6. なぜログインし続けられるのか？セッションの階層構造](#6-なぜログインし続けられるのかセッションの階層構造)
- [7. SSOの仕組み：複数サービス間の移動](#7-ssoの仕組み複数サービス間の移動)
- [8. おわりに](#8-おわりに)
- [参考文献](#参考文献)

## 2. Auth0はIdPなの？SPなの？

最初に？が浮かんだのが、「Auth0はIdP（認証を出す側）なのか、それともSP（サービス側）なのか？」という点です。

:::note info
結論から言うと、**Auth0は設定次第でその両方の役割を演じ分ける**ことができます。
:::

### パターン1：Auth0がユーザーを管理する場合（IdPとして振る舞う）

いわゆる「メールアドレスとパスワードでログインする」形式です。この時、Auth0自身がユーザー情報を持ち、認証を行う **「IdP（Identity Provider）」** となります。

### パターン2：外部IdP（Okta等）と連携する場合（SP・仲介役として振る舞う）

顧客が既に使っているIdP(OktaやAzure AD)などを利用してログインする場合、Auth0は：

- **アプリに対しては：** **「IdP」** のように振る舞い、トークンを発行する。
- **外部IdPに対しては：** **「SP（Service Provider）」** として振る舞い、認証を外に任せる。

つまり、Auth0が間に入ってプロトコルの違いを吸収してくれる **「認証のブローカー（仲介役）」** になってくれるようです。

---

## 2. パターン1：Auth0自身がユーザーを管理する場合（IdP）

まずは、Auth0のデータベースにユーザーを保存し、Auth0が直接認証を行う基本的なフローを見ていきます。

### ① ログインの開始

ユーザーがサービス（アプリ）を利用しようとアクセスします。未ログインの場合、アプリはAuth0へと誘導します。
![スクリーンショット 2025-12-20 11.19.46.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/253a3a6a-7d3d-4620-950b-a43998b389cf.png)


### ② 認証情報の入力

Auth0のログイン画面が表示されます。メールアドレスとパスワードを入力し、Auth0が自身のDBで本人確認を行います。
![スクリーンショット 2025-12-20 11.19.52.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/2135296f-b23d-4efc-98d9-23435c2b02c0.png)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/8b6b4114-a974-4bd2-825f-603b385544e0.png)




### ③ 認証成功とセッションの確立

認証が成功すると、ここがポイントですが、**2つの重要なもの**が発行されます。

![スクリーンショット 2025-12-20 11.20.02.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/c6b676cb-5641-4f2e-b442-366310809520.png)


1. **JWT（アクセストークン等）**: アプリが「この人はログイン済みだ」と認識するための鍵。
2. **Auth0 Session**: Auth0というサーバー自体がブラウザを覚えておくためのパスポート（Cookie）。

>[Auth0による認証方法を完全に理解した](https://tech.visasq.com/complete-understanding-of-auth0)（この記事のセッションとJWTの説明がわかりやすかったです）

![スクリーンショット 2025-12-20 11.20.08.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/96b04f81-dac4-41ba-af0f-bebef7baac1d.png)
これで、ユーザーは無事にサービスを利用できるようになります。



### ④ 2回目からのログインが楽な理由

あと、調べていくうちに、JWTはセキュリティのために有効期限がかなり短く設定されるのが一般的であるということがわかりました。

が、そこでふと
**「そんなにすぐ期限が切れるなら、なぜユーザーは頻繁にログインを求められないのだろう？」**
と疑問が湧きました。

その答えを探っていくと**Auth0セッション**という仕組みに辿り着きました。



一度ログインした後、アプリのJWT（チケット）が切れてしまったとします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/d46cd83b-017c-4784-9ae6-f8fa32ee25a8.png)

しかし、ブラウザに「Auth0 Session（パスポート）」が残っていれば、**パスワード入力をスキップして新しいJWTを再発行**できます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/198abddc-8833-473a-931d-5f221374d6fc.png)

![スクリーンショット 2025-12-20 11.20.08.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/0eda50b4-f09c-4087-ab32-269ab0850a3b.png)

これが、私たちが普段体験している「一度ログインすれば、しばらくはログイン画面を見なくて済む」という仕組みの正体です。


## 3. パターン2：外部IdP（Okta等）と連携する基本フロー

次に、本題である「外部のIdP（Oktaなど）を使ってログインする」パターンを見ていきます。Auth0が「仲介役」となるケースです。

### ① 外部IdPへのリダイレクト

ユーザーがログインしようとすると、Auth0はドメイン設定などを見て「このユーザーはOktaで認証が必要だ」と判断し、Oktaの画面へリダイレクトします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/f3f96123-699e-4f8d-b7eb-3c46a3dffc81.png)

![スクリーンショット 2025-12-20 11.20.28.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/114697b1-486a-4f22-946d-2b2c476d5379.png)

### ② Oktaでの認証と検証

Okta側で認証が成功すると、その結果がAuth0に戻ってきます。Auth0は事前に交換しておいた**証明書**を使って、「本当にOktaから来た正しいデータか？」を厳密にチェックします。!
![スクリーンショット 2025-12-20 11.20.32.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/d965cc6f-6b1b-48fb-a543-9c55bd9daf48.png)


### ③ 3層の「鍵」が揃う

検証が通ると、Auth0はアプリ用のJWTを作成します。

![スクリーンショット 2025-12-20 11.20.35.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/fe2f7e02-8e53-46d7-b359-3b66ca7f9067.png)

最終的に、ブラウザは以下の**3つの鍵**を同時に持つことになります。これがSSOを理解する鍵です。

![スクリーンショット 2025-12-20 11.20.39.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/fba19758-11c0-4cb8-b3c4-e8f72e1d79aa.png)

1. **JWT**: アプリ用のチケット。
2. **Auth0 Session**: Auth0でのログイン状態を示すCookie。
3. **Okta Session**: 大元の認証基盤でのログイン状態を示すCookie。



## 4. 連携のための事前設定：「名刺交換」と「荷物の送り方」

このパターン2を実現するためには、Auth0と外部IdPの間で事前の設定が必要です。

### ① 「名刺交換」（メタデータの設定）

SAML連携は、お互いの「名前」と「住所」を正しく教え合う**名刺交換**から始まります。

![Gemini_Generated_Image_7r1mlj7r1mlj7r1m.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/363d6562-9e4e-4473-b4bb-0e82c79eae1f.png)
この名刺にあたる情報が「メタデータ」です。

特に重要なのが、以下の2つの項目です。

- **Entity ID（識別子）**:
    - いわば「名前」です。「私は `urn:auth0:my-tenant` という者です」と名乗るための一意のIDです。
- **ACS URL（Assertion Consumer Service URL）**:
    - いわば「返信先の住所」です。「認証が終わったら、このURLに結果を届けてください」という窓口になります。

これらをお互いの設定画面に正しくコピペすることで、ようやく「どこの誰に認証を頼めばいいか」「終わったらどこに返せばいいか」が繋がります。


### ② 「荷物の送り方」（Binding）

![スクリーンショット 2025-12-22 12.21.44.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/9a0192ee-6161-49c8-8ec6-27e5a8aff0d9.png)
![スクリーンショット 2025-12-22 12.21.51.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/bfca6d80-9a8d-4160-be31-a316f31ce1fb.png)


設定画面に出てくる「HTTP-Redirect」と「HTTP-POST」は、データの**運び方**の違いです。「ハガキ」と「小包」でイメージすると分かりやすいです。

|  | HTTP-Redirect (ハガキ)  | HTTP-POST (小包)  |
| --- | --- | --- |
| **使われる方向** | **行き**（Auth0 → IdP） | **帰り**（IdP → Auth0） |
| **中身** | 「認証して！」という短い依頼だけ。 | ユーザー情報、電子署名、証明書などが詰まった巨大なXMLデータ。 |
| **特徴** | URLに乗せて送る。軽くて速い。 | URLに入り切らないため、ボディに乗せて送る。 |

基本的には「行き」の依頼はデータが軽いのでRedirect（ハガキ）方式がデフォルトの設定のようですが、システムやセキュリティ要件によっては、行きもHTTP-POST（小包）に指定されることがあります。

Redirect方式だと、ブラウザの履歴やサーバーのアクセスログに「どのURLにアクセスしたか」が残ってしまうので、もしURLの中に機密性の高い情報やトークンが含まれている場合、それがログとして残るのを避けたいという場合もあるようです。


## 5. なぜログインし続けられるのか？セッションの階層構造

パターン2の最大のメリットは、3層の鍵による**長時間ログイン継続**できる点です。
アプリのJWT（チケット）が切れても、裏に控える2つのセッション（パスポート）が助けてくれます。

### 第一弾：Auth0セッションによる復帰

JWTが切れた際、まず **「Auth0 Session」** が確認されます。これが残っていれば、外部IdP（Okta）には問い合わせず、その場ですぐに新しいJWTが再発行されます（[サイレント認証というらしい](https://auth0.com/docs/authenticate/login/configure-silent-authentication)）。

![スクリーンショット 2025-12-20 12.11.49.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/cb4f640e-0072-4a91-9025-df9a271b30aa.png)

![スクリーンショット 2025-12-20 12.12.08.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/52fe32c6-05d2-4bf3-99fc-e4e5c48dcf07.png)


### 第二弾：Oktaセッションによる復帰

もしAuth0セッションも切れていた場合、Auth0は再度Oktaへユーザーをリダイレクトします。

![スクリーンショット 2025-12-20 12.20.14.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/7ae2ff40-dc45-470c-a0a2-8eee68a729a6.png)

![スクリーンショット 2025-12-20 12.20.19.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/e92b8c1a-51dc-4572-b535-03616dac62a8.png)

しかし、ここで大元の **「Okta Session」** が生きていれば、Oktaは**パスワード入力画面を出さずに**、即座に「認証成功」を返します。

![スクリーンショット 2025-12-20 12.20.24.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/fc5b5d3c-a2e6-4851-983f-695e40f0b73f.png)

結果として、ユーザーは何も入力することなく、再び全ての鍵が揃った状態に戻ります。

---

## 6. SSOの仕組み：複数サービス間の移動

最後に、この仕組みが複数のサービスを跨いだ時にどう機能するかを見ます。これこそがSSOのやりたいことですね。

※かなり簡略化した図にしています。

### サービスAからサービスBへの移動

ユーザーはすでに「サービスA」にログインしており、大元の「Oktaセッション」を持っています。この状態で、まだログインしていない「サービスB」を開きます。

![スクリーンショット 2025-12-20 12.24.59.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/8f71bee6-8304-4ce0-a871-3efe43175cf7.png)


### Oktaセッションによるログイン

サービスBから認証に飛ばされた瞬間、奥に控えるOktaがブラウザのCookieを確認し、有効な **「Oktaセッション」** を見つけます。

![スクリーンショット 2025-12-20 12.25.08.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/a7d0029e-936e-4872-a428-0a9bf2ab82dc.png)


Oktaは「この人はさっきサービスAで本人確認済みだ」と判断し、即座に認証を完了させます。これにより、ユーザーは一切の入力なしでサービスBも使い始めることができます。

## 7. おわりに

今回、IdPとSPの関係や、セッションの構造を図解しながら整理したことで、自分の中の「ブラックボックス」が大きく解消されました。


この記事が、私と同じように「仕組みがいまいちピンとこない」という方の理解の一助になれば嬉しいです。


## 参考文献

- [**「Auth0」で作る！認証付きシングルページアプリケーション**](https://www.google.com/url?sa=E&source=gmail&q=https://www.amazon.co.jp/dp/484439841X)

- [**Auth0による認証方法を完全に理解した - ビザスク Tech Blog**](https://tech.visasq.com/complete-understanding-of-auth0)

- [**Auth0公式：Identity Provider (IdP) vs Service Provider (SP)**](https://www.google.com/search?q=https://auth0.com/docs/authenticate/protocols/saml/saml-identity-provider-vs-service-provider)
- [**Auth0公式：サイレント認証の設定**](https://auth0.com/docs/authenticate/login/configure-silent-authentication)
