# STATUS

最終更新: 2026-11-04（セッション終了時に必ず更新）

## 現在のフェーズ
`coord_conversion.ipynb` を終え、#850 のずれを MintPy 自身の関数で確認した段階。「ずらす」のが MintPy の設計どおりであることもコードで確認済み。upstream #850 へのコメントの準備に進む。

## 前回やったこと
- 2026-11-04: `coord_conversion.ipynb` を実行。`ut.coordinate` を基準に #850 のずれを確認（同じ画素で約 15.7 km、ずらすと約 0.1 km）。本体の 4 か所が同じ補正をしていることをコードで確認 → [`logs/2026-11-04-coord-conversion.md`](../logs/2026-11-04-coord-conversion.md)
- 2026-11-04: チュートリアルを踏まえて MintPy の全体像をおさらいし、wiki に[MintPy とは](https://github.com/rindguitar/MintPy-notes/wiki/MintPy-Overview)を追加
- 2026-10-28: 「西が負・東が正」は GNSS の東・北・上で分けると主に水平の動き（東西の差 4.5 mm/年のうち水平 3.3、上下 0.4）。C で山が負になるのは処理が作る見かけの模様（ディアブロ山で確認）。wiki に[視線方向（LOS）](https://github.com/rindguitar/MintPy-notes/wiki/Line-Of-Sight)を追加

## 次の一手（優先順）
1. upstream #850 にコメントする（調査結果と修正方針。下書きを作ってユーザーが確認。`@ユーザー名` は notes に書かない）
2. 反応を見て `~/MintPy` にブランチを切って実装 → 検証 → pre-commit・単体テスト → PR
3. 余力があれば: 上昇軌道のデータで上下と東西に分ける（`asc_desc2horz_vert.py`）、ERA5 で補正して GNSS と比べる（CDS のキーはユーザーが設定）、`workflows/README.md` の課題
4. 後回し: uv / Docker での環境構築（Docker の公式イメージは #850 の確認用途で先に使うかもしれない）

## 未解決・保留中の問題
- なし

## 直近の決定事項
- 2026-10-19: #850 の前に公式チュートリアルをやる。GIS ツールでの目視も併用、uv / Docker は後回し → 詳細は docs/decisions.md
- 2026-10-11: コントリビュートは内容に踏み込んだもの・既存 issue への貢献を優先し、非推奨対応（警告つぶし）は題材にしない → 詳細は docs/decisions.md
- 2026-10-11: 開発環境は専用の conda 環境 `mintpy`、パッケージ範囲は CI 相当（isce2 なし） → 詳細は docs/decisions.md
