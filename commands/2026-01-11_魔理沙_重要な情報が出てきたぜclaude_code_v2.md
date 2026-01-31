---
title: **魔理沙**: 重要な情報が出てきたぜ！Claude Code v2
category: commands
tags: [command]
date: 2026-01-11
source: conversation
---

# **魔理沙**: 重要な情報が出てきたぜ！Claude Code v2

## コンテキスト
- 日付: 2026-01-11
- ソース: github/com/sk8metalme/ai/agent/setup
- ロール: assistant

## トピック
**魔理沙**: 重要な情報が出てきたぜ！Claude Code v2.1.0 で Skills の仕様が大きく変わったんだな！

**霊夢**: 次は現在の環境を調査して、この機能が使えるか確認するわ。

---

**現在の理解（更新3）**:

- **出典**: GitHub - anthropics/claude-code commit 870624f (v2.1.0 CHANGELOG)
- **確認済み事実**:
  - v2.1.0 で Skills がスラッシュコマンドメニューに**デフォルト表示**される
  - `user-invocable: false` で非表示化可能
  - `/skills/` ディレクトリの Skills が自動で `/` メニューに表示
  - ホットリロード対応（セッション再起動不要）
  
- **Commands vs Skills（v2.1.0以降）**:
  - **Commands**: `/commands/` に配置、従来通り
  - **Skills**: `/skills/` に配置、**デフォルトでスラッシュコマンド表示**
  - 違いは配置場所のみ？機能的には同じ？
  
- **不明点**:
  - 現在の Claude Code バージョンは v2.1.0 以降か？
  - commands と skills の機能的な違いは本当にないのか？
  - dd プラグインで commands を削除して skills だけにして問題ないか？

---

## 知見
