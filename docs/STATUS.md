# STATUS

最終更新: 2026-10-13（セッション終了時に必ず更新）

## 現在のフェーズ
着手先を **upstream #850（`lookup_geo2radar.py` が subset された GAMMA データで落ちる）** に切り替えた段階。再現の準備中。

## 前回やったこと
- 2026-10-13: #1405（回転コヒーレンス行列）を下調べしたが、重複の可能性と内容の浅さから保留し #850 に切り替え。#1405 でやったことは notes #8 にまとめた。下調べで見つけた未報告の不具合は notes #6 / #7
- 2026-10-13: upstream の open issue 48 件を調査。外部 PR は中央値 7 日でマージ → [`logs/2026-10-13-upstream-issues.md`](../logs/2026-10-13-upstream-issues.md)
- 2026-10-11: 見つけた upstream 未報告の問題を notes の Issue に記録（#1 / #2、#3 は見送りで close）。`feature/env-setup` を main にマージ

## 次の一手（優先順）
1. **ユーザーが実行**: WellsEnvD2T399（GAMMA、280 MB）の統合テスト
   `mamba run -n mintpy python ~/MintPy/tests/smallbaselineApp.py --dir ~/data --dset WellsEnvD2T399 2>&1 | tee ~/data/smallbaselineApp-WellsEnvD2T399.log`
2. #850 を再現し、原因（subset された lookup table の座標オフセット）を確認して、「subset 対応」と「非対応メッセージ」のどちらにするか決める（詳細は notes #5）
3. notes の `feature/upstream-issue-survey` を main にマージ

## 未解決・保留中の問題
- なし

## 直近の決定事項
- 2026-10-11: コントリビュートは内容に踏み込んだもの・既存 issue への貢献を優先し、非推奨対応（警告つぶし）は題材にしない → 詳細は docs/decisions.md
- 2026-10-11: 開発環境は専用の conda 環境 `mintpy`、パッケージ範囲は CI 相当（isce2 なし） → 詳細は docs/decisions.md
