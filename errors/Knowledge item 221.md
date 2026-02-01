# Knowledge item 221

**プロジェクト**: `/Users/arigatatsuya/Work/git/github.com/sk8metalme/confluence-markdown`

**日時**: 2026-01-31T00:31:18.809Z

---

素晴らしい!要件が完全に明確になりました。では**Step 2: 再利用可能なコンテンツの計画**に進みますね。

## 要件まとめ

以下のスキルを作成します：

**機能**:
- トリガー: 「おはよう」（毎日初回起動時、1日1回のみ）
- 前日分（0時~23:59）のjsonlファイルを解析
- ナレッジ抽出: エラー解決方法、コーディングパターン、便利なコマンド、設計の学び、専門的なドメイン知識、保守・運用作業
- カテゴリ別ディレクトリ + タグメタデータで管理
- 類似度70%以上で重複判定
- mainブランチにプッシュ（コミット: "docs: ユーザ名 YYYY-MM-DD"）

**管理**:
- 発動記録: ~/.claude/daily_knowledge/
- リポジトリ設定: 後で設定可能

## スキル構成の計画

必要なリソースは以下の通りです：

**scripts/**:
1. `extract_knowledge.py` - jsonlからナレッジを抽出（LLM活用）
2. `check_similarity.py` - 類似度判定（embeddings使用）
3. `categorize_knowledge.py` - カテゴリ自動分類
4. `manage_daily_trigger.py` - 1日1回発動管理
5. `push_to_github.py` - GitHubプッシュ処理

**references/**:
1. `knowledge_format.md` - Markdown形式とタグの定義
2. `categories.md` - カテゴリ分類基準

では、**Step 3: スキルの初期化**に進みます。スキル名は`daily-knowledge-sync`としますね。
