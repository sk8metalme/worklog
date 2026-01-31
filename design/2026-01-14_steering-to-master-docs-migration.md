# ステアリングドキュメントからマスタードキュメントへの移行

## メタデータ
- **日付**: 2026-01-14
- **カテゴリ**: design
- **タグ**: #michi #documentation #refactoring #directory-structure #master-docs
- **ソース**: Claude Code会話ログ

## コンテキスト

Michiプラグインは、プロジェクト知識を管理するための「ステアリングドキュメント」機能を提供していました。これは `.michi/steering/` ディレクトリに `product.md`, `tech.md`, `structure.md` などを配置する設計でした。

しかし、この設計にはいくつかの問題がありました：
- ディレクトリが隠しディレクトリ（`.michi/`）にあり、発見しにくい
- "steering" という名称が直感的でない
- プラグイン固有のディレクトリに依存している

## 問題

1. **発見性の低さ**: `.michi/steering/` は隠しディレクトリであり、新規参画者が見つけにくい
2. **用語の不明瞭さ**: "steering" という用語が、何を意味するのか直感的でない
3. **プラグイン依存**: Michiプラグインに強く依存した構造

## 解決策

**`docs/master/` ディレクトリへの移行**

### ディレクトリ構造

```
docs/master/
├── README.md
├── core/
│   ├── product.md         # プロダクト概要
│   ├── architecture.md    # アーキテクチャ設計
│   ├── tech-stack.md      # 技術スタック
│   ├── structure.md       # ディレクトリ構造
│   └── sequence.md        # シーケンス図
├── development/
│   ├── api-design.md      # API設計
│   └── data-model.md      # データモデル
├── operations/
│   ├── deployment.md      # デプロイ手順
│   ├── monitoring.md      # 監視設定
│   ├── troubleshooting.md # トラブルシューティング
│   ├── runbook.md         # 運用手順書
│   └── incident-response.md # インシデント対応
├── decisions/
│   ├── _template.md       # 意思決定記録テンプレート
│   └── 001-example.md     # 意思決定記録例
└── glossary.md            # 用語集
```

### コマンド名の変更

- **旧**: `/michi:steering`
- **新**: `/michi:update-master-docs`

### 変更箇所

1. **コマンド名**: `steering.md` → `update-master-docs.md`
2. **ディレクトリパス**:
   - `{{MICHI_DIR}}/steering/` → `{{REPO_ROOT_DIR}}/docs/master/`
3. **削除されたファイル**:
   - `steering-custom.md` （機能を update-master-docs に統合）
   - `templates/multi-repo/steering/` （テンプレートディレクトリ）
4. **更新されたコマンド**（7ファイル）:
   - `create-tasks.md`
   - `create-design.md`
   - `create-requirements.md`
   - `review-dev.md`
   - `review-design.md`
   - `analyze-gap.md`
   - `dev.md`

### 用語の変更

英語テキストも更新：
- "steering context" → "master docs context"
- "steering files" → "master docs files"
- "All custom steering files" → "All custom master docs files"

## 利点

1. **発見性の向上**: `docs/` ディレクトリはプロジェクトルートの標準的な場所
2. **用語の明確化**: "master docs" は「マスタードキュメント」として理解しやすい
3. **プラグイン非依存**: Michiプラグインがなくても、`docs/master/` は有用
4. **階層化**: core, development, operations など、カテゴリごとに整理

## 実装フェーズ

### Phase 1: ファイル名変更
`plugins/michi/commands/michi/steering.md` → `update-master-docs.md`

### Phase 2: steering-custom.md 削除
機能を `update-master-docs.md` に統合して削除

### Phase 3: テンプレートディレクトリ削除
`templates/multi-repo/steering/` を削除

### Phase 4: コマンドファイル更新（7ファイル）
パス参照を `{{MICHI_DIR}}/steering/` → `{{REPO_ROOT_DIR}}/docs/master/` に変更

### Phase 5: ルール参照更新
`steering-principles.md` → `master-docs-principles.md`

### Phase 6: グローバル設定更新
`.claude/CLAUDE.md` からステアリング参照を削除

### Phase 7: README更新
`plugins/michi/README.md` で steering 参照がないことを確認

### Phase 8: 検証
残存する "steering" 参照を grep で確認（英語テキスト含む）

## 検証コマンド

```bash
# パス参照の確認
grep -r "{{MICHI_DIR}}/steering/" plugins/michi/

# テキスト参照の確認
grep -ri "steering" plugins/michi/commands/michi/
```

## 学んだこと

1. **ネーミングの重要性**: 用語は直感的でわかりやすいものを選ぶ
2. **ディレクトリ配置**: 隠しディレクトリではなく、標準的な場所に配置する
3. **段階的な移行**: Phase 1-8 に分けて、確実に変更を進める
4. **徹底的な検証**: grep でテキスト参照まで確認する

## 関連リンク

- [Michiプラグイン](https://github.com/sk8metalme/michi-neo)
- [Architecture Decision Records (ADR)](https://adr.github.io/)
