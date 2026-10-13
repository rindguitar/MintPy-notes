# STATUS

最終更新: 2026-10-13（セッション終了時に必ず更新）

## 現在のフェーズ
upstream の issue を調査し、着手先を **#1405（`plot_coherence_matrix.py` に回転表示を追加）** に決めた段階。

## 前回やったこと
- 2026-10-13: upstream の open issue 48 件を調査し、#1405（回転コヒーレンス行列）に決定（他の候補は notes #4 / #5）。下調べで同じスクリプトの未報告の不具合 3 件を発見（notes #6 / #7）。メンターへの相談文書を作成 → [`logs/2026-10-13-upstream-issues.md`](../logs/2026-10-13-upstream-issues.md)
- 2026-10-11: 見つけた upstream 未報告の問題を notes の Issue に記録（#1 / #2、#3 は見送りで close）。`feature/env-setup` を main にマージ
- 2026-10-11: CLAUDE.md と `.claude/` に残っていたテンプレートを削除し、このリポジトリの実態に合わせた

## 次の一手（優先順）
1. メンターに [`drafts/2026-10-13-mentor-report.md`](../drafts/2026-10-13-mentor-report.md) を送り、進め方の意見をもらう（送ったら `drafts/README.md` の状態を更新）
2. メンターの意見を反映して、upstream #1405 にコメント（担当してよいか・方針・見つけた不具合 3 件 = notes #6 / #7）
3. 返答を待って実装（`~/MintPy` にブランチを切るのはこの段階）
4. notes の `feature/upstream-issue-survey` を main にマージ

## 未解決・保留中の問題
- なし

## 直近の決定事項
- 2026-10-13: upstream に声をかける前にメンターに相談する → 詳細は docs/decisions.md
- 2026-10-11: コントリビュートは内容に踏み込んだもの・既存 issue への貢献を優先し、非推奨対応（警告つぶし）は題材にしない → 詳細は docs/decisions.md
- 2026-10-11: 開発環境は専用の conda 環境 `mintpy`、パッケージ範囲は CI 相当（isce2 なし） → 詳細は docs/decisions.md
