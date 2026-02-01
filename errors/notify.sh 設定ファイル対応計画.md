# notify.sh 設定ファイル対応計画

**プロジェクト**: `/Users/arigatatsuya/Work/git/github.com/sk8metalme/ai-agent-setup`

**日時**: 2026-01-31T12:17:32.236Z

---

Implement the following plan:

# notify.sh 設定ファイル対応計画

## 概要

`notify.sh` に Ghostty を追加し、ターミナルホワイトリストを外部設定ファイル（`notify.conf`）で管理できるようにする。

## 変更対象ファイル

| ファイル | 操作 |
|---------|------|
| `global/hooks/notify.sh` | 修正 |
| `global/hooks/notify.conf` | 新規作成 |

## 実装内容

### 1. notify.conf（新規作成）

```bash
# notify.conf
# 通知フック設定ファイル

# ===== TERM_PROGRAM → アプリ名マッピング =====
# 形式: "TERM_PROGRAM値:アプリ名"
# 配列の順序は優先度に影響しない（完全一致検索）
TERM_PROGRAM_MAP=(
  "ghostty:Ghostty"
  "iTerm.app:iTerm"
  "vscode:Visual Studio Code"
  "Apple_Terminal:Terminal"
)

# ===== 親プロセス名 → アプリ名マッピング =====
# 形式: "パターン:アプリ名"（ワイルドカード * 使用可能）
PARENT_PROCESS_MAP=(
  "*ghostty*:Ghostty"
  "*iTerm*:iTerm"
  "*Code*:Visual Studio Code"
  "*code*:Visual Studio Code"
  "*Terminal*:Terminal"
)

# ===== pgrep フォールバック（優先順） =====
# 形式: "pgrepパターン:アプリ名"
# 上から順に試行し、最初にマッチしたアプリを activate
PGREP_FALLBACK=(
  "ghostty:Ghostty"
  "iTerm2:iTerm"
  "Visual Studio Code:Visual Studio Code"
  "Code:Visual Studio Code"
)
```

### 2. notify.sh 修正内容

#### 2.1 設定読み込み（protect-branch.sh パターン踏襲）

- スクリプトと同じディレクトリの `notify.conf` を読み込む
- 設定ファイル未存在時はデフォルト値で動作（後方互換性）
- セキュリティ検証（所有者チェック）付き

#### 2.2 検出ロジックの設定ベース化

- `detect_from_term_program()`: TERM_PROGRAM_MAP を走査
- `detect_from_parent_process()`: PARENT_PROCESS_MAP を走査（ワイルドカード対応）
- `detect_from_pgrep()`: PGREP_FALLBACK を**順番に**走査（設定順が優先度）

#### 2.3 Ghostty 対応

| 検出方法 | 値 |
|----------|-----|
| TERM_PROGRAM | `ghostty` |
| 親プロセス名パターン | `*ghostty*` |
| pgrep パターン | `ghostty` |
| AppleScript アプリ名 | `Ghostty` |

## 後方互換性

| シナリオ | 動作 |
|---------|------|
| notify.conf 未存在 | スクリプト内のデフォルト配列で動作 |
| notify.conf 一部記載 | 記載された配列のみ上書き |
| 既存環境 | 動作変更なし（Ghostty 追加のみ） |

## 検証方法

```bash
# 1. 設定ファイルなしでの動作確認
mv ~/.claude/hooks/notify.conf ~/.claude/hooks/notify.conf.bak
# Claude Code でコマンド実行 → 通知 + ウィンドウ最前面化を確認

# 2. 設定ファイルありでの動作確認
mv ~/.claude/hooks/notify.conf.bak ~/.claude/hooks/notify.conf
# Claude Code でコマンド実行 → 通知 + ウィンドウ最前面化を確認

# 3. Ghostty での動作確認（Ghostty ユーザーの場合）
# Ghostty ターミナルから Claude Code を起動してコマンド実行

# 4. フォールバック優先度確認
# PGREP_FALLBACK の順序を変更して動作確認
```


If you need specific details from before exiting plan mode (like exact code snippets, error messages, or content you generated), read the full transcript at: /Users/arigatatsuya/.claude/projects/-Users-arigatatsuya-Work-git-github-com-sk8metalme-ai-agent-setup/219a1d47-dfd9-4143-bbd2-ab59da44f96e.jsonl
