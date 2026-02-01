# PR #2のCodeRabbitレビューコメント（11件のMajor問題）に対応してください。

**プロジェクト**: `/Users/arigatatsuya/Work/git/github.com/sk8metalme/WorkNote`

**日時**: 2026-01-31T16:31:19.124Z

---

PR #2のCodeRabbitレビューコメント（11件のMajor問題）に対応してください。

主な問題：
1. Frontmatter YAMLエスケープ不足（src-tauri/src/services/file_generator.rs:106）
2. save_path検証不足でディレクトリトラバーサル脆弱性（src-tauri/src/services/file_generator.rs:165）
3. CSP設定がnull（src-tauri/tauri.conf.json:22）
4. ErrorDialogのARIA属性不足（src/components/ErrorDialog.svelte:45）
5. 未使用インポート削除（src/lib/tauri-bridge.ts:2）
6. SaveKnowledgeResponse型の不完全（src/lib/tauri-bridge.ts:16）
7. E2Eテストドキュメントの期待値修正（docs/michi/20260131-worknote/test-plan/e2e/knowledge-input-to-push.md:58）
8. テスト戦略のPlaywright→tauri-driver更新（docs/michi/20260131-worknote/test-plan/strategy.md:81）

優先順位：セキュリティ問題（1,2,3）→アクセシビリティ（4）→コード品質（5,6）→ドキュメント（7,8）

PR URL: https://github.com/sk8metalme/WorkNote/pull/2
