# daily-knowledge-sync スキルの知識抽出基準を厳格化する実装計画を立ててください。

**プロジェクト**: `/Users/arigatatsuya/Work/git/github.com/sk8metalme/ai-agent-setup`

**日時**: 2026-01-31T12:23:35.724Z

---

daily-knowledge-sync スキルの知識抽出基準を厳格化する実装計画を立ててください。

## 背景

現在の問題点（調査結果）：
1. **tool_usesがあれば無条件採用** → ほぼすべてのメッセージが該当
2. **errorsの判定が甘い** → 単語マッチだけで誤判定多発
3. **技術キーワードが広すぎる** → フィルタ効果が薄い
4. **コンテキスト非考慮** → 途中経過も採用

worklogに混入した不要な知識の例：
- 完了報告（「完了だぜ」「タスク完了」など）
- 挨拶・相槌（「なるほど」「よし」）
- 実行ログ・ステータス報告
- システムメッセージ（「This session is being continued」）
- 特定参照（PR番号、コミットハッシュ）

## 改善要件

1. **厳格な除外パターン**を追加
   - 完了報告、挨拶、実行ログを除外
   - システムメッセージを除外
   - 特定参照が多すぎる内容を除外

2. **価値判断基準の改善**
   - tool_usesの存在だけでなく、ツールの種類で判断
   - errorsは実際のスタックトレースを含むかチェック
   - 問題解決パターン（「解決」「原因は」「修正」）を検出

3. **スコアリング制の導入**
   - 二値判定（True/False）→ 0-100スコア
   - 閾値を設定ファイルで調整可能に

4. **設定ファイル化**
   - 除外パターン、キーワード、閾値を外部設定ファイルに

## 対象ファイル

- `/Users/arigatatsuya/.claude/plugins/cache/ai-agent-setup/daily-knowledge-sync/1.3.0/skills/daily-knowledge-sync/scripts/extract_knowledge.py`
- `/tmp/process_knowledge_batch.py`（元は一時ファイルだが、スキルに統合すべき）

## 出力

実装計画（ファイル構成、修正内容、設定ファイルフォーマット）を提案してください。
