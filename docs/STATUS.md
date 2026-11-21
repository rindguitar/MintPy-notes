# STATUS

最終更新: 2026-11-04（セッション終了時に必ず更新）

## 現在のフェーズ
`coord_conversion.ipynb` を終え、#850 のずれを MintPy 自身の関数で確認した段階。「ずらす」のが MintPy の設計どおりであることもコードで確認済み。upstream #850 へのコメントの準備に進む。

## 前回やったこと
- 2026-11-22: メンターから issue を挙げる許可。upstream の最新（`b9bc6214`）で #850 は未修正・重複なしを確認。新しい issue ではなく #850 にコメントする方針で下書きを作成中
- 2026-11-04: `coord_conversion.ipynb` を実行。`ut.coordinate` を基準に #850 のずれを確認（同じ画素で約 15.7 km、ずらすと約 0.1 km）。本体の 4 か所が同じ補正をしていることをコードで確認 → [`logs/2026-11-04-coord-conversion.md`](../logs/2026-11-04-coord-conversion.md)
- 2026-11-04: チュートリアルを踏まえて MintPy の全体像をおさらいし、wiki に[MintPy とは](https://github.com/rindguitar/MintPy-notes/wiki/MintPy-Overview)を追加

## 次の一手（優先順）
1. **ユーザー**: メンターに下書き（[`logs/2026-11-22-issue-850-comment-draft.md`](../logs/2026-11-22-issue-850-comment-draft.md)）を見てもらう。#850 にコメントするか新しい issue / PR にするかも相談する
2. go が出たら、下書きの「投稿のしかた」の手順で `gh issue comment` で #850 に投稿（Claude が実行）
3. 1 週間ほど反応が無ければ「Fixes #850」の PR を出す。反応があれば方針に合わせて `~/MintPy` にブランチを切って実装 → 検証 → pre-commit・単体テスト → PR
4. 余力があれば: 上昇軌道のデータで上下と東西に分ける（`asc_desc2horz_vert.py`）、ERA5 で補正して GNSS と比べる（CDS のキーはユーザーが設定）、`workflows/README.md` の課題
5. 後回し: uv / Docker での環境構築

## 未解決・保留中の問題
- なし

## 直近の決定事項
- 2026-10-19: #850 の前に公式チュートリアルをやる。GIS ツールでの目視も併用、uv / Docker は後回し → 詳細は docs/decisions.md
- 2026-10-11: コントリビュートは内容に踏み込んだもの・既存 issue への貢献を優先し、非推奨対応（警告つぶし）は題材にしない → 詳細は docs/decisions.md
- 2026-10-11: 開発環境は専用の conda 環境 `mintpy`、パッケージ範囲は CI 相当（isce2 なし） → 詳細は docs/decisions.md
