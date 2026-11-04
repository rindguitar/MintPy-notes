# STATUS

最終更新: 2026-11-04（セッション終了時に必ず更新）

## 現在のフェーズ
公式チュートリアル（ARIA 版）の実行と振り返りが一区切りついた段階。次は `coord_conversion.ipynb` でレーダー座標の話を補い、#850 に戻る。

## 前回やったこと
- 2026-11-04: チュートリアルを踏まえて MintPy の全体像をおさらいし、wiki に[MintPy とは](https://github.com/rindguitar/MintPy-notes/wiki/MintPy-Overview)を追加
- 2026-10-28: 「西が負・東が正」は GNSS の東・北・上で分けると主に水平の動き（東西の差 4.5 mm/年のうち水平 3.3、上下 0.4）。C で山が負になるのは処理が作る見かけの模様（ディアブロ山で確認）。wiki に[視線方向（LOS）](https://github.com/rindguitar/MintPy-notes/wiki/Line-Of-Sight)を追加
- 2026-10-28: 対流圏補正（height_correlation）と傾きの除去を入れて比べた。GNSS と一番合うのは補正なし（RMSE 0.17 → 0.25 → 0.44 cm/年）。東西の傾きは本物の動きで、補正が本物の動きまで消していた。「開発地が負」は主に地域全体の傾きと地理の重なり → 作業ログに追記

## 次の一手（優先順）
1. 公式の `applications/coord_conversion.ipynb`（レーダー座標 ⇔ 地理座標、`ut.coordinate`）を手元の FernandinaSenDT128・WellsEnvD2T399 で動かす（#850 に直結）
2. #850 に戻る: ずれを目で示す → upstream #850 にコメント → 実装・PR（再現手順は notes #5）
3. notes の `feature/upstream-issue-survey` を main にマージ
4. 余力があれば: 上昇軌道のデータで上下と東西に分ける（`asc_desc2horz_vert.py`）、ERA5 で補正して GNSS と比べる（CDS のキーはユーザーが設定）、`workflows/README.md` の課題
5. 後回し: uv / Docker での環境構築（Docker の公式イメージは #850 の確認用途で先に使うかもしれない）

## 未解決・保留中の問題
- なし

## 直近の決定事項
- 2026-10-19: #850 の前に公式チュートリアルをやる。GIS ツールでの目視も併用、uv / Docker は後回し → 詳細は docs/decisions.md
- 2026-10-11: コントリビュートは内容に踏み込んだもの・既存 issue への貢献を優先し、非推奨対応（警告つぶし）は題材にしない → 詳細は docs/decisions.md
- 2026-10-11: 開発環境は専用の conda 環境 `mintpy`、パッケージ範囲は CI 相当（isce2 なし） → 詳細は docs/decisions.md
