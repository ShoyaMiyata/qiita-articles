---
title: インスタンスを理解して、インスタンスメソッド・クラスメソッドの違いを確認する
tags:
  - Ruby
  - Rails
  - インスタンス
  - クラスメソッド
private: false
updated_at: '2025-02-04T19:17:38+09:00'
id: dfdfcb4bcdd0d593e668
organization_url_name: lmi-inc
slide: false
ignorePublish: false
---
## はじめに
メソッドの定義や使い方をコードリーディングしていると、「これは何のメソッドだろう？」と迷うことがよくあります。自分自身、インスタンスメソッドとクラスメソッドの違いをあまり意識せずに読んでることもありました。
この記事では、インスタンスとは何かという基本から振り返り、インスタンスメソッドとクラスメソッドの違いを簡単に解説します。メソッドの定義方法や、具体的な呼び出し方を確認しながら、理解を深めていきます。


## 目次
1. [インスタンスとは](#インスタンスとは)
2. [インスタンスメソッドの定義の仕方](#インスタンスメソッドの定義の仕方)
3. [インスタンスメソッドの使い方](#インスタンスメソッドの使い方)
4. [インスタンスメソッドとインスタンス変数](#インスタンスメソッドとインスタンス変数)
5. [クラスメソッドの定義の仕方](#クラスメソッドの定義の仕方)
6. [クラスメソッドの使い方](#クラスメソッドの使い方)
7. [クイズ](#クイズ)
8. [まとめ](#まとめ)



## インスタンスとは

プログラム上でオブジェクトのひな形（設計図）がクラスです。
クラスからオブジェクトをつくることをインスタンス化といいます。
そして、クラスから作られた（インスタンス化した）オブジェクトのことを**インスタンス**といいます。


![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/de2e0b09-e344-1c5b-1788-cf6a16729ff0.png)

大事なので図で説明します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/b6f0469d-ffc0-d7f7-1f54-d781d71d03e1.png)

### ↑↑↑↑これがインスタンスです！！


:::note info
- インスタンスは具体的な個々のオブジェクトを指す
- インスタンスは、クラスで定義された**インスタンスメソッド**を呼び出すことができる。
- 継承したクラスのインスタンスは、スーパークラスで定義されたインスタンスメソッドを呼び出すことが出来る。
- インスタンス変数を保持している。
:::

## インスタンスメソッドの定義の仕方

では、次にインスタンスメソッドの定義の仕方を確認します。
```ruby
    class Animal
      def initialize(name)
        @name = name
      end
    
      def speak
        "#{@name} makes a sound"
      end
    end
```
このコードでは、```Animal``` クラスを定義し、```initialize``` メソッドでインスタンス変数 ```@name``` を初期化します。```speak``` メソッドがインスタンスメソッドであり、インスタンスごとに異なる動作を行います。



## インスタンスメソッドの使い方
```ruby
    class Animal
      def initialize(name)
        @name = name
      end
    
      def speak
        "#{@name} makes a sound"
      end
    end
    # Animalクラスのインスタンス化とインスタンスメソッド呼び出しの例
    dog = Animal.new("Dog")
    puts dog.speak  # => Dog makes a sound
    
    cat = Animal.new("Cat")
    puts cat.speak  # => Cat makes a sound
```
```<クラス名>.new（引数）```でインスタンス化することが出来ます。
作成したインスタンスに```.メソッド名```を付けてあげることでクラスで定義したインスタンスメソッドを使えるようになります。

今回の例だと、```Animal.new("Dog")``` は ```Animal``` クラスのインスタンスを生成し、```dog.speak``` でインスタンスメソッドが呼び出され、インスタンス変数 ```@name```にアクセスします。


## インスタンスメソッドとインスタンス変数

インスタンスメソッドをもう少し理解するためにメソッドの探索経路についてのお話をします。
インスタンスメソッドをきちんと理解するためには**継承**を無視することはできません。インスタンスは作成元のクラスに定義してあるインスタンスメソッドだけではなく、親クラスのメソッドも使えるからです。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/1b5058ca-120c-0456-78b9-e2b57ba0e4e8.png)

インスタンスメソッドを実行すると、そのインスタンスが属するクラスに指定されたメソッドが存在するかどうかを判定します。属するクラスにメソッドがない場合は、さらにひとつ抽象度を上げたスーパークラスにメソッドを探しに行きます。
そして最後まで見つからなかった場合は、```NoMethodError```が発生するという感じです。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/7d6eb826-3cb6-e9b0-16e7-672b522c55a1.png)


そしてちなみにの話をすると、メモリ上で**インスタンスメソッドは、クラスオブジェクト**に、**インスタンス変数はインスタンス**に保持されます。
インスタンスメソッドはクラスオブジェクトのメソッドテーブルに格納され、インスタンスがメソッド呼び出しを行う際に、メソッド探索がクラスのメソッドテーブルで行われます。



```ruby:インスタンスメソッドはクラスオブジェクトに保持される
    #クラスオブジェクトで呼び出し
    p Animal.instance_methods
    # => [:hash, :singleton_class, :dup, :itself, :methods,
    #:singleton_methods, :protected_methods, :private_methods,
    #:public_methods, :instance_variables, :instance_variable_get,
    #:instance_variable_set, :instance_variable_defined?, 
    #:remove_instance_variable, :instance_of?, :kind_of?, :is_a?,
    #:display, :public_send, :class, :tap, :frozen?, :yield_self,
    #:then, :extend, :clone, :<=>, :===, :!~, :method, :public_method, 
    #:nil?, :singleton_method, :eql?, :respond_to?, 
    #:define_singleton_method, :freeze, :inspect, :object_id, :send,
    #:to_s, :to_enum, :enum_for, :!, :equal?, :__send__, :==, :!=, 
    #:__id__, :instance_eval, :instance_exec]

    #インスタンスで呼び出し
    p dog.instance_methods 
    #(NoMethodError)
```
```ruby:インスタンス変数はインスタンスに保持される
    #クラスオブジェクトで呼び出し
    p Animal.instance_variables
    #  => [] 

    #インスタンスで呼び出し
    p dog.instance_variables
    #  => [:@name] 
```
インスタンス変数は、インスタンスごとに個別に保持されるデータです。インスタンスメソッドは、そのインスタンスが保持するインスタンス変数にアクセスして、データを操作します。
見にくいですが、下の図のような感じになります。


![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/5180a81b-3b2c-3caa-ce68-872354bd4f95.png)





## クラスメソッドの定義の仕方

```ruby
    class Animal
      @@count = 0
    
      def initialize(name)
        @name = name
        @@count += 1
      end
    
      def self.get_count
        "Total animals: #{@@count}"
      end
    end
```
このコードでは、```self.get_count``` というクラスメソッドを定義しています。このメソッドはクラス全体に対して動作します。
クラスメソッドはインスタンスメソッドと異なり、メソッド名に```self.```を付けます。

## クラスメソッドの使い方

```ruby
# クラスメソッド呼び出しの例

class Animal
  @@count = 0

  def initialize(name)
    @name = name
    @@count += 1
  end

  def self.get_count
    "Total animals: #{@@count}"
  end
end

puts Animal.get_count # => Total animals: 0 

# インスタンスを作成するとカウントが増える
lion = Animal.new("Lion") 
tiger = Animal.new("Tiger") 

puts Animal.get_count # => Total animals: 2
```

- ```self.get_count``` はクラスメソッドであり、インスタンスを生成せずに呼び出せます。
- クラスメソッドは、```Animal.get_count``` のようにクラスから直接呼び出します

**とりあえず、クラスメソッドはインスタンスメソッドと異なり、インスタンス作成を必要としません。**



## クイズ
```ruby
animals = Animal.all
```
**Q : このとき、```animals```はインスタンスですか？**
理由付きで答えられるでしょうか？
考えてからスクロールしてみて下さい。

---

答えはNOです。
```animals``` 自体はインスタンスではなく、**コレクション**（```ActiveRecord::Relation```オブジェクト）で、**配列の各要素が ```Animal``` インスタンス**です。（```ActiveRecord::Relation```に関してはまた別の記事を書こうと思います。）

Ruby on Rails の **ActiveRecord モデルクラス**で、```<クラス名>.all``` が**クラスメソッド**として定義されていて、クラスオブジェクトに対して呼び出すと、**インスタンスの配列**が返されるようになっています。

そのため、```animals```でインスタンスメソッドを呼び出そうとすると、エラーが発生します。このときの```animals```は```ActiveRecord::Relation```オブジェクト（```Array```のように振る舞う）だからです。いくらクラスを遡っても、```speak```メソッドは見つかりません。

一応確認すると、
```ruby
# animals は ActiveRecord::Relation オブジェクト
animals = Animal.all
puts animals.class # => ActiveRecord::Relation
```
コレクションに対しては、each でループすることで個々のインスタンスにアクセスできるようになります。

```ruby
animals.speak
NoMethodError: undefined method `speak' for #<Animal::ActiveRecord_Relation:0x00007f9d4a1d2b10>
```

```ruby
# 各インスタンスのメソッドを呼び出し
animals.each do |animal| 
    puts animal.speak
end
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/17d0addf-66ee-4aa5-e21b-3dcfc0f4b558.png)

:::note warn
あくまで、インスタンスメソッドは個別の情報（インスタンス変数）を持ったインスタンスをレシーバーとして呼び出すことが出来るのであり、インスタンスコレクションは対象になりません。
:::


## まとめ

:::note info
- **インスタンス**：クラスから生成されたオブジェクトのこと
- **インスタンスメソッド**: インスタンスをレシーバーとして呼び出すメソッドです。主にインスタンス変数を操作し、個々のオブジェクトの状態に依存する処理を行う
- **クラスメソッド**: クラス自体をレシーバーとして呼び出すメソッドです。インスタンスの状態には依存せず、クラス全体に関わる処理を行う
:::

端的な言葉で説明できるか否かは理解度の大きな判断基準になりそうです。
インスタンス・インスタンスメソッド・クラスメソッドの理解の助けになれば幸いです。


## 参考文献

https://www.amazon.co.jp/%EF%BC%BB%E6%94%B9%E8%A8%822%E7%89%88%EF%BC%BDRuby%E6%8A%80%E8%A1%93%E8%80%85%E8%AA%8D%E5%AE%9A%E8%A9%A6%E9%A8%93%E5%90%88%E6%A0%BC%E6%95%99%E6%9C%AC%EF%BC%88Silver-Gold%E5%AF%BE%E5%BF%9C%EF%BC%89Ruby%E5%85%AC%E5%BC%8F%E8%B3%87%E6%A0%BC%E6%95%99%E7%A7%91%E6%9B%B8-%E7%89%A7-%E4%BF%8A%E7%94%B7-ebook/dp/B0756VF9Y3

https://railsguides.jp/layouts_and_rendering.html
