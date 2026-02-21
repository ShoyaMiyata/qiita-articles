---
title: Railsのパフォーマンスを向上させる　ActiveRecordの基本と最適化手法
tags:
  - Ruby
  - Rails
  - ActiveRecord
  - 新人エンジニア
private: false
updated_at: '2024-12-16T09:53:58+09:00'
id: 332adadbd1906a103a76
organization_url_name: lmi-inc
slide: false
ignorePublish: false
---
## はじめに

日々のコードリーディングの中で「これは何のデータを取得しているのか？」と疑問に感じることがあります。

特に、ActiveRecordに定義されたカスタムクエリを読み解く必要がある場合、基本をしっかり理解していないと迷子になりがちです。

**ActiveRecord**はRailsアプリケーションの中心的な要素であり、モデルクラス内で頻繁に活用されています。

なので、この基本を押さえなければ、コードリーディングはおろか、メンテナンスや機能追加も難しいと感じています。


そこで、本記事ではRailsガイドを参考に、ActiveRecordの基本を理解し、Railsガイドを読んでいて気になった、カスタムクエリや最適化手法に焦点を当てていきます。


## 目次

1. [はじめに](#はじめに)  
2. [ActiveRecord::Base の概要](#activerecordbase-の概要)  
3. [ActiveRecordの内部動作](#activerecordの内部動作)  
4. [関連付け（アソシエーション）の活用](#関連付けアソシエーションの活用)  
5. [遅延読み込みとN + 1問題](#遅延読み込みとN+1問題)  
6. [スコープの活用](#スコープの活用)  
7. [最後に](#最後に)  
8. [参考文献](#参考文献)  




## ActiveRecord::Base の概要

まず、最初に前提の振り返りからします。


**ActiveRecord::Base** は、Ruby on Rails の**ORM（オブジェクトリレーショナルマッピング）** の中心的なクラスです。このクラスを介して、データベース操作をRubyコードで簡潔に表現できます。
Railsでモデルを作成(``rails g model``コマンドを実行)すると、**自動的にActiveRecoerdを継承したクラスが作成されます。**
ActiveRecordのおかげで、SQLクエリを意識することなく、Rubyのメソッドチェーンを使用して複雑なデータベースクエリを実行できるようになるのです。



#### **モデル定義の例**
**ApplicationRecord**
```ruby
class ApplicationRecord < ActiveRecord::Base
  self.abstract_class = true
end
```

**モデル例**
```ruby
class Animal < ApplicationRecord
  validates :name, presence: true
end
```

この構造により、アプリケーション全体で共通するロジックや設定を ApplicationRecord にまとめることができます。

## ActiveRecord の内部動作
1. **データベース接続**
   - ``ActiveRecord::Base`` は、データベース接続を管理し、クエリの実行を担当します

2. **SQL の抽象化**
   - SQL クエリの構築と実行を抽象化し、開発者がより簡潔にデータベース操作を行えるようにします


Active Recordは、データベース内のデータにアクセスできる高機能なAPIを提供してくれます。 単一レコードのクエリ、複数レコードのクエリ、属性を指定してフィルタリング、並べ替え、特定フィールドの選択など、**SQLでできることはすべてActive Recordで実行できます。**

## 関連付けアソシエーションの活用

さらに、**Active Recordの「関連付け（アソシエーション: association）」を使うと、モデル間のリレーションシップを定義できます。**
```
has_many
belongs_to
```
などが代表的ですが、これらを活用することで
モデルにさまざまな便利メソッドが追加され、関連データをより手軽に操作可能になります。

一部紹介すると、
```ruby
collection
collection<<(object, ...)
collection.delete(object, ...)
collection.destroy(object, ...)
collection=(objects)
collection_singular_ids
collection_singular_ids=(ids)
collection.clear
collection.empty?
collection.size
collection.find(...)
collection.where(...)
collection.exists?(...)
collection.build(attributes = {})
collection.create(attributes = {})
collection.create!(attributes = {})
collection.reload
```
こんなにもたくさんのメソッドが追加されます。
ただ、意外にこのさりげない優しさのおかげで、「なんだこのメソッド。どこで定義したんだ？」という状態にもなりがちだったので、

https://railsguides.jp/association_basics.html

Railsガイドで、どんなメソッドが追加されるのかを一度確認しておくとよさそうです。


## 遅延読み込みとN+1問題

``.where``メソッドは、コレクションに含まれているオブジェクトを指定された条件に基いて検索します。このメソッドではオブジェクトは**遅延読み込み**されるので、**オブジェクトに実際にアクセスするときだけデータベースへのクエリが発生します。**（遅延読み込みとは、必要になるまでリソースのロードを遅らせる設計パターンのことをいいます。）

```ruby
@available_books = @author.books.where(available: true) 
# クエリはまだ発生しない

@available_book = @available_books.first 
# ここでクエリが発生する
```

### N+1問題

**N+1問題**とは、**データベースにおいて、多くのクエリが発行されることでパフォーマンスが低下する問題を指します。** ORMを使用している時に頻繁に発生する問題です。
 ```ruby
 books = Book.limit(10)

books.each do |book|
  puts book.author.last_name
end
```

このコードは一見何の問題もなさそうですが、実は実行されたクエリの回数が1回無駄に多いです。

最初に``Book.limit(10)``で本を１０冊検索するクエリを１回発行して、次にそこから``last.name``を取り出すのにクエリを１０回発行するので、合計で１１回クエリが発行されています。

親データを取得した後に、子データを１件ずつ取得するために追加のクエリが発行される場合に発生する問題。




### Eager Loading(事前ロード)を使う

この問題を解決するための方法の一つに、``includes``を使用する方法があります。関連データを一括でロードすることで、クエリの回数を削減できます。

```ruby

books = Book.includes(:author).limit(10)

books.each do |book|
  puts book.author.last_name
end

```
この書き方をすると、クエリの実行回数をなんと11回から２回に減らすことができます。

```ruby
# 1回目
SELECT books.* FROM books LIMIT 10

# 2回目
SELECT authors.* FROM authors
    WHERE authors.id IN (1,2,3,4,5,6,7,8,9,10)
```
N+1問題はアプリケーションのパフォーマンスを低下させる原因となるため、開発段階で意識して回避することが重要になります。うまいこと遅延読み込みを活用していきたいですね。


## スコープ  (scope) 

いくらActiveRecordが便利だからと言っても、毎回毎回情報を取得するためのコードを書くのはめんどくさいですし、繰り返し同じものを書くようだと、DRYの原則に反します。

そこで、よく使うクエリを**スコープ**に設定すると、関連オブジェクトやモデルへのメソッド呼び出しとして参照できるようになります。

スコープでは、where、joins、includesなどのメソッドをすべて使えます。どのスコープメソッドも、常に``ActiveRecord::Relation``オブジェクトを返します。

例えば以下のようなコードがあります。

```ruby
class Article < ApplicationRecord
  # 1. 公開済みの記事を取得するスコープ
  scope :published, -> { where(published: true) }

  # 2. 特定の日付以降に作成された記事を取得するスコープ
  scope :created_after, ->(date) { where('created_at >= ?', date) }

  # 3. 人気の記事（例: 閲覧数が100以上）を取得するスコープ
  scope :popular, -> { where('views_count >= ?', 100) }

  # 4. 特定のカテゴリに属する記事を取得するスコープ
  scope :by_category, ->(category) { where(category: category) }

  # 5. 最近更新された記事を取得するスコープ（例: 更新日の降順で並び替え）
  scope :recently_updated, -> { order(updated_at: :desc) }
end
```

そして、事前に用意したスコープを利用するときは以下のように呼び出します。

```ruby
Article.published  
# Article.where(published: true)
Article.created_after(Date.new(2023, 1, 1))
# Article.where(created_at >= "2023-01-01") 
Article.popular
# Article.where(views_count >= 100)
Article.by_category("Tech")
# Article.where(category: "Tech")
Article.recently_updated
# Aritcle.order(updated_at: :desc)
```
scopeを使うことで、コードをそのまま読めば、どんな情報が入っているかが理解しやすくなったと言えるでしょう。その分、命名には注意を払いたいところではあります。

:::note info
**スコープを使う利点**

- コードが簡潔になる
- 共通ロジックを再利用することができる
- チェーンが可能になる(他のクエリメソッドとの組み合わせ可）
:::


## 最後に
ActiveRecordは圧倒的に便利にしてくれるものでありながら、きちんと機能を理解していないと逆に不思議なことが起こっているとなりかねないとも思いました。

ガイドだけでもかなりの量がありますが、一個一個理解して自分のものにしていきたいです。

最後までお読みいただきありがとうございました！




## 参考文献
https://railsguides.jp/active_record_basics.html

https://railsguides.jp/association_basics.html

https://railsguides.jp/active_record_querying.html

