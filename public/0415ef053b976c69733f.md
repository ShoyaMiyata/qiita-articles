---
title: 【入門】AWS-CLIのコマンドとAWSサービスを理解する
tags:
  - AWS
  - CLI
  - コマンド
  - aws-cli
  - 基礎
private: false
updated_at: '2024-11-27T10:39:37+09:00'
id: 0415ef053b976c69733f
organization_url_name: lmi-inc
slide: false
ignorePublish: false
---
はじめに
---
仕事で環境構築をする際に、マニュアルに沿ってコマンドを実行していたのですが、aws関連のコマンドについて理解せずに使っていたことに気が付きました。
今後も業務で使える再現性のある知識にしたいと思い、備忘のためにも記事として残させていただきます。

調べるなかで自分の**わからないこと**・**理解の浅いもの**が出るたびにその箇所へフォーカスしながら深ぼっていくスタイルで書いています。ご了承ください。

内容に不備等ある可能性ありますが、ご指摘いただけますと幸いです。


今回は、**AWS CLI**とAWSを利用する入口となる基本的なコマンドを調べました。

- ```aws-sso-util login```
- ```awsume```
- ```aws esc-exec```
- ```aws ec2-login```




前提：AWS CLI
---
AWSコマンドラインインターフェイス

>AWS のサービスを管理するための統合ツールです。ダウンロードおよび設定用の単一のツールのみを使用して、コマンドラインから AWS の複数のサービスを制御し、スクリプトを使用してこれらを自動化することができます。(引用元：https://aws.amazon.com/jp/cli/)

GUIの方が直観的に操作内容を理解しやすいというメリットはあるものの、コマンドラインの場合は入力自体を自動化してしまえば、業務全体を自動化することが可能になり、作業の大幅な効率化につながる。



使い方
-
1. AWS CLIをインストールする
2. AWS CLIの設定
AWSアカウントの認証情報（アクセスキーとシークレットキー）をCLIに提供する作業が含まれる。以下のコマンドを実行して設定を開始する。

        aws configure




```aws-sso-util login``` 
---


分解して理解していきます。

```
aws-sso-util login
```
```aws-sso```
AWS SSOの設定済みプロファイルにログインする際、使用される。このコードを実行すると、ブラウザが開いてSSOの認証ページにリダイレクトされ、ユーザーはそこで認証を行う。SSOとはシングルサインオン(1回のユーザー認証で複数のシステムやサービスにログインできる仕組み)の略です。




```aws-sso-util```
複数のアカウントやロールを簡単に切り替えながら使えるように設定されており、loginコマンドによりこれらのアクセスも容易になっている。



```aws-sso-util login --profile <profile-name>```

**profile-name**にはAWS CLIの設定ファイルに記載されたプロファイル名を指定する。


AWS CLIの設定ファイルは、デフォルトで、
```~/.aws/config```に保存されている。（※Linux/macOSの場合）

    cat ~/.aws/config    
で中身が確認できます。

(Windowsの場合は ```C:\Users\USERNAME\.aws\config```)




```awsume```
---
```
awsume　<profile-name>
```
 **awsume**は、AWS CLI（Command Line Interface）用のツールで、複数のAWSアカウントやIAMロールに簡単に切り替えることができるユーティリティ。
 通常、AWSアカウント間の切り替えは面倒で時間がかかる作業ですが、awsumeを使えば簡略化することができます。
 例えば、開発環境、本番環境、テスト環境など、異なるアカウントで使用する場合、awsumeを使えばスムーズに切り替えることが出来ます。

 **簡単な特徴**
- プロファイルの簡単な切り替えができる
- セッションの自動更新
- クイックアクセスができる

## awsumeを使うには？
### awsumeセットアップに使ったコマンド
- ```brew install pipx```
- ```pipx install awsume```
- ```awsume-configure```
- ```source ~/.bash_profile``` 

実際の流れに沿って、分解して理解していきます。


    brew install pipx
>pipx は、システムにインストール済みの他のパッケージとの間に依存関係の衝突を起こすことなく Python のコマンドラインアプリケーションをインストールし動作させるためのツールです。https://packaging.python.org/ja/latest/key_projects/

**pipx**
Pythonパッケージの実行バイナリを分離された仮想環境にインストールする便利ツール。
Python パッケージをシステム全体にインストールするのではなく、隔離された仮想環境内にインストールして使うためのコマンドラインツール。これにより、Python パッケージの依存関係が他のプロジェクトやシステム全体に影響を与えることなく、安全にインストールして実行することができる。
他のpythonパッケージと依存関係が競合する心配がなく、アンインストールも簡単。

    pipx install awsume

CLI ツール awsume を隔離された**仮想環境**にインストールし、システム全体で使用できるようにするコマンド。
```awsume```  : AWS CLI 用の認証・プロファイル切り替えツール

ここでいう仮想環境は**Python仮想環境**のことで、Python パッケージのみ隔離している。コンテナとは異なる。コンテナはアプリ全体の環境統一、複製を目的としているのに対し、Python仮想環境はパッケージの依存環境の競合を回避する目的で使われる。


以下のメッセージが表示される
```
Warning: the awsume shell script is not
being sourced, please use awsume-configure 
to install the alias
````

以下のコマンドを実行

    awsume-configure

AWS 認証の切り替えツール ```awsume``` を初めてインストールしたときに行う設定コマンド。このコマンドを実行することで、シェルの設定ファイルにエイリアスやオートコンプリートの設定が自動的に追加される。
具体的には、シェルの設定ファイル（~/.bash_profile, ~/.bashrc, ~/.zshenv, ~/.zshrc）に次のエイリアスを追加する。

その後、awsumeコマンドを実行しても同じエラーが出てしまった。

``` awsコマンド実行後
Warning: the awsume shell script is not
being sourced, please use awsume-configure 
to install the alias
```

再度、```aws-configure```を実行してみると、
```
===== Setting up bash =====
Alias already in /Users/username/.bash_profile
Autocomplete script already in /Users/username/.bash_profile

===== Setting up zsh =====
Alias already in /Users/username/.zshenv
Autocomplete script already in /Users/username/.zshenv
Zsh function already in /Users/username/.awsume/zsh-autocomplete/_awsume

===== Finished setting up =====
```

シェルの再読み込みをしてみる。

    source ~/.bash_profile 

成功。これでawsumeコマンドが使えるようになりました。



```aws esc-exec```
---
aws ecs-exec は、**ECS（Elastic Container Service）で稼働しているコンテナに接続**し、コンテナ内で直接コマンドを実行するためのコマンド。デバッグやトラブルシューティングに役立つ。

このコマンド実行後は、タスクの選択、コンテナの選択の順番に進んでいきます。

タスクとは、1つまたは複数のコンテナで構成される単位のことを指します。また詳しくは他の記事で書こうと思います。




```aws ec2-login```
---
aws ec2-login コマンドは、**AWS EC2インスタンスに直接ログイン**して操作するためのコマンド。



ECSとEC2の違い
---
**EC2はElastic Compute Cloud**
AWS(Amazon Web Service)が提供するクラウドサービス。このサービスを利用すると、**仮想サーバー（インスタンス）** をクラウド上に簡単に構築できる。



**ECSはElastic  Container Service**
AWSが提供するサービスの一つだが、これは**コンテナ管理**を目的としている。ECSを使用すると、Dockerコンテナを簡単にデプロイ、管理、スケールできる。つまり、複数のコンテナを効率的に運用するためのサービス。


```aws ec2-login``` はEC2インスタンス全体へのアクセスを目的とし、```aws ecs-exec``` はECSのコンテナに限定したアクセスを提供するコマンド。


ecsとec2をごっちゃにして考えていました。ここをあまり理解しないままマニュアル通りにコマンドを打っていたために、 ```aws ecs-exec```を入力してコンテナサービスを開いた後に```aws ec2-login```を入力するという過ちを犯していました。
これらのコマンドはAWSのもつ異なるサービスに対して使われています。



単語の持つ意味を理解してからだと、上の2つを組み合わせて使おうとは思いません。
一つ一つコマンドを理解しながら使うことが大事になりそうです。
