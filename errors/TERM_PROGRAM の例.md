# TERM_PROGRAM の例

**プロジェクト**: `/Users/arigatatsuya/Work/git/github.com/sk8metalme/ai-agent-setup`

**日時**: 2026-01-31T12:39:42.221Z

---

良い質問ですね。`TERM_PROGRAM` は環境変数で、ターミナルアプリが自動的に設定する値です。

## TERM_PROGRAM の例

| ターミナルアプリ | TERM_PROGRAM の値 |
|----------------|------------------|
| Ghostty | `ghostty` |
| iTerm2 | `iTerm.app` |
| VS Code | `vscode` |
| Terminal.app | `Apple_Terminal` |

## 問題点

これを設定ファイルに書かせるのは確かに複雑ですね。一般ユーザーは `TERM_PROGRAM` という概念を知る必要がないはずです。

## もっとシンプルな提案

**設定ファイルをなくす**のはどうでしょうか？

notify.sh に主要なターミナル（Ghostty、iTerm、VS Code、Terminal）をハードコードして、設定ファイルは「上級ユーザーが新しいターミナルを追加したい時だけ使う」という扱いにします。

これなら：
- ✅ 一般ユーザーは何も設定不要
- ✅ 上級ユーザーだけが設定ファイルをカスタマイズ
- ✅ シンプル

この方向性で良いですか？
