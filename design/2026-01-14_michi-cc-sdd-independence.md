# Michiプラグインのcc-sdd依存からの独立化計画

## メタデータ
- **日付**: 2026-01-14
- **カテゴリ**: design
- **タグ**: #michi #cc-sdd #dependency #refactoring #independence #template-internalization
- **ソース**: Claude Code会話ログ

## コンテキスト

Michiプラグインは、cc-sdd（Claude Code Spec-Driven Development）フレームワークの拡張機能として開発されました。14個のMichiコマンドが、kiro（cc-sddの内部名）のコマンドを `@.claude/commands/kiro/*.md` という構文で継承していました。

しかし、この依存関係により、Michiの独立性が損なわれていました。ユーザーはcc-sddとMichiの両方をインストールする必要があり、Michiを単独で使用できませんでした。

## 問題

1. **依存関係**: Michiを使うには、cc-sddのインストールが必須
2. **継承構造**: 14個のコマンドが `@.claude/commands/kiro/*.md` を参照
3. **バージョン管理**: cc-sddのバージョン変更がMichiに影響
4. **独立性の欠如**: Michiを単独のプラグインとして配布できない

## 要件定義

### 目的
**完全独立**: Michiをcc-sdd不要のスタンドアロンプラグインにする

### スコープ
- **対象**: cc-sddのみ（ai-agent-setupは維持）
- **破壊的変更**: 一部許容（メジャーバージョンアップ）
- **移行戦略**: 一括移行（全機能を1回のリリースで）

## 解決策: Template Internalization

### アプローチ

**Template Internalization（テンプレート内部化）** 戦略を採用：

1. `plugins/michi/commands/base/` ディレクトリを新規作成
2. kiroコマンドの内容を base/ にコピー
3. Michiコマンドの参照を `@.claude/commands/kiro/` → `@./base/` に変更

### アーキテクチャ

```
plugins/michi/commands/
├── base/                    # 新規作成（kiroコマンドの内部化）
│   ├── spec-init.md        # 仕様初期化
│   ├── spec-requirements.md # 要件定義
│   ├── spec-design.md      # 設計生成
│   ├── spec-tasks.md       # タスク分割
│   ├── spec-impl.md        # TDD実装
│   ├── spec-status.md      # ステータス表示
│   ├── spec-archive.md     # アーカイブ
│   ├── steering.md         # ステアリング管理
│   ├── steering-custom.md  # カスタムステアリング
│   ├── validate-design.md  # 設計検証
│   ├── validate-impl.md    # 実装検証
│   └── validate-gap.md     # Gap分析
├── michi/                   # 既存（Michi拡張コマンド）
│   ├── spec-init.md        # @./base/spec-init.md を参照
│   ├── spec-requirements.md
│   └── ...（14ファイル）
└── michi-multi-repo/        # 既存（マルチリポコマンド）
    └── ...（6ファイル）
```

### 変更内容

#### 既に内部化済み（2ファイル）
- `templates/claude-agent/commands/kiro/kiro-spec-tasks.md` → `base/spec-tasks.md`
- `templates/claude-agent/commands/kiro/kiro-spec-impl.md` → `base/spec-impl.md`

#### 新規内部化（12ファイル）
1. `spec-init.md` - 仕様初期化
2. `spec-requirements.md` - 要件定義（EARS形式）
3. `spec-design.md` - 設計生成（C4モデル）
4. `spec-status.md` - ステータス表示
5. `spec-archive.md` - アーカイブ処理
6. `steering.md` - ステアリング管理
7. `steering-custom.md` - カスタムステアリング
8. `validate-design.md` - 設計検証
9. `validate-impl.md` - 実装検証
10. `validate-gap.md` - Gap分析
11. （その他、必要に応じて）

#### コマンド参照の更新（14ファイル）
```diff
- @.claude/commands/kiro/spec-init.md
+ @./base/spec-init.md
```

## 実装フェーズ

### Phase 1: インフラ構築（0.5日）
- `plugins/michi/commands/base/` ディレクトリ作成
- 既存テンプレートを base/ に移動
- ディレクトリ構造を検証

### Phase 2: コアロジック移行（2日）

**2.1: 既存テンプレート移行**
- `kiro-spec-tasks.md` → `base/spec-tasks.md`
- `kiro-spec-impl.md` → `base/spec-impl.md`

**2.2: 主要6コマンド内部化**
1. `base/spec-init.md`
2. `base/spec-requirements.md`
3. `base/spec-design.md`
4. `base/spec-status.md`
5. `base/spec-archive.md`
6. `base/steering.md`

**2.3: 検証4コマンド内部化**
1. `base/validate-design.md`
2. `base/validate-impl.md`
3. `base/validate-gap.md`
4. `base/steering-custom.md`

### Phase 3: コマンド移行（1日）
- 14個のMichiコマンドの参照を更新
- 動作検証

### Phase 4: ドキュメント更新（0.5日）
1. `plugins/michi/README.md` - cc-sdd依存を削除
2. `README.md` - インストール手順を更新
3. `CLAUDE.md` - 依存関係を更新
4. `docs/MIGRATION.md` - 移行ガイド作成
5. `CHANGELOG.md` - 破壊的変更を記録

### Phase 5: テスト
- baseコマンド単体テスト
- Michiコマンド参照解決テスト
- E2Eワークフローテスト
- CI/CD検証

## バージョン戦略

- **旧**: v0.19.0（cc-sdd依存）
- **新**: v1.0.0（完全独立）

**理由**: 破壊的変更（cc-sdd削除）のため、メジャーバージョンアップ

## 見積もり

- **合計**: 4.5日
- **Phase 1**: 0.5日
- **Phase 2**: 2日
- **Phase 3**: 1日
- **Phase 4**: 0.5日
- **Phase 5**: 0.5日

## 利点

1. **独立性**: cc-sddなしでMichiを単独使用可能
2. **保守性**: Michiの変更がcc-sddに依存しない
3. **シンプル化**: ユーザーはMichiだけインストールすればよい
4. **バージョン管理**: Michiのバージョンアップが自由

## 代替案（検討済み）

### 1. 完全統合
- cc-sddの全コードをMichiに統合
- **却下理由**: 複雑すぎる、保守が困難

### 2. 部分内部化
- 一部のコマンドだけ内部化
- **却下理由**: 依存関係が残る、不完全

### 3. Template Internalization（採用）
- baseディレクトリを作成して参照を変更
- **採用理由**: 既存アーキテクチャを維持、クリーンな分離

## 学んだこと

1. **依存関係の管理**: 外部依存は最小限に抑える
2. **テンプレート戦略**: 継承よりも、内部化+参照の方が独立性が高い
3. **段階的な移行**: Phase分けで、確実に進める
4. **バージョン戦略**: 破壊的変更はメジャーバージョンアップ

## 検証コマンド

```bash
# kiro参照が残っていないか確認
grep -r "@.claude/commands/kiro" plugins/michi/

# baseディレクトリが作成されたか確認
ls -la plugins/michi/commands/base/

# バージョンが更新されたか確認
cat plugins/michi/.claude-plugin/plugin.json | jq '.version'
```

## 関連リンク

- [Michiプラグイン](https://github.com/sk8metalme/michi-neo)
- [cc-sdd](https://github.com/sk8metalme/cc-sdd)
- [Template Internalization Pattern](https://en.wikipedia.org/wiki/Template_method_pattern)
