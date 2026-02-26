---
title: Zenn記事をGitHub Actionsで自動投稿する方法
date: 2026-02-27 00:00:00 +0900
categories: [JAPANESE]
tags: [TAG]     # TAG names should always be lowercase
---

# Zenn記事をGitHub Actionsで自動投稿する方法

Zennに記事を書くのにブラウザが面倒くさい…そんな悩みを、GitHub push1回で解決！ Zenn CLIとGitHub Actionsを組み合わせれば、VSCodeで執筆→自動公開の夢ワークフローが完成します。この記事でステップバイステップで実装しましょう。 [zenn](https://zenn.dev/ryohei_tanaka/articles/zenn-cli-github-workflow)

## 必要な前提知識

- **Node.js**: v14以上（Zenn CLI用）。nvmで管理推奨。
- **GitHubリポジトリ**: public必須（Zenn連携）。
- **Zennアカウント**: https://zenn.dev/settings/repositories でリポジトリ連携。
- **エディタ**: VSCode + Zenn Editor拡張。
- **npm/yarn**: Zenn CLIインストール用。

macOS/Ubuntu/Windows(WSL)で動作確認。RAM 2GB以上OK。 [zenn](https://zenn.dev/zenn/articles/install-zenn-cli)

## 現在の状況分析

Zenn公式はGitHub連携でpushトリガー自動公開ですが、CLIなしだとプレビューしにくく、Front Matterミス多発。ActionsでCLI実行を自動化すれば、ローカル執筆+CI/CD完璧。 [zenn](https://zenn.dev/korosuke613/articles/zenn-metadata-updater)

## 達成目標

- GitHubリポにarticles/md push → Zenn自動公開。
- プレビュー/予約公開対応。
- 複数記事一括更新。 [zenn](https://zenn.dev/x_color/articles/create-zenn-post-scheduler)

## よくあるGapと課題

- **public repo忘れ**: privateで連携不可。
- **published: false**: Actionsでも非公開扱い。
- **Front Matter無効**: YAMLエラーで失敗。
- **Node未インストール**: CLI動作せず。
- **Secrets未設定**: 認証エラー（Zennは連携で不要だが拡張時用）。 [zenn](https://zenn.dev/zenn/articles/install-zenn-cli)

## 詳細な解決手順

### 1. Zenn-GitHub連携

1. publicリポ作成（例: `my-zenn-blog`）。
2. https://zenn.dev/settings/repositories → リポ追加。
3. `articles/`ディレクトリ作成確認。

**なぜ重要？** これでpush=公開の基盤完成。CLIはプレビュー強化。 [qiita](https://qiita.com/Qiita/items/666e190490d0af90a92b)

### 2. Zenn CLIインストール（ローカル執筆用）

```
mkdir my-zenn-blog && cd my-zenn-blog
npm init -y
npm install zenn-cli --save-dev
npx zenn init  # articles/生成
```

**記事作成**:
```
npx zenn new:article hello-world  # articles/hello-world.md（Front Matter自動）
npx zenn preview  # localhost:3000でZenn風プレビュー
```

**Front Matter例**:
```yaml
---
title: Hello Zenn
published: true  # falseで下書き
topics: [github-actions, zenn-cli]
---
```

**git push**で自動公開！ [qiita](https://qiita.com/Qiita/items/666e190490d0af90a92b)

**トラブルシュート**:
- `init失敗`: `npm install zenn-cli@latest`
- `preview port busy`: Ctrl+C後再実行。
- `連携エラー`: public確認、Zennダッシュ再リンク。 [zenn](https://zenn.dev/zenn/articles/install-zenn-cli)

### 3. GitHub ActionsでCLI自動実行（オプション強化）

Zenn公式push自動ですが、CLI一括/予約対応にActions追加。`.github/workflows/zenn.yml`:

```yaml
name: Zenn Auto Publish

on:
  push:
    branches: [main]
    paths: ['articles/**']

jobs:
  preview-build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npx zenn build  # 静的生成（オプション）
      - name: Upload preview
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./preview/
```

**予約公開拡張**（published_at使用）:
Actionsで`published_at`チェックスクリプト追加。 [zenn](https://zenn.dev/x_color/articles/create-zenn-post-scheduler)

**なぜ重要？** pushでCLI build/preview自動。PRレビューで品質確保。
**Secrets不要**（Zenn連携で認証済み）。

**トラブルシュート**:
- `path無視`: `paths: ['articles/**']`追加。
- Nodeエラー: version固定。
- ビルド失敗: YAMLインデント確認。 [qiita](https://qiita.com/endo-ly/items/077167e4826ce4c8ae16)

## Best Practices（実務経験から）

- **draft branch**: main前は`draft`でPR→main mergeで公開。
- **.gitignore**: `*.swp`, node_modules。
- **Lint/Prettier**: Actionsで`npm run lint`追加。
- **画像**: `articles/images/`相対パス。
- **multi-repo**: 1リポでbooks/articles分離。
- **通知**: Slack/Discord Actions追加。

**⚠️ 注意**: published: true忘れで非公開。push前`zenn preview`必須。

## 次の一歩

- **公式ドキュメント**: https://zenn.dev/zenn/articles/install-zenn-cli [qiita](https://qiita.com/Qiita/items/666e190490d0af90a92b)
- **拡張Actions**: https://github.com/marketplace?query=zenn
- **VSCode連携**: Zenn Editor拡張インストール。
- **高度**: OpenAI+Actionsでタイトル生成（X自動ツイート）。

## まとめと励まし

これでpush=投稿の快適ライフ！ 1記事から始めて、ブログ習慣化を。あなたの知識共有がエンジニア界を豊かにします。がんばって✨

***

*gen by ai, review and edited by me.*
