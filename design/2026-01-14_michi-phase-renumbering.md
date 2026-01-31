# Michiプラグインのフェーズ番号体系の整理

## メタデータ
- **日付**: 2026-01-14
- **カテゴリ**: design
- **タグ**: #michi #phase-system #workflow #refactoring
- **ソース**: Claude Code会話ログ

## コンテキスト

Michiプラグインは、Spec-Driven Development（SDD）ワークフローを実装するClaude Codeプラグインです。以前は、フェーズ番号が混在した体系（Phase 0.0, 0.1, 0.2, 0.3-0.4, 0.5, 2, A, B, 0.6-0.7, 4-5）を使用しており、ユーザーにとって非常にわかりにくい状態でした。

## 問題

- フェーズ番号が小数点（0.0, 0.1, 0.2, 0.3-0.4）とアルファベット（A, B）、整数（1, 2, 3）が混在
- フェーズの順序が直感的でない
- ドキュメント全体で一貫性がない
- 新規ユーザーがワークフローを理解しにくい

## 解決策

フェーズ番号を **Phase 1-10 の連番体系** に整理しました。

### マッピング

| 旧フェーズ | 新フェーズ | 名称 |
|-----------|----------|------|
| Phase 0.0 | Phase 1 | 仕様初期化 (/michi:spec-init) |
| Phase 0.1 | Phase 2 | 要件定義 (/michi:spec-requirements) |
| Phase 0.2 | Phase 3 | 設計 (/michi:spec-design) |
| Phase 0.3-0.4 | Phase 4 | テスト計画 (/michi:test-planning) |
| Phase 0.5 | Phase 5 | タスク分割 (/michi:spec-tasks) |
| Phase 2 | Phase 6 | TDD実装 (/michi:spec-impl) |
| Phase A | Phase 6.2/7.1 | 品質監査/PRテスト |
| Phase 3 | Phase 7 | PR/コードレビュー |
| Phase B | Phase 8 | リリース準備 |
| Phase 0.6-0.7 | Phase 9 | JIRA/Confluence連携 |
| Phase 4-5 | Phase 10 | アーカイブ |

### Phase 6 のサブフェーズ

Phase 6（TDD実装）は、品質自動化のため以下のサブフェーズを持ちます：

- Phase 6.1: コンテキストロード
- Phase 6.2: 事前品質監査（License/Version Audit）
- Phase 6.3: TDD実装サイクル（RED→GREEN→REFACTOR）
- Phase 6.4: 事後品質レビュー（Code/Design Review）
- Phase 6.5: 最終検証（Coverage 95%+）
- Phase 6.6: タスク完了マーク
- Phase 6.7: Progress Check
- Phase 6.8: アーカイブ準備

## 影響を受けたファイル

変更が必要だったファイル：
1. `plugins/michi/README.md` - メイン文書
2. `plugins/michi/commands/michi/spec-impl.md` - 最も複雑なコマンド
3. `plugins/michi/commands/michi/test-planning.md` - テスト計画
4. `plugins/michi/commands/michi/michi-spec-impl.md` - 品質自動化
5. `plugins/michi/commands/michi/validate-impl.md` - 実装検証
6. `plugins/michi/commands/michi/spec-design.md` - 設計
7. `plugins/michi/commands/michi/validate-design.md` - 設計検証
8. `plugins/michi/commands/michi/spec-requirements.md` - 要件定義
9. `plugins/michi/commands/michi/pr-resolve.md` - PR解決
10. `templates/multi-repo/steering/multi-repo.md` - マルチリポテンプレート
11. `templates/multi-repo/tests/strategy.md` - テスト戦略
12. `~/.claude/CLAUDE.md` - グローバル設定

## 検証

変更後、以下のコマンドで旧フェーズ番号が残っていないことを確認：

```bash
grep -r "Phase 0\." plugins/michi/ templates/
grep -r "Phase A\|Phase B" plugins/michi/ templates/
```

結果: 0件（すべて変更済み）

## 学んだこと

1. **一貫性の重要性**: フェーズ番号は、プラグイン全体で一貫している必要がある
2. **直感的な命名**: 連番（1, 2, 3...）は、小数点やアルファベットより直感的
3. **サブフェーズの活用**: 複雑なフェーズ（Phase 6）は、サブフェーズ（6.1, 6.2...）で詳細化できる
4. **徹底的な検索**: grep で全ファイルを検索し、見落としがないか確認することが重要

## 関連リンク

- [Michiプラグイン README](https://github.com/sk8metalme/michi-neo/blob/main/plugins/michi/README.md)
- [Spec-Driven Development (SDD)](https://github.com/sk8metalme/cc-sdd)
