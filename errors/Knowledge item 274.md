# Knowledge item 274

**プロジェクト**: `/Users/arigatatsuya/Work/git/github.com/sk8metalme/WorkNote`

**日時**: 2026-01-31T23:44:13.804Z

---

なるほど、shortcut_manager.rsに関して追加の指摘がありますね。

## 🔍 残りの問題

**Modifiers::SUPERの問題:**
- 現在: macOS で `SUPER` を使用
- 推奨: macOS で `META`（Commandキー）を使用

Tauriのドキュメントでは、`SUPER`はmacOSのCommandキーを意味しますが、より明確に`META`を使う方が推奨されているようです。

どうしますか？

**A)** `SUPER`を`META`に修正してから全てresolve（推奨・より明確）
**B)** 現在の実装で問題ないと判断し、全てresolve（動作は同じ）

どちらにしますか？
