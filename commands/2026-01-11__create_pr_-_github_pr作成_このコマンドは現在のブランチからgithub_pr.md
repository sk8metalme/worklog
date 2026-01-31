---
title: # Create PR - GitHub PR作成

このコマンドは、現在のブランチからGitHub PRを作成します。

## 実行内容

0
category: commands
tags: [error-fix, error, cli]
date: 2026-01-11
source: conversation
---

# # Create PR - GitHub PR作成

このコマンドは、現在のブランチからGitHub PRを作成します。

## 実行内容

0

## コンテキスト
- 日付: 2026-01-11
- ソース: /Users/arigatatsuya/dotfiles/setup/scripts
- ロール: user

## 問題
```
# Create PR - GitHub PR作成

このコマンドは、現在のブランチからGitHub PRを作成します。

## 実行内容

0. **事前確認フェーズ（ユーザー確認項目を最初にまとめて実施）**
   - ターゲットブランチの確認（main / develop / その他）
   - バージョンファイルの存在確認
   - バージョン更新方針の確認（バージョンファイルがある場合のみ）
     - スキップ / PATCH / MINOR / MAJOR

1. **現在の状態確認**
   - 現在のブランチを確認（main/masterでないことを確認）
   - 未コミットの変更を確認
   - git statusを表示

1.5. **Git追跡対象の検証**

- 未追跡ファイル（untracked files）のスキャン
- ステージング済みファイルのスキャン
- 危険パターン（秘密情報、ビルド成果物、OS生成ファイル、一時ファイル）との照合
- 問題検出時にユーザーへ確認（.gitignoreに追加/個別確認/無視/中止）

2. **変更内容の...
```

## 知見
