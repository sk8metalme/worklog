# daily-knowledge-sync コーヒー豆リファレンス拡充プラン

**プロジェクト**: `/Users/arigatatsuya/Work/git/github.com/sk8metalme/ai-agent-setup`

**日時**: 2026-01-31T06:53:31.491Z

---

Implement the following plan:

# daily-knowledge-sync コーヒー豆リファレンス拡充プラン

## 概要

世界のコーヒー豆データを別ファイルに分離し、候補を大幅に増やします。
各コーヒー豆に苦さ・酸味・香りなどの★評価を追加し、Step 10 ではリファレンスから1つをランダムに選んで紹介する形式に変更します。

## 変更内容

### 1. 新規ファイル作成

**ファイル**: `plugins/daily-knowledge-sync/skills/daily-knowledge-sync/references/coffee_beans.md`

**内容**:
- 世界のコーヒー豆データ（20種類程度）
- 各コーヒー豆に5段階★評価を追加:
  - 苦さ（Bitterness）
  - 酸味（Acidity）
  - 香り（Aroma）
  - コク（Body）
  - 甘み（Sweetness）

**コーヒー豆リスト案**:

| # | 産地 | 銘柄 |
|---|------|------|
| 1 | 🇪🇹 エチオピア | イルガチェフェ |
| 2 | 🇪🇹 エチオピア | シダモ |
| 3 | 🇨🇴 コロンビア | スプレモ |
| 4 | 🇨🇴 コロンビア | エメラルドマウンテン |
| 5 | 🇧🇷 ブラジル | サントス |
| 6 | 🇧🇷 ブラジル | イパネマ |
| 7 | 🇬🇹 グアテマラ | アンティグア |
| 8 | 🇬🇹 グアテマラ | ウエウエテナンゴ |
| 9 | 🇰🇪 ケニア | AA |
| 10 | 🇯🇲 ジャマイカ | ブルーマウンテン |
| 11 | 🇮🇩 インドネシア | マンデリン |
| 12 | 🇮🇩 インドネシア | トラジャ |
| 13 | 🇻🇳 ベトナム | ロブスタ |
| 14 | 🇹🇿 タンザニア | キリマンジャロ |
| 15 | 🇭🇳 ホンジュラス | SHG |
| 16 | 🇨🇷 コスタリカ | タラス |
| 17 | 🇵🇦 パナマ | ゲイシャ |
| 18 | 🇾🇪 イエメン | モカマタリ |
| 19 | 🇷🇼 ルワンダ | キブ |
| 20 | 🇭🇹 ハイチ | ブルーパイン |

### 2. SKILL.md の Step 10 を修正

**変更前**: テーブル形式で8種類のコーヒーを一覧表示

**変更後**: リファレンスファイルから1つを選んで詳細紹介

```markdown
### Step 10: 今日のコーヒー豆 🌍

お疲れさまでした！知識同期が完了しました。

最後に、世界のコーヒーから今日の1杯をご紹介します。
[references/coffee_beans.md](references/coffee_beans.md) からランダムに1つ選んで紹介してください。

**紹介フォーマット例**:

---

☕ **本日のコーヒー: エチオピア イルガチェフェ**

| 項目 | 評価 |
|------|------|
| 苦さ | ★★☆☆☆ |
| 酸味 | ★★★★☆ |
| 香り | ★★★★★ |
| コク | ★★★☆☆ |
| 甘み | ★★★★☆ |

**特徴**: フローラル、フルーティー、紅茶のような上品さ
**おすすめの飲み方**: ハンドドリップ
**ひとこと**: コーヒー発祥の地エチオピアが誇る最高級品種...

---

また明日も、新しい知見と美味しいコーヒーをお楽しみに！☕
```

## 対象ファイル

| ファイル | 変更内容 |
|---------|---------|
| `references/coffee_beans.md` | **新規作成** - 世界のコーヒー豆データ（20種類、★評価付き） |
| `SKILL.md` | Step 10 を修正（リファレンス参照形式に変更） |
| `plugin.json` | バージョン 1.1.0 → 1.1.1 |
| `marketplace.json` | バージョン 1.1.0 → 1.1.1 |

## 実装手順

### Step 1: coffee_beans.md を新規作成

`plugins/daily-knowledge-sync/skills/daily-knowledge-sync/references/coffee_beans.md` に20種類のコーヒー豆データを作成。各コーヒーに以下の情報を含める:
- 産地（国旗絵文字付き）
- 銘柄
- ★評価（苦さ、酸味、香り、コク、甘み）
- 特徴
- おすすめの飲み方
- ひとこと（豆の歴史やトリビア）

### Step 2: SKILL.md の Step 10 を修正

現在のテーブル形式から、リファレンス参照 + 紹介フォーマット形式に変更。

### Step 3: バージョン更新

- `plugin.json`: 1.1.0 → 1.1.1（コンテンツ追加のため PATCH バージョン）
- `marketplace.json`: 1.1.0 → 1.1.1

### Step 4: Git コミット

```bash
git add references/coffee_beans.md SKILL.md plugin.json
git add ../../.claude-plugin/marketplace.json
git commit -m "feat(daily-knowledge-sync): コーヒー豆リファレンスを追加 (v1.1.1)"
git push
```

## 検証方法

1. coffee_beans.md の Markdown フォーマットが正しいことを確認
2. 20種類のコーヒー豆が正しく記載されていることを確認
3. 各コーヒーの★評価が5段階で記載されていることを確認
4. SKILL.md の Step 10 がリファレンス参照形式になっていることを確認
5. plugin.json と marketplace.json のバージョンが一致していることを確認


If you need specific details from before exiting plan mode (like exact code snippets, error messages, or content you generated), read the full transcript at: /Users/arigatatsuya/.claude/projects/-Users-arigatatsuya-Work-git-github-com-sk8metalme-ai-agent-setup/6d123c36-8c90-4cea-a5b3-f159ebd9f88d.jsonl
