# daily-knowledge-sync 知識抽出ロジック調査結果

**プロジェクト**: `/Users/arigatatsuya/Work/git/github.com/sk8metalme/ai-agent-setup`

**日時**: 2026-01-31T12:22:30.991Z

---

調査結果をまとめます。

---

## daily-knowledge-sync 知識抽出ロジック調査結果

### 1. ファイル構成と役割

| ファイル | 役割 |
|---------|------|
| `extract_knowledge.py` | JSONLから会話ログを抽出（フィルタリングなし） |
| `process_knowledge_batch.py` | **メインの価値判断（`is_valuable`メソッド）** |
| `categorize_knowledge.py` | キーワードベースのカテゴリ分類 |
| `check_similarity.py` | TF-IDF + cosine similarity で重複検出 |

---

### 2. 現在の価値判断基準（`is_valuable` メソッド）

**ファイル**: `/tmp/process_knowledge_batch.py:48-110`

```python
def is_valuable(self, candidate: dict) -> bool:
    # 除外条件
    if len(text) < 50:  # 50文字未満
        return False
    if text.startswith("<command-message>") or text.startswith("<command-name>"):
        return False
    
    # 高価値（無条件採用）
    if errors:      # エラーがあれば採用
        return True
    if tool_uses:   # ツール使用があれば採用
        return True
    
    # 中価値（条件付き採用）
    if role == "assistant" and len(text) > 200:
        # 技術キーワードを含むか
        technical_keywords = ["error", "fix", "implement", "create", "update", ...]
        if any(keyword in text_lower for keyword in technical_keywords):
            return True
    
    return False
```

---

### 3. 現在の基準の問題点

#### 問題1: **tool_uses があれば無条件採用**
- **原因**: `process_knowledge_batch.py:77-78`
  ```python
  if tool_uses:
      return True
  ```
- **問題**: Claude Codeのassistantレスポンスは**ほぼ必ずtool_use**（Read, Bash, Glob等）を含む
- **結果**: 単純なファイル読み込みや検索結果も「知識」として採用

#### 問題2: **errors の判定が甘い**
- **原因**: `extract_knowledge.py:134-135`
  ```python
  if "error" in text_content.lower() or "exception" in text_content.lower():
      errors.append(text_content)
  ```
- **問題**: テキストに "error" という**単語が含まれるだけ**でerrorsリストに追加
- **結果**: 「エラーはありませんでした」のような文も高価値と判定

#### 問題3: **技術キーワードが広すぎる**
```python
technical_keywords = [
    "error", "fix", "implement", "create", "update", "configure",
    "deploy", "test", "debug", "refactor", "optimize", "install",
    "setup", "git", "npm", "python", "bash", "docker", "api", "database"
]
```
- 「ファイルを**create**しました」「**test**を実行します」など日常的な発言がすべてマッチ
- **ほぼすべての開発会話が該当**

#### 問題4: **コンテキストを考慮しない**
- 問題解決の「過程」と「結論」を区別しない
- 試行錯誤の**途中経過**も知識として登録される
- 最終的な解決策のみを抽出する仕組みがない

#### 問題5: **userロールの扱いが不明確**
- `role == "assistant"` の条件があるため、userメッセージは中価値ルートに入らない
- しかし、tool_usesやerrorsがあれば採用される（userメッセージにtool_resultが含まれる場合）

---

### 4. 推奨する改善案

#### 短期改善（is_valuable メソッド修正）

**A. tool_uses の価値判断を追加**
```python
# 低価値ツールを除外
LOW_VALUE_TOOLS = {"Read", "Glob", "Grep", "Bash", "LS"}
MEDIUM_VALUE_TOOLS = {"Edit", "Write", "WebFetch"}
HIGH_VALUE_TOOLS = {"TaskCreate", "Skill"}

valuable_tools = [t for t in tool_uses if t.get("name") not in LOW_VALUE_TOOLS]
if valuable_tools:
    return True
```

**B. errors の判定を厳格化**
```python
# 実際のエラースタックトレースを含むかチェック
REAL_ERROR_PATTERNS = [
    r"Traceback \(most recent",
    r"Error: ",
    r"Exception: ",
    r"failed with exit code",
    r"npm ERR!",
    r"SyntaxError:",
]
has_real_error = any(
    re.search(pattern, str(e)) for e in errors for pattern in REAL_ERROR_PATTERNS
)
```

**C. 技術キーワードを「解決パターン」に変更**
```python
# 問題解決を示す表現
SOLUTION_PATTERNS = [
    r"解決", r"修正しました", r"原因は", r"問題は.*でした",
    r"The fix is", r"The solution is", r"fixed by",
    r"This was caused by", r"The issue was",
    r"以下のように変更", r"changed .* to",
]
```

**D. 結論・まとめ検出を追加**
```python
CONCLUSION_INDICATORS = [
    "まとめ", "結論", "完了しました", "成功しました",
    "Summary:", "In summary", "Done", "Successfully",
    "これで解決", "問題なく動作"
]
has_conclusion = any(ind in text for ind in CONCLUSION_INDICATORS)
```

#### 中期改善（アーキテクチャ変更）

| 改善案 | 説明 | 優先度 |
|-------|------|--------|
| **スコアリング制** | 二値判定→0-100スコア、閾値設定可能 | 高 |
| **会話クラスタリング** | 関連会話をグループ化し、クラスタ単位で判定 | 中 |
| **LLMフィルタリング** | 軽量モデルで「知識として価値があるか」判定 | 低（コスト考慮） |
| **ユーザー確認フロー** | 価値判断をユーザーに確認（Step 9-10で実施）| 高 |

---

### 5. 重複チェックの問題点

**ファイル**: `check_similarity.py`

- **閾値**: 0.7（類似度70%以上で警告）、0.9以上でスキップ
- **問題**: TF-IDFは技術用語が多い文書では高スコアになりやすい
- **改善案**: 
  - タイトル類似度を別途チェック
  - 同一ファイルパス・同一エラーメッセージの検出

---

### 6. 調査結果サマリー

| 観点 | 現状 | 問題の深刻度 |
|------|------|-------------|
| tool_uses判定 | 存在すれば採用 | **高**（ほぼ全メッセージが該当） |
| errors判定 | 単語マッチで採用 | **高**（誤判定多発） |
| 技術キーワード | 汎用的すぎる | **中**（フィルタ効果薄い） |
| コンテキスト考慮 | なし | **中**（途中経過も採用） |
| 重複チェック | 0.9閾値 | **低**（概ね機能している） |
