---
paths:
  - "**/README.md"
---

# README 規約（README編集時のみロードされる）

## このリポジトリの README

| ファイル | 書く内容 |
|---|---|
| `README.md` | リポジトリの目的・構成・運用ルール |
| `logs/README.md` | ログの一覧（新しい順。日付・リンク・1〜2行の要約） |
| `reference/README.md` | ファイルの一覧（ファイル・内容） |

`logs/` `reference/` にファイルを足したら、**同じコミットで一覧表にも1行足す**（別コミットに分けない）。

`~/MintPy` 内の README は upstream のものなので、この規約は適用しない。

## 原則

- **推測で書かない。** 実測結果と既存の記述が食い違ったら、実測が正しい
- 構成・流れを説明するときは図にする。作り方は `.claude/rules/mermaid.md` に従う
- 概念そのものの解説は README に書かず、Wiki へリンクする（`.claude/rules/wiki.md` 参照）
