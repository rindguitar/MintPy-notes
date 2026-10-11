# STATUS

最終更新: 2026-10-11（セッション終了時に必ず更新）

## 現在のフェーズ
開発環境の構築が完了。最初のコントリビュートの題材を決める段階。

## 前回やったこと
- 2026-10-11: 統合テスト（FernandinaSenDT128）PASS。DL 7 分 21 秒 + 計算 56 秒。全出力を保存して再実行し、警告 3 件の原因を特定（いずれも upstream 未報告） → ログに追記
- 2026-10-11: CLAUDE.md と `.claude/` に残っていたテンプレート（make / Docker / GPU / `src/` 等の README 必須ルール / 別プロジェクトのテスト規約）を削除し、このリポジトリの実態に合わせた
- 2026-10-11: CONTRIBUTING.md に沿って conda 環境 `mintpy` を作り、fork を editable install。pre-commit 全項目 Passed、CI と同じ単体テスト全件成功 → [`logs/2026-10-11-env-setup.md`](../logs/2026-10-11-env-setup.md)

## 次の一手（優先順）
1. notes の `feature/env-setup` を main にマージする（PR）
2. 最初のコントリビュート先を決める（いずれも upstream 未報告。詳細は [`logs/2026-10-11-env-setup.md`](../logs/2026-10-11-env-setup.md) の末尾）
   - `temporal_average(outFile=False)` が `os.path.isfile(False)` を呼ぶ潜在バグ（`utils1.py:384`）← 実害の可能性あり・修正は 1 行
   - CONTRIBUTING.md の `git pull upstream/main` → `git pull upstream main`
   - h5py: dtype 無しの `create_dataset`（2 か所）
   - scikit-image: `remove_small_objects(min_size=)` の非推奨（意味の変化と旧版との両立に注意）
3. upstream の open issue / PR を分類して、次の題材を探す

## 未解決・保留中の問題
- なし

## 直近の決定事項
- 2026-10-11: 開発環境は専用の conda 環境 `mintpy`、パッケージ範囲は CI 相当（isce2 なし） → 詳細は docs/decisions.md
