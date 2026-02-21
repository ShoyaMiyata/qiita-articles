---
title: privateメソッドの影響範囲と使い所を確認する
tags:
  - Ruby
  - Rails
  - private
  - メソッド
private: false
updated_at: '2024-11-25T11:12:53+09:00'
id: 59f8cb264f03831ff7b0
organization_url_name: lmi-inc
slide: false
ignorePublish: false
---
## はじめに
「他の場所から呼び出せないように、他のクラスの処理に影響を与えないようにするために使う？」
プライベートメソッドについて分かっているようで、完全に理解している自信がなかったので、どんな状況で使うのかをしっかり確認し、意図をもって実装できるようになりたいと思い、今回は調べることにしました。

## privateメソッドの目的

**定義されたクラス内部でのみ**使用され、クラス外部やサブクラスからは直接呼び出せないメソッドのことで、**他のメソッドをサポートする「補助的なメソッド」や、クラスの内部動作をカプセル化する**ために使います。

クラスの内部動作をカプセル化するというのは、クラスの実装詳細（内部ロジックや状態）を隠蔽し、外部から直接アクセスできないようにすることを指していて、これにより、外部から誤って操作されるリスクを減らします。

そして、**保守性の向上**にもなります。というのも、プライベートメソッドにすることで、**「このメソッドはクラス内部でしか使用されない」という意図を明確に示せる**ので、コードレビューや後々の修正時に、意図しない変更が生じるリスクを低減できるからです。

## privateメソッドの特徴
**インスタンスから呼び出されても答えてくれません。**

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/f94ce0ae-3c58-2356-cbfb-2f00c1735644.png)


あくまで、プライベートメソッドが働く範囲はクラスの内部だけになっています。
**クラスの中の人たちとだけ口を利いてくれます。
というかクラス内の人にしか知られていないイメージ**

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/ada7b576-23fa-a654-3a6f-9428ea2cf31c.png)


![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/0e021660-f916-9d26-a6e7-c643dc06e8b8.png)


## コードで確認してみる
サンプルでコードを書いてみました。
```ruby
    class Hoge
      
      def use_private_methods
        hoge
        hogehoge
      end

      # ここから下はprivateメソッド
        private
        def hoge
          puts "hoge"
        end
        def hogehoge
          # privateメソッド内で他のprivateメソッドも呼び出せる
          hoge
        end
      end
      
      # クラス内でプライベートメソッドを呼び出し
      Hoge.new.use_private_methods
      #=> hoge
      #=> hoge
    
      # プライベートメソッドはクラス外からは直接よびだせない
      Hoge.new.hoge
      #=> private method `hoge' called for 
      #<Fuga:0x000002529aed37f0> (NoMethodError)   
```
### 定義の仕方
これからプライベートメソッドを定義したいというときに``private``と書いてあげれば大丈夫です。
privateメソッドは``private``と書いたところから1段インデントしてあげると、他のメソッドと差がついて見やすくなります。

### privateメソッド呼び出しルール
:::note info

1. **インスタンスから直接呼び出せない**
1. **同じクラスのメソッドからの呼び出しは可能**
1. **サブクラスからは呼び出せない**
:::


サブクラスでプライベートメソッドを使用したい場合、**``private``** にしてしまうと継承先で使えないので、代わりに **``protected``** を使うのが適切です。

基本的にクラス内でしか使わないメソッドは``private``にしておくとよさそうです。（←雑かも？）

``private``にしておかないと、そのクラスから生成したインスタンスが、外部から呼び出せるメソッドになってしまうため、もし万が一、そのメソッドを間違えて使ってしまうと、予期せぬ事態になることも考えられます。

では具体的にクラス内でしか使わないメソッドはどんなものがあるのでしょうか？


## privateメソッドをよく使うタイミング
補助的な処理を``private``メソッドに切り出すことが一般的です。


-  **リクエストハンドリング**

入力処理の整形や検証処理は、クラスの外部から直接呼び出される必要がないためprivateにするのが一般的。

例えば、ストロングパラメータを定義した時は、コントローラ外部から呼び出す必要はないので、privateメソッドとして定義する。

```ruby
      private
      # ストロングパラメータ
      def user_params
        params.require(:user).permit(:name, :email, :password, :password_confirmation)
      end
    end
```
- **コールバック**
一応、そもそもコールバックとは

> コールバックとは、オブジェクトのライフサイクル期間における特定の瞬間に呼び出されるメソッドのことです。コールバックを利用することで、Active Recordオブジェクトが作成/保存/更新/削除/検証/データベースからの読み込み、などのイベント発生時に常に実行されるコードを書くことができます。

https://railsguides.jp/v4.2/active_record_callbacks.html#%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%81%AE%E3%83%A9%E3%82%A4%E3%83%95%E3%82%B5%E3%82%A4%E3%82%AF%E3%83%AB

コールバックの例
```ruby
    class ArticlesController < ApplicationController
      before_action :set_article, only: [:show, :edit, :update, :destroy]
      
    ## 省略
     
      private
    
      def set_article
        @article = Article.find(params[:id])
      end
    end
```

この``set_article``もクラスの外部から参照される利用する必要がないので、``private``メソッドにします。

- **共通処理の抽出**
クラス内の複数メソッドで使用されるが、外部から直接呼び出す必要のない共通処理をprivateに定義してまとめる。

```ruby:サンプルコード by GPT🙏
class Product
  TAX_RATE = 0.1

  def initialize(price)
    @price = price
  end

  # 税込価格を返す
  def price_with_tax
    apply_tax(@price)
  end

  # 割引後の税込価格を返す
  def discounted_price_with_tax(discount_rate)
    discounted_price = @price * (1 - discount_rate)
    apply_tax(discounted_price)
  end

  private

  # 共通処理として税込価格を計算
  def apply_tax(amount)
    (amount * (1 + TAX_RATE)).round(2)
  end
end
```

## おわりに
設計時に「この処理は外部に公開する必要があるか？」と意識することで、クラスの責務を明確にすることが大事だなと思いました。

またなぜこのメソッドをプライベートにするのか？を考えながらリーディング、実装していきたいと思います！

内容に不備あれば、ご指摘くださると幸いです。
最後までご覧いただきありがとうございました！
