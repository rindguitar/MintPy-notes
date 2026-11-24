# STATUS

最終更新: 2026-11-24（セッション終了時に必ず更新）

## 現在のフェーズ
#850 の修正を `~/MintPy` の手元で終え（2 コミット、検証・単体テスト・pre-commit 済み）、PR 本文と #850 へのコメントの下書きをメンターに確認してもらう段階。fork への push・PR・コメントはまだしていない。

## 前回やったこと
- 2026-11-24: `~/MintPy` のブランチ `lookup_geo2radar_subset` に 2 コミット（切り出し量を引く修正、tqdm → `ptime.progressBar`）。300 画素での距離の中央値 15.4 km → 0.07 km。PR 本文と短くした #850 コメントの下書きを作り、notes #5 で共有 → [`logs/2026-11-24-issue-850-pr-draft.md`](../logs/2026-11-24-issue-850-pr-draft.md)。残る NaN の原因（`split_range()` の丸め）を見つけ notes #9 に記録
- 2026-11-22: メンターから issue を挙げる許可。upstream の最新（`b9bc6214`）で #850 は未修正・重複なしを確認。新しい issue ではなく #850 にコメントする方針に
- 2026-11-04: `coord_conversion.ipynb` を実行。`ut.coordinate` を基準に #850 のずれを確認（同じ画素で約 15.7 km、ずらすと約 0.1 km）。本体の 4 か所が同じ補正をしていることをコードで確認 → [`logs/2026-11-04-coord-conversion.md`](../logs/2026-11-04-coord-conversion.md)

## 次の一手（優先順）
1. **ユーザー**: notes #5 のコメント（PR 本文と #850 コメントの下書き）をメンターに見てもらう
2. go が出たら、下書きの「投稿のしかた」の順に Claude が実行: upstream が進んでいないか確認 → fork に push → `gh pr create` → PR 番号を入れて `gh issue comment 850`
3. PR を出したら、CI（pre-commit.ci・CircleCI・Codacy）の結果を見て、緑ならチェックリストにチェックを付ける。レビューのコメントには方針を相談してから対応する
4. notes #9（`split_range()`）は、メンテナの返事しだいで #850 の PR に足すか、別の PR にする
5. 余力があれば: 上昇軌道のデータで上下と東西に分ける（`asc_desc2horz_vert.py`）、ERA5 で補正して GNSS と比べる（CDS のキーはユーザーが設定）、`workflows/README.md` の課題。後回し: uv / Docker での環境構築

## 未解決・保留中の問題
- なし（課題は Issue に記録）

## 直近の決定事項
- 2026-11-24: `split_range()` の不具合は #850 の PR に入れず、本文で触れるだけにする → 詳細は docs/decisions.md
- 2026-11-24: #850 の PR は draft ではなく通常の PR にし、`Fix #850` を付ける。投稿はメンターの go のあと → 詳細は docs/decisions.md
- 2026-11-22: #850 は新しい issue を立てず #850 にコメントする → 詳細は docs/decisions.md
