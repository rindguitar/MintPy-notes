# 作業ログ

日付ごとの作業記録。

**追記のみ。過去のログは修正しない。**
「そのとき何が起きたか」を残すのが目的なので、後から正しい知識に直してしまうと
記録としての価値が失われる。内容が古くなった事実は [`reference/`](../reference/README.md) 側を更新する。

ファイル名は `YYYY-MM-DD-<短い題名>.md`。

## 一覧（新しい順）

| 日付 | ログ | 内容 |
|---|---|---|
| 2026-10-13 | [upstream の open issue 調査](2026-10-13-upstream-issues.md) | open 48 件を分類し 11 件を精読。外部 PR は中央値 7 日でマージされる。候補 3 件（#215 / #850 / #1405）から **#1405（回転コヒーレンス行列）に着手**。#215 は手元で再現し、レーダー座標では `-i linear` が必ず落ちることを確認 |
| 2026-10-11 | [開発環境の構築](2026-10-11-env-setup.md) | CONTRIBUTING.md に沿って conda 環境 `mintpy` を作成し、fork を editable install。pre-commit 全項目 Passed・CI 相当の単体テスト全件成功・統合テスト（FernandinaSenDT128）PASS。fork にタグが無いとバージョンが壊れること、CONTRIBUTING.md の `git pull upstream/main` が動かないこと、統合テストで出る警告 3 件（`os.path.isfile(False)` の潜在バグなど。いずれも upstream 未報告）を発見 |
