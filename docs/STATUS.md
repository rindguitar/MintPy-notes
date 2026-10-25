# STATUS

最終更新: 2026-10-25（セッション終了時に必ず更新）

## 現在のフェーズ
#850 の再現と原因の特定が済んだ段階。#850 に取り組む前に、公式チュートリアルで `smallbaselineApp.py` の各ステップを理解する（メンターの助言）。

## 前回やったこと
- 2026-10-15: wiki に初学者向けの用語 10 ページを追加（座標・ルックアップテーブル・ジオコーディング・切り出し・マルチルック・コヒーレンス・HDF5・tqdm・griddata・ptime.progressBar）。ドキュメントマップを作り直し
- 2026-10-15: #850 が落ちなくなった理由を特定。MintPy は直っておらず、scipy 1.18 の `griddata` が点 0 個の入力で落ちずに NaN を返すようになっただけ。ずれは 2022 年からあった。notes #8 のメンションを削除
- 2026-10-13: #850 を再現。**もう落ちないが、subset データでは緯度経度が 89 × 11 画素ずれて黙って書き込まれる**ことを標高の独立チェックで確認。原因は lookup の値（切り出し前の座標）をずらさずに使っていること → [`logs/2026-10-13-issue-850-repro.md`](../logs/2026-10-13-issue-850-repro.md)

## 次の一手（優先順）
1. **ユーザー**: 公式推奨の ARIA 版 `~/MintPy-tutorial/workflows/smallbaselineApp_aria.ipynb` をやる
   - 準備: `sudo apt install pv`（展開に使う）、データ 4.9 GB を `~/data/test/SanFranSenDT42.tar.gz` にダウンロード（AWS: `aws --region us-west-2 --no-sign-request s3 cp s3://asf-jupyter-data-west/unavco2022/SanFranSenDT42.tar.gz ~/data/test/`）
   - VS Code でカーネル `mintpy` を選び、上から実行。分からない出力は質問 → 解説し、学んだことはログと wiki に残す
   - 講習会の録画（README にリンク）も参考になる
2. チュートリアルの出力の KMZ を Google Earth で目視（`google_earth` ステップ、`save_kmz.py`）。QGIS（`save_gdal.py` で GeoTIFF）は慣れてから
3. #850 に戻る: QGIS で「書き込まれた緯度経度の標高」と正しい標高を重ねてずれを目で示す → upstream #850 にコメント → 実装・PR
4. 後回し: uv / Docker での環境構築（Docker の公式イメージは #850 の確認用途で先に使うかもしれない）
5. notes の `feature/upstream-issue-survey` を main にマージ

## 未解決・保留中の問題
- なし

## 直近の決定事項
- 2026-10-19: #850 の前に公式チュートリアルをやる。GIS ツールでの目視も併用、uv / Docker は後回し → 詳細は docs/decisions.md
- 2026-10-11: コントリビュートは内容に踏み込んだもの・既存 issue への貢献を優先し、非推奨対応（警告つぶし）は題材にしない → 詳細は docs/decisions.md
- 2026-10-11: 開発環境は専用の conda 環境 `mintpy`、パッケージ範囲は CI 相当（isce2 なし） → 詳細は docs/decisions.md
