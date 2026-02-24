---
title: api/controllerとweb/controller
tags:
  - API
private: false
updated_at: '2024-12-31T10:52:43+09:00'
id: cc5f5d9ad2a349b3b45c
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに
今回はコードリーディングをしていた時に、引っかかった`api`や`json`について調べたことをまとめていこうと思います。`web/controler`と`api/controller`は何が違うのか、json形式のデータはどのように使われているのか、確認する良い機会になりました。



## APIについて復習
API（Application Programming Interface）は、**クライアントとサーバー、またはサーバーとデータベースをつなぐ役割**を持っています。特にWebアプリケーションでは、以下のような2つの主な役割を担います。

1. **クライアントとサーバー間の通信**  
   クライアント（フロントエンド）にデータを提供する。
2. **サーバーとデータベースの橋渡し**  
   データベースから取得した情報をクライアントに渡す。

## API ControllerとWeb Controllerの違い
Railsでは、コントローラーはリクエストに応じた処理を行い、クライアントにデータやページを返す役割を持っています。このコントローラーには主に2つの種類が存在します。

### Web Controller
Web Controllerは主にHTMLやCSS、JavaScriptといった**ユーザーインターフェース（UI）を生成する**ためのコントローラーです。例えば、ブラウザで直接アクセスするページ（ビュー）を作成する場合に使用します。これに対応するビューはHTMLやERBファイルなどです。

```ruby
class UsersController < ApplicationController
  def index
    @users = User.all 
  end
end
```

この場合、`app/views/users/index.html.erb` というビューが呼び出され、ユーザー一覧がHTML形式で表示されます。


### API Controller
一方、API Controllerは**UIではなくデータそのものを返す**ために設計されています。これにより、クライアント（たとえばフロントエンドのJavaScriptやモバイルアプリ）はデータを取得して独自に表示や操作を行えます。ビューとしてはHTMLではなく、JSONやXML形式が用いられます。

```ruby
class Api::UsersController < ApplicationController
  def index
    @users = User.all
    render json: @users
  end
end
```

この例では、`@users`に含まれる全ユーザー情報をJSON形式で返します。`render json: @users`により、サーバーサイドでデータを動的に生成し、クライアントに提供します。

---

### Web ControllerとAPI Controllerの違いまとめ


| 項目             | API Controller                         | Web Controller                   |
|------------------|----------------------------------------|----------------------------------|
| **目的**          | データを返す                          | HTMLなどのUIを返す               |
| **返却形式**      | JSON, XML                             | HTML, CSS, JavaScript            |
| **例**            | `Api::UsersController`               | `UsersController`                |
| **RailsのView**   | JSONやXMLを生成                       | HTMLやCSSを生成                  |



## 動的・静的なページ
ここまでで、Web ControllerとAPI Controllerがそれぞれの役割に応じてどのような形式でレスポンスを返すのかを確認しました。これを踏まえて、次に「動的なページ」と「静的なページ」の違いを考えてみましょう。

- **動的ページ**  
  サーバーサイドで処理を行い、リクエストに応じてデータを動的に生成するページです。たとえば、API Controllerで認証済みユーザーに応じたデータを返したり、条件に基づいてリソースをフィルタリングしたりする場合が該当します。

- **静的ページ**  
  あらかじめ生成された固定データを返すページです。たとえば、特定の定義情報やバージョン情報を提供するAPIや、更新頻度が低いWebページが該当します。

Railsでは、動的なデータ処理を得意とするAPI Controllerと、固定的なHTMLを返すWeb Controllerを使い分けることで、柔軟な設計が可能です。


## MVCアーキテクチャにおけるWebとAPIの共通点
RailsはWebアプリケーション開発で使用されるMVC（Model-View-Controller）アーキテクチャを、API開発にも適用できます。それぞれの役割は以下の通りです。

### Model
データやビジネスロジックを管理し、データベースとのやり取りを行います。WebとAPIの両方で役割は基本的に同じです。

### View
- **Web**: HTML、CSS、JavaScriptを使い、ブラウザに表示する画面を生成します。
- **API**: JSONやXMLなどの形式で、クライアントに返すデータを生成します。


## JSONレスポンスを効率よく構築する方法
Railsでは、JSON形式のレスポンスを効率的に構築するために`Jbuilder`を利用できます。`Jbuilder`を使ったコード例は以下のようになります。

```ruby
# app/views/users/index.json.jbuilder
json.array! @users do |user|
  json.id user.id
  json.name user.name
  json.email user.email
end
```

この例では、`@users`に含まれるユーザー情報をJSON形式で整形し、レスポンスとして返します。

`json.array!`
`array!`は`jbulder`が配列を生成するためのメソッドです。このコードでは、`@users`に含まれるデータを基にJSON形式の配列を作成します。

`json.id`
JSONオブジェクトのキー`id`を生成します。

`json.id user.id`
`user`オブジェクトの`id`属性を取得して、その値を`id`キーに割り充てる。


## API開発における認証と認可
セキュリティはAPI開発において非常に重要です。Railsは、以下のような一般的な認証方法をサポートしています。

1. **Token-based Authentication**  
   各リクエストにトークンを含めることで、ユーザーを認証します。
2. **OAuth**  
   外部サービスを利用した認証やアクセストークンの発行に使用されます。


## まとめ
Railsは、API開発に必要な機能を多く備えたフレームワークです。APIとWebの違いを理解し、Railsの強力なツール（例えば`Jbuilder`や認証機能）を活用することで、効率的かつセキュアなAPIを構築できます。

