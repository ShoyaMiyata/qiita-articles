---
title: "Claude Code で Qiita 記事執筆を自動化する方法"
tags:
  - tech
private: true
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

「記事のネタはあるのに、書く時間がない」

エンジニアあるあるではないでしょうか。アイデアをメモしても、いざ執筆するとなるとリサーチ、構成、本文、レビュー...と工数がかかり、下書きフォルダに眠ったままになりがちです。

この記事では、**Claude Code を中心にした記事執筆の自動化ワークフロー**を紹介します。GitHub Issue でネタを管理し、コマンド一つで下書きまで自動生成する仕組みを実際に構築しました。

<!-- IMAGE: A workflow diagram showing: GitHub Issue (idea) → Claude Code (research, write, review) → Qiita draft (markdown file), clean flat design with arrows connecting each step -->
<!-- ALT: GitHub Issueからclaude Codeを経てQiita下書きまでのワークフロー図 -->

## この記事で作るもの

以下の流れを自動化します：

1. **GitHub Issue** にネタをストック（カンバン管理）
2. 「Issue #1 の記事を書いて」と Claude Code に指示
3. Claude Code が自動でリサーチ・構成・執筆・レビュー
4. Qiita CLI の下書き（`private: true`）として保存
5. 人間が最終確認して公開

**ポイント**: 外部 AI API は不要です。Claude Code 自身がすべてのテキスト生成を担当します。

## 前提環境

- [Claude Code](https://claude.ai/code) がインストール済み
- [Qiita CLI](https://github.com/increments/qiita-cli) でリポジトリをセットアップ済み
- [GitHub CLI（gh）](https://cli.github.com/) がインストール・認証済み

Qiita CLI のセットアップがまだの方は、[こちらの記事](https://qiita.com)を参考にしてください。

## アーキテクチャ

```
~/ai/                          ← 自動化スクリプト・設定
  ├── scripts/
  │   └── article-writer.js    ← GitHub操作のCLIラッパー
  ├── skills/
  │   └── article_writing.md   ← Claude Code用のスキル定義
  └── context/
      └── profile.md           ← 著者プロフィール

~/qiita-articles/              ← 記事ストレージ（Qiita CLI管理）
  ├── public/
  │   └── 2026-02-22-xxx.md    ← 生成された下書き
  └── .github/workflows/
      └── publish.yml          ← 自動公開ワークフロー
```

**設計思想**: 自動化の仕組み（`~/ai/`）と記事データ（`~/qiita-articles/`）を分離することで、それぞれ独立して管理・更新できます。

## ステップ 1: GitHub Issue でネタ管理

まず、GitHub Issue をカンバンボードとして使うためのラベルを作成します。

```bash
# カンバン用ラベルを作成
gh label create "ネタ" --color "0e8a16" --repo your-user/qiita-articles
gh label create "ライティング" --color "fbca04" --repo your-user/qiita-articles
gh label create "レビュー中" --color "d93f0b" --repo your-user/qiita-articles
gh label create "完了" --color "0075ca" --repo your-user/qiita-articles
```

記事のアイデアが浮かんだら Issue を作成し、`ネタ` ラベルを付けます：

```bash
gh issue create \
  --title "Claude Code × Qiita CLI で記事執筆を自動化する方法" \
  --body "## メモ\n- 自動化ワークフローの構築手順を解説\n- 初心者向けにステップバイステップで" \
  --label "ネタ,tech" \
  --repo your-user/qiita-articles
```

<!-- IMAGE: A GitHub Projects kanban board with 4 columns labeled ネタ, ライティング, レビュー中, 完了, each containing issue cards, clean minimal UI style -->
<!-- ALT: GitHubカンバンボードの4列構成（ネタ、ライティング、レビュー中、完了） -->

## ステップ 2: CLI ラッパースクリプトの作成

Claude Code 自身がテキスト生成を担当するため、スクリプトは GitHub 操作だけに絞ったシンプルな設計です。

```javascript:~/ai/scripts/article-writer.js
#!/usr/bin/env node

import { writeFile, mkdir } from 'fs/promises';
import { existsSync } from 'fs';
import { join, dirname } from 'path';
import { execSync } from 'child_process';

const QIITA_REPO = process.env.QIITA_REPO || join(process.env.HOME, 'qiita-articles');
const REPO = process.env.GITHUB_REPO || 'your-user/qiita-articles';
const STATUS_LABELS = ['ネタ', 'ライティング', 'レビュー中', '完了'];

function slugify(text) {
  return text.toLowerCase()
    .replace(/[^\w\s-]/g, '')
    .replace(/\s+/g, '-')
    .replace(/--+/g, '-')
    .trim();
}

function getCurrentDate() {
  const d = new Date();
  return `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, '0')}-${String(d.getDate()).padStart(2, '0')}`;
}

// Issue データを JSON で取得
function fetchIssue(num) {
  const json = execSync(
    `gh issue view ${num} --repo ${REPO} --json number,title,body,labels`,
    { encoding: 'utf-8', timeout: 15000 }
  );
  const issue = JSON.parse(json);
  console.log(JSON.stringify({
    number: issue.number,
    title: issue.title,
    body: issue.body || '',
    labels: (issue.labels || []).map(l => l.name)
  }, null, 2));
}

// カンバンステータスを更新
function updateStatus(num, label) {
  for (const sl of STATUS_LABELS) {
    try {
      execSync(`gh issue edit ${num} --repo ${REPO} --remove-label "${sl}"`, {
        encoding: 'utf-8', timeout: 10000, stdio: 'pipe'
      });
    } catch { /* 存在しないラベルは無視 */ }
  }
  execSync(`gh issue edit ${num} --repo ${REPO} --add-label "${label}"`, {
    encoding: 'utf-8', timeout: 10000
  });
  console.log(`✅ Issue #${num} → ${label}`);
}

// stdin から記事を読み込み、Qiita 下書きとして保存
async function saveDraft(num) {
  const json = execSync(
    `gh issue view ${num} --repo ${REPO} --json title,labels`,
    { encoding: 'utf-8', timeout: 15000 }
  );
  const issue = JSON.parse(json);
  const chunks = [];
  for await (const chunk of process.stdin) { chunks.push(chunk); }
  const article = Buffer.concat(chunks).toString('utf-8');

  const date = getCurrentDate();
  const slug = slugify(issue.title);
  const filepath = join(QIITA_REPO, 'public', `${date}-${slug}.md`);

  const contentLabels = (issue.labels || [])
    .map(l => l.name)
    .filter(l => !new Set(STATUS_LABELS).has(l));
  const tags = contentLabels.length > 0
    ? contentLabels.map(l => `  - ${l}`).join('\n')
    : '  - Qiita';

  const frontmatter = `---
title: "${issue.title}"
tags:
${tags}
private: true
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

`;

  const dir = dirname(filepath);
  if (!existsSync(dir)) await mkdir(dir, { recursive: true });
  await writeFile(filepath, frontmatter + article, 'utf-8');
  console.log(filepath);
}

const [cmd, ...args] = process.argv.slice(2);
switch (cmd) {
  case 'fetch': fetchIssue(args[0]); break;
  case 'status': updateStatus(args[0], args[1]); break;
  case 'save': await saveDraft(args[0]); break;
  default:
    console.log('Usage: node article-writer.js <fetch|status|save> <args>');
}
```

3つのコマンドだけのシンプルな構成です：

| コマンド | 用途 |
|---------|------|
| `fetch <番号>` | Issue データを JSON で取得 |
| `status <番号> <ラベル>` | カンバンステータスを更新 |
| `save <番号>` | stdin から記事を読み込み下書き保存 |

## ステップ 3: Claude Code のスキル定義

Claude Code に「記事を書いて」と伝えたときに、自動でワークフローを実行するためのスキル定義を作成します。

```markdown:~/ai/skills/article_writing.md
# スキル: article_writing（記事執筆フルワークフロー）

## ワークフロー

[1] GitHub Issue 取得
    node ~/ai/scripts/article-writer.js fetch <issue-number>
    ↓
[2] Issue ステータスを「ライティング」に変更
    ↓
[3] context/ を参照（profile.md, projects.md）
    ↓
[4] Claude がリサーチ（Web検索含む）
    ↓
[5] Claude が記事構成案を作成
    ↓
[6] Claude が本文を執筆（Qiita Markdown形式）
    ↓
[7] Issue ステータスを「レビュー中」に変更
    ↓
[8] Claude が編集長としてレビュー → 必要なら自己修正
    ↓
[9] 図解挿入箇所にコメントプロンプトを埋め込む
    ↓
[10] 下書きとして保存（private: true）
    echo "記事本文" | node ~/ai/scripts/article-writer.js save <issue-number>
    ↓
[11] SNS 告知文を生成（X / LinkedIn）
    ↓
[12] Issue ステータスを「完了」に変更
```

このスキル定義ファイルを置いておくことで、Claude Code が参照しながらワークフローを順番に実行してくれます。

## ステップ 4: 画像の扱い

記事に図解が必要な箇所には、HTML コメントで画像生成プロンプトを埋め込みます：

```markdown
<!-- IMAGE: A flowchart showing the CI/CD pipeline with GitHub Actions, clean flat design -->
<!-- ALT: CI/CDパイプラインのフローチャート -->
```

レビュー時に人間が以下のいずれかの方法で画像を差し替えます：

- 画像生成 AI（DALL-E、Imagen 等）でプロンプトから生成
- Unsplash / Pixabay 等で適切な画像を検索
- 自分で図を作成（draw.io 等）

差し替え後：

```markdown
![CI/CDパイプラインのフローチャート](./images/cicd-pipeline.png)
```

## 実際に動かしてみる

それでは、実際にワークフローを実行してみましょう。

### 1. Claude Code に指示する

```
$ claude

> Issue #1 の記事を書いて
```

Claude Code がスキル定義を参照し、以下を自動で実行します：

```
✅ Issue #1 のデータを取得
✅ ステータスを「ライティング」に変更
✅ Web検索でリサーチ
✅ 記事構成案を作成
✅ 本文を執筆（3000〜5000文字）
✅ 編集長としてセルフレビュー
✅ 画像プロンプトを埋め込み
✅ 下書きとして保存
✅ SNS告知文を生成
✅ ステータスを「完了」に変更
```

### 2. 下書きを確認する

```bash
cd ~/qiita-articles
npx qiita preview
# ブラウザで http://localhost:8888 にアクセス
```

### 3. レビューして公開

下書きの内容を確認し、必要に応じて修正したら：

```bash
# private: true → false に変更
git add . && git commit -m "publish: 記事タイトル" && git push
```

GitHub Actions が自動で Qiita に公開してくれます。

## なぜ Gemini ではなく Claude Code 単体にしたのか

当初は Gemini API を組み合わせた構成を検討していました。しかし、実際に構築してみると：

- **API レート制限**: 無料枠のリクエスト制限に引っかかりやすい
- **SDK の変更**: `@google/generative-ai` → `@google/genai` への移行が必要だった
- **依存関係の増加**: API キー管理、エラーハンドリング、リトライロジックなど

これらの問題に対して、Claude Code 単体で完結する構成に切り替えたところ：

- **外部依存ゼロ**: API キーの管理が不要（GitHub CLI の認証のみ）
- **コードの簡素化**: スクリプトが 134 行 → 3コマンドのシンプルな CLI に
- **品質の安定**: Claude Code 自身が文脈を理解して書くため、一貫性が高い

結果として、シンプルで壊れにくいワークフローになりました。

## まとめ

この記事では、Claude Code を中心にした記事執筆の自動化ワークフローを紹介しました。

**構成のポイント**:
- GitHub Issue をカンバンボードとしてネタ管理
- Claude Code がリサーチから執筆・レビューまで一気通貫で実行
- Qiita CLI の下書き機能で安全に保存
- 外部 AI API への依存をなくしたシンプルな設計

「記事を書く時間がない」という課題に対して、AIの力を借りて初稿の生成を自動化する。最終的なクオリティは人間がレビューで担保する。この分担が、現時点では最もバランスの良いアプローチだと感じています。

ぜひ、ご自身のワークフローに合わせてカスタマイズしてみてください。

## 参考リンク

- [Claude Code 公式ドキュメント](https://docs.anthropic.com/en/docs/claude-code)
- [Qiita CLI 公式リポジトリ](https://github.com/increments/qiita-cli)
- [GitHub CLI 公式サイト](https://cli.github.com/)
- [MCP（Model Context Protocol）仕様](https://modelcontextprotocol.io/)
