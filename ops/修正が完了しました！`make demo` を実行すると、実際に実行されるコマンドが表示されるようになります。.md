# 修正が完了しました！`make demo` を実行すると、実際に実行されるコマンドが表示されるようになります。

**プロジェクト**: `/Users/arigatatsuya/Work/git/github.com/sk8metalme/confluence-markdown`

**日時**: 2026-01-31T03:09:31.417Z

---

修正が完了しました！`make demo` を実行すると、実際に実行されるコマンドが表示されるようになります。

変更内容：
1. **md_to_confluence()** 関数に実行コマンド表示を追加（131行目付近）
2. **confluence_to_md()** 関数に実行コマンド表示を追加（160行目付近）

それぞれ、変換実行前にマゼンタ色で「▼ 実行コマンド」というヘッダーとともに、実際に実行されるコマンドが表示されます。

例：
```
▼ 実行コマンド
conf-md md2conf "examples/sample.md" -o "examples/output/sample_output.xml"

ℹ 変換実行中...
```

動作確認してみますか？
