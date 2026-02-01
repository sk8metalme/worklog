# Knowledge item 197

**プロジェクト**: `/Users/arigatatsuya/Work/git/github.com/sk8metalme/ai-agent-setup`

**日時**: 2026-01-31T09:40:10.537Z

---

## 1月13日のまとめ

1月13日は、**guardrail-builder スキル**の追加と**Skills path format問題**の修正という、2つの大きな改善が行われました。

### 📌 主な成果（9コミット）

#### 1. Issue #49 - Skills path format 修正 🔧
**コミット**: `c044bb3`（00:18）

**問題点**:
- 全プラグインでskillsパスに `/SKILL.md` を含めていた
- Claude Codeは**ディレクトリパス**を期待（自動的にSKILL.mdを探す仕様）
- 結果: "Unknown skill" エラーが発生

**修正内容**:
```diff
- "skills": ["./skills/changelog/SKILL.md"]
+ "skills": ["./skills/changelog"]
```

**影響を受けたプラグイン（12個）**:
- development-toolkit
- deep-dive
- lang-java-spring, lang-python, lang-php, lang-perl
- jujutsu-workflow
- ci-cd-tools
- oss-compliance
- version-audit
- design-review
- e2e-planning

---

#### 2. guardrail-builder skill 追加 ✨
**コミット**: `dac7560`（00:27）

**新機能**:
- 会話履歴から学習内容を自動抽出
- CLAUDE-guardrail.md に自動追記
- 4カテゴリ分類:
  - プロジェクト仕様
  - エラー対応
  - コーディング規約
  - Tips
- 重複チェック機能
- SessionEndフックによる自動実行（macOS対応）
- macOS通知によるフィードバック

**追加ファイル**:
- `skills/guardrail-builder/SKILL.md`（300+行）
- `scripts/guardrail-builder-hook.sh`

**バージョン**: development-toolkit v1.5.7 → v1.6.0

---

#### 3. CLAUDE-guardrail.md 生成 📝
**コミット**: `016ba1c`（00:36）

guardrail-builderの初回実行で**13件の学習内容**を記録:

**プロジェクト仕様（1件）**:
- Claude Codeプラグインシステムの配布リポジトリ構造
- plugin.jsonとmarketplace.jsonの2箇所定義の必要性

**エラー対応（5件）**:
- ✅ Skillsパスに`/SKILL.md`を含めると"Unknown skill"エラー（最重要）
- SKILL.mdのdescriptionは1行形式（複数行パイプ非対応）
- plugin.jsonのhooks設定は非対応（settings.jsonで手動登録必須）
- marketplace.jsonとplugin.jsonのversion同期必須
- プラグイン名変更時の6箇所一貫性チェック

**コーディング規約（2件）**:
- プラグイン関連ファイル修正時のバージョン更新ルール
- main/masterブランチへの直接push禁止

---

#### 4. グローバル配布への追加 🌍
**コミット**: `8a7227f`（11:46）

**変更内容**:
- guardrail-builder SessionEndフックを `global/hooks/` に移動
- `install-global.sh` で自動インストール
- `settings.template.json` にフック設定を追加

---

#### 5. suggest-claude-md 削除 🗑️
**コミット**: `e522873`（00:40）

guardrail-builderに置き換えられたため削除

---

#### 6. PR #62 マージ 🎉
**コミット**: `69f5ecb`（12:32）

上記の変更をすべて含むPRがマージされました。

**変更サマリー**:
- 8ファイル変更
- +106行追加
- -88行削除

---

### 🎯 1月13日の意義

1. **品質向上**: Skills path format問題の全面修正により、全プラグインが正常に動作
2. **自動化**: guardrail-builderにより、繰り返しミスを防止する仕組みが導入
3. **知識蓄積**: 会話履歴から自動的に学習内容を抽出・記録
4. **配布改善**: グローバルフックの自動インストールで、ユーザー体験が向上

では、最後に今日のコーヒーをご紹介します。
