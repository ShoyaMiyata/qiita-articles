---
title: 'rakeタスク '
tags:
  - Ruby
  - Rails
  - rake
private: false
updated_at: '2024-11-23T12:06:36+09:00'
id: 89d6f9dd329e1daeb20f
organization_url_name: lmi-inc
slide: false
ignorePublish: false
---
## はじめに
rakeタスクの実装をすることになったので、その使い方やコードの要素についてまとめていきます。

## 目次  
1. [Rakeタスクとは](#rakeタスクとは)  
2. [Rakeタスクを利用する場面](#rakeタスクを利用する場面)  
3. [Rakeタスクの実装](#rakeタスクの実装)  
4. [コード分解と解説](#コード分解と解説)  
   - [名前空間 `namespace`](#名前空間-namespace)  
   - [`desc`](#desc)  
   - [`task`](#task)  
   - [`environment`](#environment)  
5. [Rakeタスクの呼び出しメソッド](#rakeタスクの呼び出しメソッド)  
   - [`invoke`メソッド](#invokeメソッド)  
   - [`execute`メソッド](#executeメソッド)  
6. [Rakeタスクの実行](#rakeタスクの実行)


## Rakeタスクとは
rakeはRuby版のmakeと呼ばれますが、ビルドだけでなく、データ処理やシステムの運用タスクを柔軟に自動化できるツールです

rakeタスクは、Railsプロジェクト内で```lib/tasks```フォルダに```rake```ファイル(```.rake```拡張子)を作成して定義します。


## Rakeタスクを利用する場面
- **データ連携**
- **DBのバックアップ**
- **定期的なデータ更新・削除**

## Rakeタスクの実装


以下のコマンドを入力することで、```lib/tasks```ディレクトリ配下にファイルが作成されます。


```ruby
 rails g task file_name 
 #file_nameは自分で考えられますが、タスク内容がわかるようにします

=> create  lib/tasks/file_name.rake


# サブディレクトリを指定する場合
 rails g task batch/file_name 
=> create  lib/tasks/batch/file_name.rake
```

そして以下のコマンドでタスク一覧を確認できます。
自分で設定したタスクが反映されているか確認します。
```ruby
 rake -T
```


作成したファイルに色々と書いていくと、rakeタスクが出来上がります。サンプルコードはこんな感じです。


```ruby
    # lib/tasks/batch/example_task.rake
    
    namespace :batch do
      namespace :hoge do
        desc "サンプルのRakeタスクを実行する"
        task fuga: :environment do
          # 実行する処理を定義
          puts "バッチ処理を開始します。"
    
          # 他のRakeタスクを呼び出す例
          Rake::Task["batch:hoge:sub_task"].invoke
    
          puts "バッチ処理が完了しました。"
        end
    
        desc "サブタスクの説明"
        task sub_task: :environment do
          puts "サブタスクを実行中です。"
          # サブタスクの処理
        end
      end
    end
```

## 分解
上のコードを分解して考えていきます
### 名前空間 ```namespace```
- **```namespace```**:名前空間を利用することで、異なるタスクが同じ名前を持っていても区別できるようにしています
- 次に```hoge```という名前空間でタスクグループをまとめています。

>**モジュールと名前空間の復習**
>モジュールはクラスのように継承したりインスタンスのように生成することはできないが、主に2種類の役割で使用される。
>1. **名前空間の提供**
>1. Mix-in
>
>- **名前空間の提供**
プログラムの規模が大きくなるにつれて、クラス名やメソッド名などで同じ名前を使ってしまうことによる名前の衝突が発生しやすくなる。モジュールを活用することで、名前の衝突を回避することができる。
>
>例えば、```Batch```モジュールに属する```hoge```クラスのフルネームは```Batch::hoge```となる。hogeクラスのメソッドを呼び出したい時は、```Batch::hoge.method_name```となる。

### desc
describeの略でタスクの説明を書くことができ、
```ruby
rake -T
```
を実行した時に、タスク実行用コマンドと一緒に出てくる説明用の文書になります。以下のように表示される。

```ruby
rake batch:hoge:fuga
    サンプルのRakeタスクを実行する
```

### task

```ruby
task fuga: :environment do
```
ここにtask名を書き、以下のタスクに行わせたい処理の内容を書いていきます。
このタスクを呼び出す時は、上の階層に書いた名前空間を付けてあげます。


### environment
**```:environment```** 
Railsの環境をロードすることを意味しています。これにより、Railsのモデルやメソッドが利用可能になります。

↑どういうことかというと、

通常RakeタスクはRailsアプリケーションのプロセスとは独立して実行されます。そのため、何も実行しなければ、Railsのモデルや設定はロードされません。
多くのRakeタスクではデータベース操作やモデルのメソッドを呼び出すことが一般的なので、```environment```を指定してあげます。



では、```environment```を指定すると何がよみこまれるのかというと以下の内容が読み込まれます。


- **データベース接続**：```config/database.yml``` の設定に基づいてデータベースに接続する
- **Railsの初期設定**：```config/application.rb``` や```config/environments/production.rb``` などの設定が読み込まれる
- **モデル・コントローラーのロード**：```app/models```、```app/controllers``` 内のクラスがロードされる
- **Gemの読み込み**：```Gemfile``` に記載されたgemが読み込まれる

そのため、データベースの更新やユーザーデータの取得を行う場合、environmentを省略すると動作しません。


### 呼び出しメソッド

####  ```invoke```メソッド
 - rakeタスク内で別のタスクを呼び出して実行するメソッドです
- ```Rake::Task```クラスのインスタンスメソッド
 - 指定したタスクを呼び出して実行します
- ```invoke```メソッドは同じタスクが複数回呼び出された場合、同じプロセス内では一度だけ実行避けます
- 呼び出すtaskの依存タスクも実行します
    
#### ```execute```メソッド
- 呼び出すタスクに依存するタスクは実行しない
- 毎回強制的に実行します。何度呼び出してもスキップしません





## Rakeタスクの実行
作成したrakeタスクは、ターミナル上でタスクを実行したいときは、以下のようにで実行できます。
```ruby
rake task_name
```
上のコードの例でいくと
```ruby 
rake batch:hoge:fuga
```


実行環境を指定したいときは、```RAILS_ENV```を追記してあげることができます。
```ruby 
RAILS_ENV=production rake batch:hoge:fuga
```
これは本番環境を指定したバージョンです。




