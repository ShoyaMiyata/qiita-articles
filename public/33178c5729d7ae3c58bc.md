---
title: Rakeタスクにdry_runを組み込む理由と実装方法
tags:
  - Ruby
  - Rails
  - rake
  - dry_run
private: false
updated_at: '2024-12-09T15:56:58+09:00'
id: 33178c5729d7ae3c58bc
organization_url_name: lmi-inc
slide: false
ignorePublish: false
---
## はじめに
rakeタスク実装に当たり、``dry_run``がなぜ必要なのか、どんな仕組みで機能しているのかをすぐに腹落ちさせることができなかったので、今回は記事にすることで理解を深めようと思い、書いています。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3924509/ffea84a5-cfdc-5deb-6772-8d94569ba52b.png)


## dry_runとは？
まず、前提としてプログラム（処理）を実行することを**run**といいます。

**dry_run**とは、プログラミングやシステムの用語で、「**本番の処理を実行せずに、その過程をシミュレーションする**」ための方法や、フラグを指します。

## dry_runの目的
**dry_run**は実際にデータベースや外部に影響を与えずに、処理が正常に動作するかを確認できるので、本番作業へ移る前に欠かせないステップです。

かなりの量のデータを更新するrakeタスクを実装した場合、それをすぐに本番環境で実施するのは**リスキー**です。

意図しないデータまで書き換わってしまったり、思ってたよりも影響範囲が狭かったりなど。一か八かでコマンドを押すようなギャンブルはやめようということで、事前に処理をシミュレーションできる（ログ）、**dry_run**を利用していきます。


:::note info
**dry run(シミュレーション)モード**

- 実際のデータの更新や変更は行われない
- その処理が実行されるときにどのデータがどのように変わるか、標準出力やログで確認できるようにすることで、実際の処理を実行する前に動作確認を行うことができる
:::



## 実装方法
### 実装コード
以下は、``dry_run``を取り入れたRakeタスクの一例です
<br>
**メインロジック**
```ruby:lib/pathces/update_examples
module Patches
    class Example
      attr_reader :run_mode
    
      def initialize(run_mode: "dry_run")
        @run_mode = run_mode
      end
    
      def execute
          update_examples
      end
    
        private
        
          def dry_run?
            @run_mode != "run"
          end
          
          def update_examples
            Rails.logger.info("更新対象のexample数:#{count.example}件")
                examples.each do |example|
                    
                    Rails.logger.info("id: #{example.id}の処理を開始します")
    
                    unless dry_run?
                        begin
                            example.update!(status:"hoge")
                            Rails.logger.info("id: #{example.id}のstatusがhogeになりました")
                        rescue => e
                            Rails.logger.error("id: #{example.id}の更新に失敗しました")
                        end
                    end
               end
           end
    end
end
```

**Rakeタスクの定義**

```ruby:lib/tasks/example.rake

namespace :patches do
  desc "Example Rake Task"
  task :update_example, ['run_mode'] => :environment do |t, args|
    Patches::Example.new(run_mode: args.run_mode).execute
  end
end
```
## コードの詳細解説

``dry_run`` の機能を実際にはどうやって実装しているのか、パーツを確認していきます。

### ``attr_reader``

``run``するか``dry_run``するかの判断をするために``run_mode``をセットします。
```ruby
attr_reader :run_mode
```
``attr_reader``はインスタンス変数に対して「読み取り」を可能にするアクセサーメソッドを自動で定義します。今回は``@run_mode``というインスタンス変数に対して、``run_mode``（ゲッター）というメソッドを提供します。``attr_reader``は外部からの不正なデータ変更を防ぐため、インスタンス変数を変更させたくない状況の時に使えるメソッドです。

---
***参考***
- ``attr_accessor`` : インスタンス変数への外部からのアクセス（参照・代入）を可能にする
インスタンス変数に対して「読み取り」と「書き込み」を可能にするアクセサーメソッドを自動で定義する。
``@name``というインスタンス変数に対して、``name``（ゲッター）,``name=``（セッター）という2つのメソッドを提供する。
- ``attr_reader``：外部からの不正なデータ変更を防ぐため、インスタンス変数を変更させたくない状況の時に使える。
- ``attr_writer``：外部から値は変更できるが、読み取りを禁止したい場合に用いられる（パスワードなど）

---
### ``def initialize(run_mode: "dry_run")``
```ruby
def initialize(run_mode: "dry_run")
    @run_mode = run_mode
end
```
``initialize``メソッドの``run_mode``引数にデフォルト値として``"dry_run"``が設定されています。そのため、``run_mode``を明示的に指定しない場合、``dry_run``モードで動作します。

### ``def execute``
```ruby
def execute 
    update_hoge
end
```
``update_hoge``メソッドを実行するためのメソッドです。
``Patches::example``クラスのインスタンスを作成して、``execute``メソッドを呼び出すことが今回のパッチ処理のスイッチみたいなものです。


### ``def dry_run?``
```ruby
def dry_run?
    run_mode != "run"
end
```
``run_mode``の引数として渡されてきた値が、**"run"** の時だけ、``false``を返すメソッドです。この判定のおかげで **``dry_run``** が実装できます。

### ``Rails.logger``を活用したログ出力

**logger**はログを出力するために、Railsにあらかじめ用意されている機能です。

Railsのloggerの機能は、``ActiveSupport``クラスを継承した``Logger``クラス（``ActiveSupport::Loggerクラス``）を利用しています。

※``Rails.logger``の後につけるものはログの種類によって分けますが、詳しくはこの記事では言及しません。

```ruby
Rails.logger.info("hogehogeって言います！")

=> hogehogeって言います！ 
```

以下のように、ログを処理の前に用意することで実行対象の件数を確認することができます。

```ruby
Rails.logger.info("id: #{example.id}の処理を開始します")

#以下の処理はrun_modeがrunの時だけ実行される
unless dry_run?
    begin
        #更新処理を実行
        example.status.update!("hoge")
        Rails.logger.info("id: #{example.id}のステータスが#{example.status}に変更されました")
    rescue StandardError => e  
        #エラーが発生した場合
        Rails.logger.error("id: #{example.id}の更新に失敗しました")
    end
end
```
dry_runの時に確認したいのが、処理実行前のこのコードです。
```ruby
Rails.logger.info("id: #{example.id}の処理を開始します")
```
このログは **``unless dry_run?``の真偽に関係なく実行される**ので、本番作業を行う前に処理対象が正しいものかどうかを（期待値通りか）確認するのに活躍してくれます。

run_modeが``run``の時は以下の処理が実行されますが、
```ruby
unless dry_run?
    begin
        #更新処理を実行
        example.status.update!("hoge")
        Rails.logger.info("id: #{example.id}のステータスが#{example.status}に変更されました")
    rescue StandardError => e  
        #エラーが発生した場合
        Rails.logger.error("id: #{example.id}の更新に失敗しました")
    end
end
```
その時もロガーを用意することで処理がうまく行ったかどうかを追跡することができます。

### Rakeタスク

```ruby
task update_example, ['run_mode'] => :environment do |task, args|
    Patches::Example.new(run_mode: args.run_mode).execute
end
```
rakeタスクは以上のように定義したものをコマンドで呼び出すことで実行します。

:::note warn

Rakeタスクでは、引数はすべて**文字列**として渡されます。
:::
なぜかというと、**コマンドラインで入力される値はすべてテキスト形式（文字列）で処理されるからです。** そのため引数も文字列として扱われるわけです。

正しく文字列として認識されるために明示的に``''``クォートで囲ってあげてます。

```ruby
 |task, args|
```
- ``task``：タスクオブジェクトは、タスクに関する情報や操作を提供する**Rake特有のオブジェクト**。タスクを実行する際に、ブロックの引数として渡されるもので、通常、``t``という名前で使用されているが、基本的に使う機会はないっぽいです。
- ``args``：タスクに渡された引数（arguments）を保持するタスク引数オブジェクトで、``args.run_mode``のように指定してあげることで特定の引数を取り出すことができます。

:::note warn
ただし、Rakeは**第１引数にタスクオブジェクト**を受け取り、**第２引数にタスク引数**を受け取ることになっているので、**``args``** にしか興味がなくても``|t,args|``と書いてあげましょう。
:::


### コマンド操作

そして、rakeタスクを書き終わったら、あとはコンソールでコマンド実行してあげるだけです。``rake``に``名前空間:タスク名``そして、引数``[]``を受け取ります。

```ruby:タスクの実行方法(dry_run)
rake patch:update_example[run以外]
rake patch:update_example
# run以外、もしくは指定なしの場合はdry_runで実行されます。
```
``[run]``と渡す以外では``dry_run``が実行されます。


```ruby
Patches::UpdateExample.new(run_mode: "dry_run").execute
```


**実際にrunを実行したいとき**は引数でrun_modeに値を渡してあげることで、明示的に指示してあげます。
```ruby:タスクの実行方法（run）
rake patch:update_example[run]
```
をコマンドで実行すると、``Patches::UpdateExample``クラスのインスタンスに"run"が引数として渡されるので、Rakeタスクファイルで以下の処理ができあがります。
```ruby
Patches::UpdateExample.new(run_mode: "run").execute
```

## 最後に
最後までお読みいただきありがとうございました。
どうやってdry_run処理が実現されているのか、もしくはPatch処理ができているのかを理解する助けになれば幸いです。

