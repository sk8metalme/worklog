# daily-knowledge-sync バッチ処理出力の簡潔化

**プロジェクト**: `/Users/arigatatsuya/Work/git/github.com/sk8metalme/ai-agent-setup`

**日時**: 2026-01-31T16:45:59.138Z

---

Implement the following plan:

# daily-knowledge-sync バッチ処理出力の簡潔化

## 概要

`process_knowledge_batch.py` の出力が詳細すぎる問題を解決する。

**現状の問題:**
```
[1/2878] ❌ Rejected (score: 0)
   Reason: Text too short (< 80 chars)
[2/2878] ❌ Rejected (score: 0)
   ...（2878回繰り返し）
```

## 実装方針

**A案: `--quiet` オプションの追加（推奨）**

デフォルトで簡潔な出力にし、`--verbose` で詳細出力を有効化。

### 変更後の出力イメージ

**デフォルト（quiet モード）:**
```
Processing 2878 candidates for 2026-01-31...
Progress: 2878/2878 completed

==================================================
SUMMARY
==================================================
Total candidates: 2878
  ✅ Accepted:  5
  ❌ Rejected:  2860
  ⚠️  Duplicates: 13
  📝 Created:   5
```

**`--verbose` オプション付きの場合:**
```
[1/2878] ❌ Rejected (score: 0)
   Reason: Text too short (< 80 chars)
...（従来通りの詳細出力）
```

## 変更対象ファイル

| ファイル | 変更内容 |
|----------|----------|
| `plugins/daily-knowledge-sync/skills/daily-knowledge-sync/scripts/process_knowledge_batch.py` | `--verbose` オプション追加、出力制御 |
| `plugins/daily-knowledge-sync/.claude-plugin/plugin.json` | バージョン更新 (1.5.1 → 1.6.0) |
| `.claude-plugin/marketplace.json` | バージョン同期 |

## 実装詳細

### 1. process_knowledge_batch.py の変更

```python
# __init__ に verbose パラメータ追加
def __init__(self, repo_path, similarity_threshold=0.7, dry_run=False, verbose=False):
    self.verbose = verbose

# process_candidates メソッド内の出力を条件付きに
if self.verbose:
    print(f"[{i}/{len(candidates)}] ❌ Rejected (score: {result.score})")
    if result.excluded_by:
        print(f"   Reason: {result.excluded_by}")
else:
    # 100件ごとに進捗表示
    if i % 100 == 0:
        print(f"\rProgress: {i}/{len(candidates)}", end="", flush=True)

# 引数パーサーに --verbose 追加
parser.add_argument('--verbose', '-v', action='store_true',
                    help='Show detailed output for each candidate')
```

### 2. バージョン更新

- plugin.json: `"version": "1.5.1"` → `"version": "1.6.0"`
- marketplace.json: 同期

## 検証方法

```bash
# 1. 修正後のスクリプトをテスト（デフォルト＝簡潔）
python plugins/daily-knowledge-sync/skills/daily-knowledge-sync/scripts/process_knowledge_batch.py \
  /tmp/knowledge_candidates_2026-01-31.json \
  /path/to/repo \
  --dry-run

# 2. verbose モードをテスト
python ... --verbose

# 3. プラグインを再インストールして動作確認
/plugin uninstall daily-knowledge-sync
/plugin install daily-knowledge-sync@ai-agent-setup
```

## リスク

- 既存のワークフローに影響なし（デフォルト動作が変わるのみ）
- 後方互換性あり（`--verbose` で従来の動作を再現可能）


If you need specific details from before exiting plan mode (like exact code snippets, error messages, or content you generated), read the full transcript at: /Users/arigatatsuya/.claude/projects/-Users-arigatatsuya-Work-git-github-com-sk8metalme-ai-agent-setup/51b18d02-7d14-4680-aecb-8fb0b575c224.jsonl
