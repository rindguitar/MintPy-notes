# STATUS

最終更新: 2026-10-11（セッション終了時に必ず更新）

## 現在のフェーズ
開発環境の構築が完了。最初のコントリビュートの題材を決める段階。

## 前回やったこと
- 2026-10-11: 統合テスト（FernandinaSenDT128）PASS。DL 7 分 21 秒 + 計算 56 秒。PASS は数値照合ではなく「エラーなく完走」の意味 → ログに追記
- 2026-10-11: CLAUDE.md と `.claude/` に残っていたテンプレート（make / Docker / GPU / `src/` 等の README 必須ルール / 別プロジェクトのテスト規約）を削除し、このリポジトリの実態に合わせた
- 2026-10-11: CONTRIBUTING.md に沿って conda 環境 `mintpy` を作り、fork を editable install。pre-commit 全項目 Passed、CI と同じ単体テスト全件成功 → [`logs/2026-10-11-env-setup.md`](../logs/2026-10-11-env-setup.md)

## 次の一手（優先順）
1. notes の `feature/env-setup` を main にマージする（PR）
2. 最初のコントリビュート候補: CONTRIBUTING.md の `git pull upstream/main` → `git pull upstream main` の修正（upstream 未報告を確認済み）
3. upstream の open issue / PR を分類して、次の題材を探す

## 未解決・保留中の問題
- なし

## 直近の決定事項
- 2026-10-11: 開発環境は専用の conda 環境 `mintpy`、パッケージ範囲は CI 相当（isce2 なし） → 詳細は docs/decisions.md
