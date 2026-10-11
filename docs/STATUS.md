# STATUS

最終更新: 2026-10-11（セッション終了時に必ず更新）

## 現在のフェーズ
開発環境の構築が完了。コントリビュートの題材を探す段階（**内容に踏み込んだもの・既存 issue への貢献を優先**）。

## 前回やったこと
- 2026-10-11: 見つけた upstream 未報告の問題を notes の Issue に記録（#1 `os.path.isfile(False)` で落ちる潜在バグ / #2 CONTRIBUTING.md の誤記 / #3 非推奨警告は見送りで close）。`feature/env-setup` を main にマージ
- 2026-10-11: CLAUDE.md と `.claude/` に残っていたテンプレート（make / Docker / GPU / `src/` 等の README 必須ルール / 別プロジェクトのテスト規約）を削除し、このリポジトリの実態に合わせた
- 2026-10-11: 開発環境を構築（conda 環境 `mintpy` + editable install）。pre-commit・単体テスト・統合テスト（FernandinaSenDT128）すべて通過 → [`logs/2026-10-11-env-setup.md`](../logs/2026-10-11-env-setup.md)

## 次の一手（優先順）
1. upstream の open issue を読み、内容に踏み込んで貢献できるものを探す（再現・原因調査・コメントで既存 issue に参加できるもの）
2. notes の Issue #1・#2 は、上の貢献のついでに upstream へ出すかを判断する

## 未解決・保留中の問題
- なし

## 直近の決定事項
- 2026-10-11: コントリビュートは内容に踏み込んだもの・既存 issue への貢献を優先し、非推奨対応（警告つぶし）は題材にしない → 詳細は docs/decisions.md
- 2026-10-11: 開発環境は専用の conda 環境 `mintpy`、パッケージ範囲は CI 相当（isce2 なし） → 詳細は docs/decisions.md
