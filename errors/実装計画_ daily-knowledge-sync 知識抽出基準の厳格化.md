# 実装計画: daily-knowledge-sync 知識抽出基準の厳格化

**プロジェクト**: `/Users/arigatatsuya/Work/git/github.com/sk8metalme/ai-agent-setup`

**日時**: 2026-01-31T12:26:03.199Z

---

コードベースを十分に理解できました。実装計画を詳細に作成します。

---

## 実装計画: daily-knowledge-sync 知識抽出基準の厳格化

### 1. 現状分析

#### 1.1 現在のアーキテクチャ

```
plugins/daily-knowledge-sync/
├── .claude-plugin/
│   └── plugin.json                 # v1.3.0
├── skills/daily-knowledge-sync/
│   ├── SKILL.md                    # ワークフロー定義
│   ├── scripts/
│   │   ├── extract_knowledge.py    # 候補抽出（主要問題箇所）
│   │   ├── categorize_knowledge.py # カテゴリ分類
│   │   ├── check_similarity.py     # 重複チェック
│   │   └── manage_daily_trigger.py # 日次実行管理
│   └── references/
│       ├── categories.md
│       ├── knowledge_format.md
│       └── ...
```

加えて、`/tmp/process_knowledge_batch.py` という一時ファイルに価値判断ロジックが存在する。

#### 1.2 特定された問題点

| 問題 | 場所 | 現在の挙動 |
|-----|-----|----------|
| tool_uses 無条件採用 | `process_knowledge_batch.py` L77-78 | `if tool_uses: return True` |
| errors 判定が甘い | `extract_knowledge.py` L134-135 | 単語マッチのみ（"error" or "exception"） |
| 技術キーワード広すぎ | `process_knowledge_batch.py` L83-104 | "create", "update", "test" など汎用的 |
| 除外パターン不足 | `process_knowledge_batch.py` L68-69 | `<command-message>` と `<command-name>` のみ |
| 二値判定 | `is_valuable()` メソッド | True/False のみ、閾値調整不可 |

### 2. 改善設計

#### 2.1 新規ファイル構成

```
plugins/daily-knowledge-sync/
├── skills/daily-knowledge-sync/
│   ├── scripts/
│   │   ├── extract_knowledge.py         # 改善（フィルタリング強化）
│   │   ├── evaluate_knowledge.py        # 新規: スコアリングエンジン
│   │   ├── process_knowledge_batch.py   # 新規: /tmp から移動・統合
│   │   └── ...
│   ├── config/
│   │   ├── exclusion_patterns.yaml      # 新規: 除外パターン設定
│   │   ├── scoring_rules.yaml           # 新規: スコアリング設定
│   │   └── tool_classifications.yaml    # 新規: ツール分類設定
│   └── ...
```

#### 2.2 設定ファイル設計

##### config/exclusion_patterns.yaml

```yaml
# 除外パターン設定
version: "1.0"

# テキストパターン（正規表現）
text_patterns:
  # 完了報告
  completion:
    - "^(完了|done|finished|completed)"
    - "^タスク(を)?完了"
    - "完了(だぜ|です|しました)"
    - "^(✅|✔|☑)"
  
  # 挨拶・相槌
  greetings:
    - "^(おはよう|こんにちは|こんばんは)"
    - "^(なるほど|了解|わかりました|OK)"
    - "^(よし|さて|では)"
    - "^(ありがとう|thanks)"
  
  # 実行ログ・ステータス
  execution_logs:
    - "^(Running|Executing|Starting|Finished)"
    - "^\\[\\d{2}:\\d{2}:\\d{2}\\]"  # タイムスタンプログ
    - "^(✓|→|•)\\s*\\S+\\s*(done|完了|OK)"
  
  # システムメッセージ
  system_messages:
    - "This session is being continued"
    - "Session restored"
    - "^<system-"
    - "^<environment-"
  
  # 特定参照が多い（3つ以上）
  reference_heavy:
    - "(?:[a-f0-9]{7,40}\\s*){3,}"  # 3つ以上のコミットハッシュ
    - "(?:#\\d+\\s*){3,}"          # 3つ以上のPR/Issue番号

# ロールベースの除外
role_patterns:
  user:
    # ユーザーの短い確認・指示
    - "^(yes|no|ok|うん|はい|いいえ)$"
    - "^\\d+$"  # 数字のみの選択
    - "^[a-z]$"  # 1文字の選択

# 最小文字数（これ未満は除外）
min_text_length: 80

# 最大特定参照数
max_specific_references:
  commit_hashes: 2
  pr_numbers: 3
  file_paths: 10
```

##### config/scoring_rules.yaml

```yaml
# スコアリング設定
version: "1.0"

# スコア閾値
thresholds:
  accept: 60       # この値以上で採用
  maybe: 40        # 40-60 は要検討（デフォルトは採用）
  reject: 40       # この値未満は除外

# ポジティブスコア（加点要素）
positive_scores:
  # エラー解決パターン
  error_resolution:
    has_stack_trace: 20          # 実際のスタックトレースを含む
    has_fix_pattern: 15          # 「原因は」「解決」「修正」パターン
    has_error_explanation: 10    # エラーの説明がある
  
  # 問題解決パターン
  problem_solving:
    problem_solution_pair: 25    # 問題→解決の流れ
    before_after: 15             # 変更前後の比較
    root_cause_analysis: 20      # 根本原因の分析
  
  # ツール使用パターン（ツール種別で重み付け）
  tool_usage:
    high_value_tools: 20         # Edit, Write, Bash
    medium_value_tools: 10       # Read, Grep, Glob
    low_value_tools: 5           # その他
  
  # コンテンツ品質
  content_quality:
    code_example: 15             # コード例を含む
    step_by_step: 10             # ステップバイステップの説明
    explanation_depth: 10        # 詳細な説明（500文字以上）

# ネガティブスコア（減点要素）
negative_scores:
  # 中途半端なコンテンツ
  incomplete:
    in_progress: -15             # 「作業中」「WIP」
    partial_result: -10          # 部分的な結果のみ
  
  # 低価値パターン
  low_value:
    status_report: -20           # ステータス報告のみ
    simple_confirmation: -15     # 単純な確認
    repetitive_content: -10      # 繰り返しコンテンツ
  
  # コンテキスト不足
  lacking_context:
    no_explanation: -15          # 説明なし
    isolated_command: -10        # 孤立したコマンドのみ

# ロールベースの重み
role_weights:
  assistant: 1.2    # アシスタントの回答を重視
  user: 0.8         # ユーザーの質問は少し低め
```

##### config/tool_classifications.yaml

```yaml
# ツール分類設定
version: "1.0"

# 高価値ツール（知識として価値が高い）
high_value:
  - Edit           # ファイル編集
  - Write          # ファイル作成
  - Bash           # コマンド実行
  - WebFetch       # Web情報取得

# 中価値ツール
medium_value:
  - Read           # ファイル読み取り
  - Grep           # 検索
  - Glob           # ファイル検索
  - WebSearch      # Web検索

# 低価値ツール（主にナビゲーション）
low_value:
  - TaskCreate     # タスク管理
  - TaskUpdate     # タスク更新
  - TaskList       # タスク一覧
  - ToolSearch     # ツール検索

# 除外ツール（知識として不要）
excluded:
  - AskUserQuestion  # ユーザーへの質問
  - Skill            # スキル呼び出し

# ツール入力から価値を判定するルール
tool_value_rules:
  Bash:
    high_value_patterns:
      - "git (rebase|cherry-pick|bisect)"  # 高度なGit操作
      - "docker (build|compose)"           # Docker操作
      - "npm (install|run)"                # npm操作
      - "(pytest|jest|vitest)"             # テスト実行
    low_value_patterns:
      - "^(ls|pwd|cd|echo)"               # 基本的なコマンド
      - "cat.*\\| head"                    # 単純なファイル確認
  
  Edit:
    high_value_indicators:
      min_changes: 3                       # 最低3行の変更
      has_meaningful_diff: true            # 意味のある差分
```

#### 2.3 新規クラス設計

##### evaluate_knowledge.py

```python
#!/usr/bin/env python3
"""
Knowledge value evaluation engine with scoring system.
"""

import re
import yaml
from pathlib import Path
from dataclasses import dataclass
from typing import Any


@dataclass
class EvaluationResult:
    """Result of knowledge evaluation."""
    score: int                    # 0-100
    category: str                 # accept, maybe, reject
    reasons: list[str]            # スコアの理由
    positive_factors: dict[str, int]
    negative_factors: dict[str, int]
    
    @property
    def is_valuable(self) -> bool:
        return self.category in ("accept", "maybe")


class KnowledgeEvaluator:
    """Evaluate knowledge candidates using configurable scoring rules."""
    
    def __init__(self, config_dir: str | Path | None = None):
        self.config_dir = Path(config_dir) if config_dir else self._default_config_dir()
        self.exclusion_patterns = self._load_config("exclusion_patterns.yaml")
        self.scoring_rules = self._load_config("scoring_rules.yaml")
        self.tool_classifications = self._load_config("tool_classifications.yaml")
        
        # コンパイル済み正規表現のキャッシュ
        self._compiled_patterns: dict[str, re.Pattern] = {}
    
    def _default_config_dir(self) -> Path:
        return Path(__file__).parent.parent / "config"
    
    def _load_config(self, filename: str) -> dict:
        config_path = self.config_dir / filename
        if config_path.exists():
            with open(config_path, encoding="utf-8") as f:
                return yaml.safe_load(f)
        return {}
    
    def evaluate(self, candidate: dict[str, Any]) -> EvaluationResult:
        """
        Evaluate a knowledge candidate and return a score.
        
        Args:
            candidate: Knowledge candidate with text, role, tool_uses, errors
        
        Returns:
            EvaluationResult with score and categorization
        """
        text = candidate.get("text", "")
        role = candidate.get("role", "")
        tool_uses = candidate.get("tool_uses", [])
        errors = candidate.get("errors", [])
        
        # Step 1: 除外パターンチェック（即座に除外）
        if self._should_exclude(text, role):
            return EvaluationResult(
                score=0,
                category="reject",
                reasons=["Matched exclusion pattern"],
                positive_factors={},
                negative_factors={"exclusion_pattern": -100}
            )
        
        # Step 2: スコア計算
        positive_factors = {}
        negative_factors = {}
        reasons = []
        
        # エラー解決パターンのスコア
        error_score, error_reasons = self._score_error_resolution(text, errors)
        if error_score > 0:
            positive_factors["error_resolution"] = error_score
            reasons.extend(error_reasons)
        
        # 問題解決パターンのスコア
        problem_score, problem_reasons = self._score_problem_solving(text)
        if problem_score > 0:
            positive_factors["problem_solving"] = problem_score
            reasons.extend(problem_reasons)
        
        # ツール使用のスコア
        tool_score, tool_reasons = self._score_tool_usage(tool_uses)
        if tool_score > 0:
            positive_factors["tool_usage"] = tool_score
            reasons.extend(tool_reasons)
        elif tool_score < 0:
            negative_factors["low_value_tools"] = tool_score
        
        # コンテンツ品質のスコア
        quality_score, quality_reasons = self._score_content_quality(text)
        if quality_score > 0:
            positive_factors["content_quality"] = quality_score
            reasons.extend(quality_reasons)
        
        # ネガティブ要素のスコア
        neg_score, neg_reasons = self._score_negative_factors(text)
        if neg_score < 0:
            negative_factors["low_value"] = neg_score
            reasons.extend(neg_reasons)
        
        # ロール重み適用
        role_weight = self.scoring_rules.get("role_weights", {}).get(role, 1.0)
        
        # 合計スコア計算
        base_score = sum(positive_factors.values()) + sum(negative_factors.values())
        final_score = int(min(100, max(0, base_score * role_weight)))
        
        # カテゴリ判定
        thresholds = self.scoring_rules.get("thresholds", {})
        if final_score >= thresholds.get("accept", 60):
            category = "accept"
        elif final_score >= thresholds.get("maybe", 40):
            category = "maybe"
        else:
            category = "reject"
        
        return EvaluationResult(
            score=final_score,
            category=category,
            reasons=reasons,
            positive_factors=positive_factors,
            negative_factors=negative_factors
        )
    
    def _should_exclude(self, text: str, role: str) -> bool:
        """Check if text matches exclusion patterns."""
        patterns = self.exclusion_patterns
        
        # 最小文字数チェック
        min_length = patterns.get("min_text_length", 80)
        if len(text.strip()) < min_length:
            return True
        
        # テキストパターンチェック
        text_patterns = patterns.get("text_patterns", {})
        for category, pattern_list in text_patterns.items():
            for pattern in pattern_list:
                if self._match_pattern(pattern, text):
                    return True
        
        # ロールパターンチェック
        role_patterns = patterns.get("role_patterns", {}).get(role, [])
        for pattern in role_patterns:
            if self._match_pattern(pattern, text):
                return True
        
        return False
    
    def _match_pattern(self, pattern: str, text: str) -> bool:
        """Match a regex pattern against text."""
        if pattern not in self._compiled_patterns:
            self._compiled_patterns[pattern] = re.compile(pattern, re.IGNORECASE | re.MULTILINE)
        return bool(self._compiled_patterns[pattern].search(text))
    
    def _score_error_resolution(self, text: str, errors: list) -> tuple[int, list[str]]:
        """Score error resolution patterns."""
        score = 0
        reasons = []
        rules = self.scoring_rules.get("positive_scores", {}).get("error_resolution", {})
        
        # スタックトレース検出
        if self._has_stack_trace(text):
            score += rules.get("has_stack_trace", 20)
            reasons.append("Contains stack trace")
        
        # 修正パターン検出
        fix_patterns = ["原因は", "解決", "修正", "fix", "solved", "resolved"]
        if any(p in text.lower() for p in fix_patterns):
            score += rules.get("has_fix_pattern", 15)
            reasons.append("Contains fix pattern")
        
        return score, reasons
    
    def _has_stack_trace(self, text: str) -> bool:
        """Check if text contains an actual stack trace."""
        stack_indicators = [
            r"Traceback \(most recent call last\)",
            r"at .+\(.+:\d+\)",            # Java/JS style
            r"File \".+\", line \d+",       # Python style
            r"^\s+at .+ \(.+\)$",           # Node.js style
        ]
        for pattern in stack_indicators:
            if re.search(pattern, text, re.MULTILINE):
                return True
        return False
    
    def _score_problem_solving(self, text: str) -> tuple[int, list[str]]:
        """Score problem-solving patterns."""
        score = 0
        reasons = []
        rules = self.scoring_rules.get("positive_scores", {}).get("problem_solving", {})
        
        # 問題→解決の流れ検出
        problem_patterns = ["問題", "issue", "problem", "エラー"]
        solution_patterns = ["解決", "solution", "対処", "修正"]
        has_problem = any(p in text.lower() for p in problem_patterns)
        has_solution = any(p in text.lower() for p in solution_patterns)
        if has_problem and has_solution:
            score += rules.get("problem_solution_pair", 25)
            reasons.append("Problem-solution pair detected")
        
        return score, reasons
    
    def _score_tool_usage(self, tool_uses: list) -> tuple[int, list[str]]:
        """Score based on tool usage."""
        if not tool_uses:
            return 0, []
        
        score = 0
        reasons = []
        classifications = self.tool_classifications
        
        high_value = set(classifications.get("high_value", []))
        medium_value = set(classifications.get("medium_value", []))
        excluded = set(classifications.get("excluded", []))
        
        for tool in tool_uses:
            tool_name = tool.get("name", "")
            
            if tool_name in excluded:
                continue
            elif tool_name in high_value:
                score += 20
                reasons.append(f"High-value tool: {tool_name}")
            elif tool_name in medium_value:
                score += 10
            else:
                score += 5
        
        return score, reasons
    
    def _score_content_quality(self, text: str) -> tuple[int, list[str]]:
        """Score content quality."""
        score = 0
        reasons = []
        rules = self.scoring_rules.get("positive_scores", {}).get("content_quality", {})
        
        # コード例検出
        if "```" in text or re.search(r"^\s{4,}\S", text, re.MULTILINE):
            score += rules.get("code_example", 15)
            reasons.append("Contains code example")
        
        # 詳細な説明（500文字以上）
        if len(text) >= 500:
            score += rules.get("explanation_depth", 10)
            reasons.append("Detailed explanation")
        
        return score, reasons
    
    def _score_negative_factors(self, text: str) -> tuple[int, list[str]]:
        """Score negative factors."""
        score = 0
        reasons = []
        rules = self.scoring_rules.get("negative_scores", {})
        
        # 進行中パターン
        wip_patterns = ["WIP", "作業中", "TODO", "未完了"]
        if any(p in text for p in wip_patterns):
            score += rules.get("incomplete", {}).get("in_progress", -15)
            reasons.append("Work in progress")
        
        return score, reasons
```

#### 2.4 既存ファイルの改修

##### extract_knowledge.py の改修

主な変更点:
1. `_extract_candidate()` でのエラー検出ロジック改善
2. 事前フィルタリングの追加
3. `KnowledgeEvaluator` との連携

```python
# 追加するメソッド（_extract_candidate の改善）

def _is_actual_error(self, text: str) -> bool:
    """Check if text contains an actual error, not just the word 'error'."""
    # 実際のエラーパターン
    actual_error_patterns = [
        r"Traceback \(most recent call last\)",
        r"Error:\s+.{10,}",              # Error: followed by message
        r"Exception:\s+.{10,}",          # Exception: followed by message
        r"failed with exit code \d+",
        r"FATAL|CRITICAL|ERROR\]",       # ログレベル
    ]
    
    for pattern in actual_error_patterns:
        if re.search(pattern, text, re.IGNORECASE):
            return True
    
    # 単なる言及は除外
    false_positives = [
        r"no error",
        r"error handling",
        r"error message",
        r"if error",
        r"catch error",
    ]
    
    for pattern in false_positives:
        if re.search(pattern, text, re.IGNORECASE):
            # 他の実際のエラーパターンがなければ除外
            return False
    
    return False
```

##### process_knowledge_batch.py のスキルへの統合

`/tmp/process_knowledge_batch.py` を `scripts/process_knowledge_batch.py` に移動し、以下を改修:

1. `is_valuable()` を削除し、`KnowledgeEvaluator.evaluate()` に置き換え
2. 統計情報にスコア分布を追加
3. 設定ファイルからの読み込みに対応

### 3. 実装ステップ

#### Phase 1: 設定ファイルの作成（優先度: 高）

| タスク | 説明 | 推定工数 |
|-------|------|---------|
| 1-1 | `config/` ディレクトリ作成 | 5分 |
| 1-2 | `exclusion_patterns.yaml` 作成 | 30分 |
| 1-3 | `scoring_rules.yaml` 作成 | 30分 |
| 1-4 | `tool_classifications.yaml` 作成 | 20分 |

#### Phase 2: スコアリングエンジンの実装（優先度: 高）

| タスク | 説明 | 推定工数 |
|-------|------|---------|
| 2-1 | `evaluate_knowledge.py` 新規作成 | 2時間 |
| 2-2 | `EvaluationResult` データクラス | 15分 |
| 2-3 | `KnowledgeEvaluator` クラス | 1.5時間 |
| 2-4 | 各スコアリングメソッド実装 | 1時間 |
| 2-5 | ユニットテスト作成 | 1時間 |

#### Phase 3: 既存コードの改修（優先度: 中）

| タスク | 説明 | 推定工数 |
|-------|------|---------|
| 3-1 | `extract_knowledge.py` のエラー検出改善 | 45分 |
| 3-2 | `/tmp/process_knowledge_batch.py` をスキルに統合 | 30分 |
| 3-3 | `is_valuable()` を `evaluate()` に置き換え | 1時間 |
| 3-4 | 統計情報の拡張 | 30分 |

#### Phase 4: 統合とテスト（優先度: 中）

| タスク | 説明 | 推定工数 |
|-------|------|---------|
| 4-1 | SKILL.md の Step 5 更新 | 20分 |
| 4-2 | 統合テスト | 1時間 |
| 4-3 | ドキュメント更新 | 30分 |

#### Phase 5: バージョンアップとリリース（優先度: 高）

| タスク | 説明 | 推定工数 |
|-------|------|---------|
| 5-1 | `plugin.json` のバージョン更新（1.3.0 → 1.4.0） | 5分 |
| 5-2 | `marketplace.json` の version 同期 | 5分 |
| 5-3 | CHANGELOG 更新 | 15分 |

### 4. 依存関係

```
Phase 1 ──→ Phase 2 ──→ Phase 3 ──→ Phase 4 ──→ Phase 5
(設定)      (エンジン)   (改修)      (テスト)     (リリース)
```

### 5. リスクと対策

| リスク | 影響 | 対策 |
|-------|-----|-----|
| 過度なフィルタリング | 価値ある知識の除外 | 閾値を設定ファイルで調整可能に |
| パフォーマンス低下 | 処理時間増加 | 正規表現のキャッシュ、早期除外 |
| 設定ファイルの複雑さ | 運用負荷 | デフォルト値を適切に設定、ドキュメント充実 |
| 後方互換性 | 既存ワークフローへの影響 | `evaluate()` の結果に `is_valuable` プロパティを維持 |

### 6. テスト計画

#### 6.1 除外パターンのテスト

```python
# test_exclusion_patterns.py

def test_exclude_completion_reports():
    evaluator = KnowledgeEvaluator()
    candidates = [
        {"text": "完了だぜ！", "role": "assistant"},
        {"text": "タスク完了しました", "role": "assistant"},
        {"text": "✅ Done!", "role": "assistant"},
    ]
    for c in candidates:
        result = evaluator.evaluate(c)
        assert result.category == "reject"

def test_exclude_greetings():
    evaluator = KnowledgeEvaluator()
    candidates = [
        {"text": "なるほど", "role": "user"},
        {"text": "よし、次へ進もう", "role": "user"},
    ]
    for c in candidates:
        result = evaluator.evaluate(c)
        assert result.category == "reject"
```

#### 6.2 スコアリングのテスト

```python
# test_scoring.py

def test_high_score_for_error_with_stacktrace():
    evaluator = KnowledgeEvaluator()
    candidate = {
        "text": """
        Traceback (most recent call last):
          File "test.py", line 10, in <module>
            raise ValueError("test")
        ValueError: test
        
        原因は引数の型が間違っていました。修正方法は以下の通りです。
        """,
        "role": "assistant",
        "errors": [],
        "tool_uses": []
    }
    result = evaluator.evaluate(candidate)
    assert result.score >= 60
    assert result.category == "accept"

def test_low_score_for_simple_status():
    evaluator = KnowledgeEvaluator()
    candidate = {
        "text": "Running tests... all passed.",
        "role": "assistant",
        "errors": [],
        "tool_uses": []
    }
    result = evaluator.evaluate(candidate)
    assert result.score < 40
    assert result.category == "reject"
```

---

### Critical Files for Implementation

実装に最も重要なファイル:

- `/Users/arigatatsuya/Work/git/github.com/sk8metalme/ai-agent-setup/plugins/daily-knowledge-sync/skills/daily-knowledge-sync/scripts/extract_knowledge.py` - エラー検出ロジックの改善箇所
- `/tmp/process_knowledge_batch.py` - スキルに統合する価値判断ロジック（`is_valuable()` メソッド）
- `/Users/arigatatsuya/Work/git/github.com/sk8metalme/ai-agent-setup/plugins/daily-knowledge-sync/skills/daily-knowledge-sync/SKILL.md` - Step 5 の価値判断基準の更新
- `/Users/arigatatsuya/Work/git/github.com/sk8metalme/ai-agent-setup/plugins/daily-knowledge-sync/.claude-plugin/plugin.json` - バージョン更新（1.3.0 → 1.4.0）
- `/Users/arigatatsuya/Work/git/github.com/sk8metalme/ai-agent-setup/.claude-plugin/marketplace.json` - バージョン同期
