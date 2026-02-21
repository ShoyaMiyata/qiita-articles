# Qiita Articles

このリポジトリは、Qiitaに投稿する記事をGitで管理するためのものです。

## 概要

[Qiita CLI](https://github.com/increments/qiita-cli)を使用して、Qiitaの記事をローカルで執筆・管理し、GitHubと連携して自動公開する仕組みを構築しています。

## 構成

- `public/` - 公開記事のMarkdownファイル
- `qiita.config.json` - Qiita CLIの設定ファイル
- `.github/workflows/publish.yml` - 記事の自動公開ワークフロー

## セットアップ

### 前提条件

- Node.js
- Qiitaアカウント
- Qiitaアクセストークン

### インストール

```bash
npm install -g @qiita/qiita-cli
```

### 初期設定

1. Qiitaアクセストークンを取得（[Qiita設定ページ](https://qiita.com/settings/tokens)から）
2. リポジトリのSecretsに`QIITA_TOKEN`を設定

## 使い方

### ローカルプレビュー

```bash
npx qiita preview
```

ブラウザで http://localhost:8888 にアクセスすると記事のプレビューが確認できます。

### 新規記事の作成

```bash
npx qiita new 記事のタイトル
```

`public/`ディレクトリに新しい記事ファイルが生成されます。

### 記事の公開

`main`または`master`ブランチにプッシュすると、GitHub Actionsが自動的に記事をQiitaに公開します。

```bash
git add .
git commit -m "記事を追加"
git push origin main
```

## 設定

`qiita.config.json`で以下の設定が可能です：

- `includePrivate`: 限定共有記事を含めるか（デフォルト: false）
- `host`: プレビューサーバーのホスト（デフォルト: localhost）
- `port`: プレビューサーバーのポート（デフォルト: 8888）

## 参考リンク

- [Qiita CLI公式ドキュメント](https://github.com/increments/qiita-cli)
- [Qiita](https://qiita.com/)
