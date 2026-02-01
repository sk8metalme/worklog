# 設計提案: notify.conf による設定ファイル管理

**プロジェクト**: `/Users/arigatatsuya/Work/git/github.com/sk8metalme/ai-agent-setup`

**日時**: 2026-01-31T12:05:29.846Z

---

設計を整理しました。以下に提案を示します。

---

## 設計提案: notify.conf による設定ファイル管理

### 1. notify.conf の形式

既存パターン（protect-branch.conf, protect-secrets.conf）の `KEY=VALUE` 形式を踏襲しつつ、マッピング情報は bash 配列で表現します。

```bash
# notify.conf
# 通知フック設定ファイル
# 配置場所: ~/.claude/hooks/notify.conf

# ===== TERM_PROGRAM 環境変数 → アプリ名マッピング =====
# 形式: "TERM_PROGRAM値:アプリ名"
# Claude Code から継承される環境変数に基づく検出（最優先）
TERM_PROGRAM_MAP=(
  "ghostty:Ghostty"
  "iTerm.app:iTerm"
  "vscode:Visual Studio Code"
  "Apple_Terminal:Terminal"
)

# ===== 親プロセス名 → アプリ名マッピング =====
# 形式: "パターン:アプリ名"
# パターンはワイルドカード（*）を使用可能
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

# ===== その他の設定 =====
# デバッグモード（1で有効化）
# DEBUG=1

# デバッグログ出力先
# DEBUG_LOG="${HOME}/.claude/hooks/notify.log"
```

### 2. notify.sh の修正方針

#### 2.1 設定読み込みパターン（既存パターン踏襲）

```bash
# スクリプトのディレクトリを取得
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
CONFIG_FILE="${SCRIPT_DIR}/notify.conf"

# デフォルト設定（後方互換性）
TERM_PROGRAM_MAP=(
  "ghostty:Ghostty"
  "iTerm.app:iTerm"
  "vscode:Visual Studio Code"
  "Apple_Terminal:Terminal"
)

PARENT_PROCESS_MAP=(
  "*ghostty*:Ghostty"
  "*iTerm*:iTerm"
  "*Code*:Visual Studio Code"
  "*code*:Visual Studio Code"
  "*Terminal*:Terminal"
)

PGREP_FALLBACK=(
  "ghostty:Ghostty"
  "iTerm2:iTerm"
  "Visual Studio Code:Visual Studio Code"
  "Code:Visual Studio Code"
)

# デバッグ設定
DEBUG="${DEBUG:-0}"
DEBUG_LOG="${HOME}/.claude/hooks/notify.log"

# 設定ファイルが存在すれば読み込む
if [[ -f "$CONFIG_FILE" ]]; then
    # セキュリティ検証：設定ファイルの所有者が現在のユーザーであることを確認
    if [[ "$(stat -f '%u' "$CONFIG_FILE" 2>/dev/null || stat -c '%u' "$CONFIG_FILE" 2>/dev/null)" != "$(id -u)" ]]; then
        log_debug "WARNING: Config file owner mismatch, skipping load"
    else
        # shellcheck source=/dev/null
        source "$CONFIG_FILE"
    fi
fi
```

#### 2.2 検出ロジックの設定ベース化

```bash
# 方法1: TERM_PROGRAM 環境変数から検出
detect_from_term_program() {
    [[ -z "$TERM_PROGRAM" ]] && return 1
    
    for mapping in "${TERM_PROGRAM_MAP[@]}"; do
        local term_val="${mapping%%:*}"
        local app_name="${mapping##*:}"
        if [[ "$TERM_PROGRAM" == "$term_val" ]]; then
            bring_to_front "$app_name"
            return 0
        fi
    done
    return 1
}

# 方法2: 親プロセスから検出
detect_from_parent_process() {
    local ppid_check=$$
    for _ in {1..10}; do
        ppid_check=$(ps -o ppid= -p "$ppid_check" 2>/dev/null | tr -d ' ')
        [[ -z "$ppid_check" || "$ppid_check" == "1" ]] && break
        
        local pname=$(ps -o comm= -p "$ppid_check" 2>/dev/null)
        for mapping in "${PARENT_PROCESS_MAP[@]}"; do
            local pattern="${mapping%%:*}"
            local app_name="${mapping##*:}"
            # ワイルドカードパターンマッチ
            if [[ "$pname" == $pattern ]]; then
                bring_to_front "$app_name"
                return 0
            fi
        done
    done
    return 1
}

# 方法3: フォールバック - 実行中のアプリを優先順位で検出
detect_from_pgrep() {
    for mapping in "${PGREP_FALLBACK[@]}"; do
        local pgrep_pattern="${mapping%%:*}"
        local app_name="${mapping##*:}"
        if pgrep -x "$pgrep_pattern" > /dev/null 2>&1 || \
           pgrep -f "$pgrep_pattern" > /dev/null 2>&1; then
            bring_to_front "$app_name"
            return 0
        fi
    done
    return 1
}

# メイン検出関数
detect_and_activate() {
    detect_from_term_program && return
    detect_from_parent_process && return
    detect_from_pgrep
}
```

### 3. 後方互換性の担保

| 項目 | 対応方法 |
|------|---------|
| 設定ファイル未存在 | スクリプト内のデフォルト配列で動作 |
| 設定ファイル部分記載 | 記載されていない配列はデフォルト値を使用 |
| 既存の動作 | デフォルト値は現行ハードコードと同等 |
| セキュリティ | 設定ファイル所有者チェック（既存パターン踏襲） |

### 4. Ghostty 対応の追加情報

| 検出方法 | 値 |
|----------|-----|
| TERM_PROGRAM | `ghostty` |
| 親プロセス名パターン | `*ghostty*` |
| pgrep パターン | `ghostty` |
| AppleScript アプリ名 | `Ghostty` |

### 5. ファイル構成（変更後）

```
global/hooks/
├── notify.sh          # 修正（設定ファイル読み込み対応）
├── notify.conf        # 新規作成（ターミナル設定）
├── protect-branch.sh
├── protect-branch.conf
├── protect-secrets.sh
├── protect-secrets.conf
└── statusline.sh
```

### 6. 設計上の考慮点

**利点:**
- 既存パターン（protect-branch.conf）との一貫性
- `source` で読み込むためパース処理不要
- 配列の順序で優先度を明示的に表現
- Ghostty や他のターミナル追加が設定ファイルのみで可能

**代替案（不採用）:**
- JSON 形式: `jq` 必須化は避けたい（notify 目的ならシンプルに）
- パイプ区切り文字列: マッピング情報の表現が困難

**注意点:**
- pgrep の `-x`（完全一致）と `-f`（コマンドライン全体）の使い分け
- Visual Studio Code は `Code` プロセスとして起動するケースがある

---

### Critical Files for Implementation

- `/Users/arigatatsuya/Work/git/github.com/sk8metalme/ai-agent-setup/global/hooks/notify.sh` - 主要な修正対象、設定読み込みロジック追加
- `/Users/arigatatsuya/Work/git/github.com/sk8metalme/ai-agent-setup/global/hooks/protect-branch.sh` - 設定読み込みパターンの参考実装
- `/Users/arigatatsuya/Work/git/github.com/sk8metalme/ai-agent-setup/global/hooks/protect-branch.conf` - .conf ファイル形式の参考
- `/Users/arigatatsuya/Work/git/github.com/sk8metalme/ai-agent-setup/global/hooks/protect-secrets.conf` - 設定ファイル形式の追加参考
