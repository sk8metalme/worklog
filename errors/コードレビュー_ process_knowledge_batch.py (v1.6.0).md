# コードレビュー: process_knowledge_batch.py (v1.6.0)

**プロジェクト**: `/Users/arigatatsuya/Work/git/github.com/sk8metalme/ai-agent-setup`

**日時**: 2026-01-31T17:09:31.753Z

---

コードを確認しました。以下にレビュー結果と改善提案をまとめます。

---

## コードレビュー: process_knowledge_batch.py (v1.6.0)

### 良い点

1. **docstring の充実**: 各メソッドに適切な docstring がある
2. **短いオプションのサポート**: `-n`, `-v` の追加でユーザビリティ向上
3. **ヘルプメッセージの詳細化**: Options セクションと例が追加されている
4. **処理完了後のプログレスクリア**: 156-159行目で適切に処理されている

---

### 改善提案

#### 1. 進捗表示ロジックの重複を解消

**問題箇所**: 85-108行目で、rejected/accepted 時の進捗表示条件が異なり、ロジックが分散している。

**現状**:
```python
# rejected 時（96-97行目）
if i % 100 == 0:
    print(f"\rProgress: {i}/{len(candidates)}", end="", flush=True)

# accepted 時（107-108行目）
if i % 100 == 0 or stats["accepted"] <= 10:  # Show first 10 accepted
    print(f"\rProgress: {i}/{len(candidates)}", end="", flush=True)
```

**提案**: ヘルパーメソッドを追加して一貫性を確保する。

```python
PROGRESS_INTERVAL = 100

def _print_progress(self, current: int, total: int) -> None:
    """Print progress indicator in quiet mode."""
    if not self.verbose and current % PROGRESS_INTERVAL == 0:
        print(f"\rProgress: {current}/{total}", end="", flush=True)
```

#### 2. verbose チェックの集約

**問題箇所**: `if self.verbose:` が 9 箇所に散在している（90, 103, 115, 122, 127, 145, 150, 153, 157行目）。

**提案**: ログ出力用ヘルパーメソッドを追加する。

```python
def _log(self, message: str, indent: int = 0) -> None:
    """Print message only in verbose mode."""
    if self.verbose:
        prefix = "   " * indent if indent else ""
        print(f"{prefix}{message}")
```

**使用例**:
```python
# Before
if self.verbose:
    print(f"[{i}/{len(candidates)}] ❌ Rejected (score: {result.score})")

# After
self._log(f"[{i}/{len(candidates)}] ❌ Rejected (score: {result.score})")
```

#### 3. 関数内 import の移動

**問題箇所**: 266行目の `import os` がメソッド内にある。

```python
def _commit_to_git(self, files: list[Path], date: str):
    # ...
    original_dir = Path.cwd()
    import os  # <- ここ

    os.chdir(self.repo_path)
```

**提案**: ファイル冒頭（12-16行目付近）に移動する。

```python
import json
import os  # <- ここに追加
import subprocess
from datetime import datetime
from pathlib import Path
```

#### 4. 戻り値の型アノテーション追加

**問題箇所**: `_commit_to_git` メソッドに戻り値の型アノテーションがない（255行目）。

**提案**:
```python
def _commit_to_git(self, files: list[Path], date: str) -> None:
```

#### 5. マジックナンバーの定数化

**問題箇所**: 進捗表示間隔 `100` がハードコードされている（96, 107行目）。

**提案**: クラス定数として定義する。

```python
class KnowledgeBatchProcessor:
    """Process knowledge candidates in batch."""

    PROGRESS_INTERVAL = 100  # 追加
```

#### 6. 条件の意図を明確化

**問題箇所**: 107行目の条件が複雑。

```python
if i % 100 == 0 or stats["accepted"] <= 10:  # Show first 10 accepted
```

**問題**: 「最初の10件の accepted を表示する」意図だが、実際には `stats["accepted"] <= 10` の間はすべての反復で表示される（rejected も含む）。

**提案**: 意図に沿った条件に修正するか、この特殊条件を削除してシンプルにする。

```python
# シンプルな案: rejected と同じ条件にする
if i % PROGRESS_INTERVAL == 0:
    print(f"\rProgress: {i}/{len(candidates)}", end="", flush=True)
```

---

### 優先度順まとめ

| 優先度 | 項目 | 理由 |
|--------|------|------|
| 高 | 関数内 import の移動 | 慣例違反、保守性向上 |
| 高 | 型アノテーション追加 | プロジェクト規約（明示的な戻り値型） |
| 中 | ヘルパーメソッド追加 | 重複削減、可読性向上 |
| 中 | マジックナンバーの定数化 | 保守性向上 |
| 低 | 進捗表示条件の簡略化 | 挙動の一貫性向上 |

---

修正を実施しますか？優先度の高いものから順に対応できます。
