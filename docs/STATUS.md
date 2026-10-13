# STATUS

最終更新: 2026-10-13（セッション終了時に必ず更新）

## 現在のフェーズ
upstream の issue を調査し、着手先を **#1405（`plot_coherence_matrix.py` に回転表示を追加）** に決めた段階。

## 前回やったこと
- 2026-10-13: upstream の open issue 48 件を調査。候補 3 件から #1405 に着手決定。残りの候補は notes #4（upstream #215）・#5（upstream #850）に記録 → [`logs/2026-10-13-upstream-issues.md`](../logs/2026-10-13-upstream-issues.md)
- 2026-10-11: 見つけた upstream 未報告の問題を notes の Issue に記録（#1 / #2、#3 は見送りで close）。`feature/env-setup` を main にマージ
- 2026-10-11: CLAUDE.md と `.claude/` に残っていたテンプレートを削除し、このリポジトリの実態に合わせた

## 次の一手（優先順）
1. #1405: 現在の `plot_coherence_matrix.py`（#1493 / #1495 適用後）と `utils/plot.py` の `plot_rotate_diag_coherence_matrix` を読み、回転表示の追加方針を作る
2. #1405 に着手表明と方針をコメントする（huchangyang さんがこのスクリプトを改良中なので、重ならないか確認）→ 反応を見て PR
3. notes の `feature/upstream-issue-survey` を main にマージ

## 未解決・保留中の問題
- なし

## 直近の決定事項
- 2026-10-11: コントリビュートは内容に踏み込んだもの・既存 issue への貢献を優先し、非推奨対応（警告つぶし）は題材にしない → 詳細は docs/decisions.md
- 2026-10-11: 開発環境は専用の conda 環境 `mintpy`、パッケージ範囲は CI 相当（isce2 なし） → 詳細は docs/decisions.md
