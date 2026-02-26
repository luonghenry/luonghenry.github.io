---
title: Qiita/Zenn CLI + GitHubで記事投稿を自動化する方法
date: 2026-02-27 00:00:00 +0900
categories: [JAPANESE]
tags: [TAG]     # TAG names should always be lowercase
---



# Qiita/Zenn CLI + GitHubで記事投稿を自動化する方法

CLIツールを使ってQiitaやZennに記事を投稿するのは、ブラウザエディタより断然効率的！ 特にGitHub連携でpushするだけで自動投稿できると、バージョン管理も楽チンになります。この記事では、実務で私が愛用するワークフローをステップバイステップで解説します。 [qiita](https://qiita.com/Qiita/items/666e190490d0af90a92b)

## 必要な前提知識

- **Node.js**: v20以上推奨（Qiita CLIはv20必須、Zenn CLIはv14以上）。Homebrewや公式サイトからインストール。 [github](https://github.com/increments/qiita-cli)
- **Git/GitHub**: リポジトリ作成・pushの基本操作。public repoが必要（Zenn連携用）。
- **エディタ**: VSCode推奨。Markdown拡張（Qiita/Zenn syntax highlight）を入れておくと快適。
- **APIトークン**: Qiita/Zennそれぞれ発行（後述）。

Ubuntu/macOS/Windows (WSL)で動作確認済み。RAM 4GB以上あればサクサク。 [zenn](https://zenn.dev/ai4u_shunsuke/articles/zenn-cli-usage)

## 現在の状況分析

ブラウザで書くと、プレビューが遅く、バージョン管理しにくく、複数デバイス同期が面倒。CLI+GitHubならローカルエディタで執筆→pushで即投稿・更新。チームレビューもPRで可能に。 [qiita](https://qiita.com/ryocha12/items/e412306f9e8339d7cffe)

## 達成目標

このガイド終了後、以下のワークフローを実現：
- ローカルで記事執筆・プレビュー。
- GitHub pushでQiita/Zennに自動投稿/更新。
- 投稿済み記事をpullして同期。 [qiita](https://qiita.com/Qiita/items/666e190490d0af90a92b)

## よくあるGapと課題

- **トークン管理ミス**: Secrets未設定で投稿失敗。
- **Front Matterエラー**: YAML構文ミスでpublish拒否（インデント厳守）。
- **private repo不可**: Zennはpublicのみ（QiitaはActionsでOK）。
- **Nodeバージョン**: 古いとコマンドエラー。`nvm`で管理推奨。
- **プレビュー起動忘れ**: ローカル確認なしでpush→typo投稿。 [zenn](https://zenn.dev/nomuraya/articles/26a453b749e53c392639)

## 詳細な解決手順

### 1. Qiita CLIセットアップ

記事管理ディレクトリを作成：
```
mkdir qiita-zenn-blog && cd qiita-zenn-blog
npm init -y
npm install @qiita/qiita-cli --save-dev
npx qiita init  # .gitignore, workflow, config生成
```

**トークン発行**（https://qiita.com/settings/tokens/new）: `read_qiita` + `write_qiita`選択。
```
npx qiita login  # トークン入力
```

**記事作成/プレビュー/投稿**:
```
npx qiita new my-first-post  # public/my-first-post.md生成（Front Matter入り）
npx qiita preview  # http://localhost:8888 でブラウザプレビュー（投稿記事もpull）
npx qiita publish my-first-post  # 投稿/更新
npx qiita publish --all  # 全記事反映
```

**なぜ重要？** Front Matter（title/tags/private）が自動同期。previewでQiita風見た目確認可能。 [qiita](https://qiita.com/Qiita/items/666e190490d0af90a92b)

**トラブルシュート**:
- `login失敗`: トークン再発行、権限確認。
- `publishエラー`: YAMLインデント（スペース2つ）、`---`挟み確認。`--force`で上書き。
- `port in use`: `qiita.config.json`で`"port": 8889`変更。 [github](https://github.com/increments/qiita-cli)

### 2. Zenn CLIセットアップ

同じディレクトリで：
```
npm install zenn-cli --save-dev
npx zenn init  # articles/booksディレクトリ生成
```

**Zenn-GitHub連携**（https://zenn.dev/settings/repositories）: public repo作成後、Zenn側で紐付け。

**記事作成/プレビュー/投稿**:
```
npx zenn new:article my-zenn-post  # articles/my-zenn-post.md
npx zenn preview  # http://localhost:3000 でプレビュー
git add . && git commit -m "Add zenn post" && git push  # pushで自動公開！
```

**なぜ重要？** Zennはpushトリガーで即反映。CLI不要でGitHub直結。 [zenn](https://zenn.dev/zenn/articles/install-zenn-cli)

**トラブルシュート**:
- `preview失敗`: Node v14+確認、`npm install`再実行。
- `連携エラー`: public repo必須、Zennダッシュボードで確認。
- `slug重複`: `--slug unique-slug`指定。 [zenn](https://zenn.dev/ai4u_shunsuke/articles/zenn-cli-usage)

### 3. GitHub Actionsで自動化（Qiita/Zenn共通）

`npx qiita init`で`.github/workflows/publish.yml`生成済み。

**Secrets設定**（https://github.com/[user]/[repo]/settings/secrets/actions）:
- `QIITA_TOKEN`: Qiitaトークン。

**pushで自動投稿**:
```
git add public/articles/ && git commit -m "Update posts" && git push origin main
```
Actionsタブでログ確認。Qiita/Zenn両対応リポで運用可。 [zenn](https://zenn.dev/imudak/articles/github-blog-auto-publish)

**なぜ重要？** 手動publish不要。CI/CDでDRY原則遵守。
**トラブルシュート**: Token無効→再発行。`--force`でActions YAML編集（ignorePublish: false確認）。 [zenn](https://zenn.dev/noraworld/articles/github-to-qiita-by-github-actions)

## Best Practices（実務経験から）

- **1リポ両対応**: `public/`をQiita、`articles/`をZennに分ける。VSCode Workspaceで管理。
- **Branch運用**: `main`(公開)、`draft`(下書き)。PRレビュー必須。
- **Lint追加**: `.prettierrc` + GitHub Actions lint。
- **画像管理**: `public/images/` or `articles/images/`。相対パス使用。
- **ignorePublish: true**で下書き隠蔽。
- **定期pull**: `npx qiita pull` or `git pull`で同期。 [qiita](https://qiita.com/shogo_wada_pro/items/c41d4a6a9b2b2394e407)

**⚠️ 注意**: トークン漏洩厳禁（.gitignore確認）。private記事はQiitaのみ（includePrivate: true）。

## 次の一歩

- **深掘りリソース**:
  - Qiita CLI公式: https://github.com/increments/qiita-cli [qiita](https://qiita.com/Qiita/items/666e190490d0af90a92b)
  - Zenn CLIドキュメント: https://zenn.dev/zenn/articles/install-zenn-cli [zenn](https://zenn.dev/zenn/articles/install-zenn-cli)
  - VSCode拡張: "Qiita Syntax" / "Zenn Editor"
- **拡張アイデア**: textlint統合、AI生成（Claude+Actions）、multi-org対応。
- **コミュニティ**: Qiita/Zenn Discussionsで質問。

## まとめと励まし

CLI+GitHubで投稿が劇的に速くなり、継続執筆のモチベUP！ 今日から1記事試してみて。あなたの技術共有が誰かの助けになります。一緒にブログマスター目指しましょう🚀

***

*gen by ai, review and edited by me.*
