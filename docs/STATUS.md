# STATUS

最終更新: 2026-10-28（セッション終了時に必ず更新）

## 現在のフェーズ
#850 の再現と原因の特定が済んだ段階。#850 に取り組む前に、公式チュートリアルで `smallbaselineApp.py` の各ステップを理解する（メンターの助言）。

## 前回やったこと
- 2026-10-28: 対流圏補正（height_correlation）と傾きの除去を入れて比べた。GNSS と一番合うのは補正なし（RMSE 0.17 → 0.25 → 0.44 cm/年）。東西の傾きは本物の動きで、補正が本物の動きまで消していた。「開発地が負」は主に地域全体の傾きと地理の重なり → 作業ログに追記
- 2026-10-28: Google Earth Pro で結果を目視。「開発地は負、手付かずの土地は正」というユーザーの観察を確かめると、速度と標高の相関が 0.37（低地 −3.2 mm/年、高地 +2.6〜+2.9 mm/年）。原因は地盤沈下・成層の大気（対流圏補正なし）・地図全体の傾きの混在と推定。対流圏補正を wiki に追加（[対流圏遅延と補正](https://github.com/rindguitar/MintPy-notes/wiki/Tropospheric-Delay)）
- 2026-10-25: チュートリアルの振り返り。各ステップの入力・処理・出力と実際の数字を作業ログにまとめ、wiki に時系列解析のステップ 8 ページを追加 → [`logs/2026-10-25-tutorial-aria.md`](../logs/2026-10-25-tutorial-aria.md)

## 次の一手（優先順）
1. **ユーザー**: ノートブックを保存（Ctrl + S）し、KMZ を作って **Google Earth Pro（無料のデスクトップアプリ。ウェブ版の有料プラン Professional とは別。Earth Versions のページの「Download Earth Pro on desktop」から入れる。ダウンロード提供は 2027-06-25 まで予定）**で見る（ウェブ版は ScreenOverlay（凡例）・NetworkLink・Region に非対応で、凡例が出ず時系列の KMZ は読み込めない。ターミナルでは先に `conda activate mintpy`。`save_kmz.py velocity.h5 -u mm -m maskTempCoh.h5`、`save_kmz_timeseries.py timeseries_demErr.h5 --vel velocity.h5 --tcoh temporalCoherence.h5 --mask maskTempCoh.h5`）
2. 振り返りのまとめ: 比較用の KMZ（`velocity_A_noTropo.kmz` など、色の尺度をそろえたもの）を Google Earth Pro で見比べる。余力があれば ERA5（CDS のキーはユーザーが設定）でも補正して GNSS と比べる。その後、公式の `applications/coord_conversion.ipynb` を手元の FernandinaSenDT128・WellsEnvD2T399 で動かす（#850 に直結）
3. #850 に戻る: QGIS で「書き込まれた緯度経度の標高」と正しい標高を重ねてずれを目で示す → upstream #850 にコメント → 実装・PR
4. 後回し: uv / Docker での環境構築（Docker の公式イメージは #850 の確認用途で先に使うかもしれない）
5. notes の `feature/upstream-issue-survey` を main にマージ

## 未解決・保留中の問題
- なし

## 直近の決定事項
- 2026-10-19: #850 の前に公式チュートリアルをやる。GIS ツールでの目視も併用、uv / Docker は後回し → 詳細は docs/decisions.md
- 2026-10-11: コントリビュートは内容に踏み込んだもの・既存 issue への貢献を優先し、非推奨対応（警告つぶし）は題材にしない → 詳細は docs/decisions.md
- 2026-10-11: 開発環境は専用の conda 環境 `mintpy`、パッケージ範囲は CI 相当（isce2 なし） → 詳細は docs/decisions.md
