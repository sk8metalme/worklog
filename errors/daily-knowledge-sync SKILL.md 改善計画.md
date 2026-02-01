# daily-knowledge-sync SKILL.md 改善計画

**プロジェクト**: `/Users/arigatatsuya/Work/git/github.com/sk8metalme/ai-agent-setup`

**日時**: 2026-01-31T09:02:44.135Z

---

Implement the following plan:

# daily-knowledge-sync SKILL.md 改善計画

## 問題の振り返り

### 問題1: スクリプトが見つからなかった

| 項目 | 値 |
|------|-----|
| 実行したパス | `.../1.2.0/scripts/manage_daily_trigger.py` |
| 正しいパス | `.../1.2.0/skills/daily-knowledge-sync/scripts/manage_daily_trigger.py` |
| 原因 | `skills/daily-knowledge-sync/` 階層を見落とした |

### 問題2: 環境変数を確認しなかった

**設定済みだった環境変数**:
```
KNOWLEDGE_REPO_PATH: /Users/arigatatsuya/Work/git/github.com/sk8metalme/worklog
KNOWLEDGE_REPO_URL: https://github.com/sk8metalme/worklog
```

**私の行動**: デフォルトパス（~/knowledge-base）だけをチェックし、環境変数を無視した

---

## 改善方針

- Step番号は**整数のみ**使用（0.5などの小数は使わない）
- 新しいStep 1「前提条件の確認」を追加
- 既存のStep 1以降を繰り下げ

---

## 変更後のStep構成

| 旧Step | 新Step | 内容 |
|--------|--------|------|
| Step 0 | Step 0 | コーヒーでひと息（変更なし） |
| (新規) | **Step 1** | **前提条件の確認（環境変数・パス）** |
| Step 1 | Step 2 | 今日実行すべきか確認 |
| Step 2 | Step 3 | リポジトリの設定（初回のみ） |
| Step 3 | Step 4 | 知識候補の抽出 |
| Step 4 | Step 5 | 知識の分析と統合 |
| Step 5 | Step 6 | 知識のカテゴリ分類 |
| Step 6 | Step 7 | 重複のチェック |
| Step 7 | Step 8 | 知識ファイルの作成 |
| Step 8 | Step 9 | GitHubへのコミットとプッシュ |
| Step 9 | Step 10 | 今日実行済みとしてマーク |
| Step 10 | Step 11 | 今日のコーヒー豆 |

---

## 新Step 1の内容

```markdown
### Step 1: 前提条件の確認

スキル実行前に、必要な設定を確認します。

#### 1-1. 環境変数の確認

```bash
echo "KNOWLEDGE_REPO_PATH: ${KNOWLEDGE_REPO_PATH:-未設定}"
echo "KNOWLEDGE_REPO_URL: ${KNOWLEDGE_REPO_URL:-未設定}"
```

- **設定済み** → その値を使用して続行
- **未設定** → Step 3（リポジトリの設定）で設定が必要

#### 1-2. Base Directoryの確認

このスキルのスクリプトは、Base directory からの相対パスで参照します。

```bash
# Base directory（スキル読み込み時に表示される）
SKILL_BASE="<Base directory for this skill の値>"

# スクリプトの存在確認
ls "$SKILL_BASE/scripts/"
```

**重要**: スクリプトパスは `$SKILL_BASE/scripts/xxx.py` です。
プラグインルートからの相対パスではありません。

#### 1-3. リポジトリの状態確認

```bash
REPO_PATH="${KNOWLEDGE_REPO_PATH:-$HOME/knowledge-base}"
if [ -d "$REPO_PATH" ]; then
  echo "✅ リポジトリ存在: $REPO_PATH"
  cd "$REPO_PATH" && git status
else
  echo "❌ リポジトリ未設定 → Step 3 へ"
fi
```
```

---

## 修正対象ファイル

| ファイル | 変更内容 |
|---------|---------|
| `plugins/daily-knowledge-sync/skills/daily-knowledge-sync/SKILL.md` | Step 1追加、既存Step繰り下げ |

---

## 検証方法

1. 改善後のSKILL.mdを読み、Step 1で環境変数が正しく確認されること
2. スクリプトパスがBase directoryからの相対パスとして明記されていること
3. 既存のStep番号が正しく繰り下げられていること

---

## 教訓

1. **Base directoryを活用**: スキル読み込み時に表示されるBase directoryを必ず使う
2. **環境変数を最初に確認**: デフォルト値を仮定する前に既存設定を確認
3. **Step番号は整数のみ**: 0.5などの小数は使わない


If you need specific details from before exiting plan mode (like exact code snippets, error messages, or content you generated), read the full transcript at: /Users/arigatatsuya/.claude/projects/-Users-arigatatsuya-Work-git-github-com-sk8metalme-ai-agent-setup/4f41abf5-f796-4884-9268-2dc25ad5107d.jsonl
