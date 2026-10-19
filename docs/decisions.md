# 設計判断の記録

## 2026-10-11: 開発環境は専用の conda 環境 `mintpy` に、CI 相当のパッケージで作る
- 決定: `mamba create -n mintpy --file requirements.txt gdal libgdal-netcdf pre-commit pyfftw pytest setuptools_scm` → `pip install -e ~/MintPy`
- 理由: CI（`.circleci/config.yml`）と同じ範囲にしておけば、手元で通ったものが CI でも通る。環境名は isce3 と揃えてプロジェクト名にし、依存の衝突を避けるため isce3 環境とは分ける
- 却下した代替案:
  - installation.md 例示の環境名 `insar` → 何の環境か分かりにくい
  - `isce2` も追加 → ISCE-2 の出力を自分で処理する予定がなく、CI も入れていない。依存が重い
  - `requirements.txt` + pytest + pre-commit のみ → ARIA / HyP3 / GMTSAR の統合テストが動かない
  - path setup 方式（`PYTHONPATH` を通す）→ installation.md で非推奨寄り。entry point が登録されない

## 2026-10-19: #850 の前に MintPy の公式チュートリアルをやる
- 決定: #850 は続けるが、その前に公式チュートリアル（insarlab/MintPy-tutorial の `workflows/smallbaselineApp.ipynb`）を一通りやる。結果の目視には GIS ツール（Google Earth・QGIS など）への書き出しも使う。環境の uv / Docker 化は後回し
- 理由: メンターの助言（mtg）。これまで統合テストは通したが、`smallbaselineApp.py` の各ステップが何をしているかは追えていない。理解してから取り組むほうが #850 の説明にも次の題材探しにも効く。今の conda 環境は全テストが通っていて、作り直しても理解や #850 は進まない
- 却下した代替案: #850 の修正を先に進める / 先に uv・Docker で環境を作り直す
- 補足: Docker の公式イメージ（`ghcr.io/insarlab/mintpy`）は、#850 が配布版の環境でも起きるかを確かめる用途で使える
- 補足: 目視は **KML / KMZ（Google Earth）から**始める（「初心者には KML が楽」というメンターの助言）。チュートリアルの `google_earth` ステップでそのまま KMZ ができる。QGIS は慣れてから
- 補足: ノートブックは VS Code で開く。conda 環境 `mintpy` に `ipykernel` だけ追加した

## 2026-10-11: CLAUDE.md のコーディング規約・Git規約は notes 用とし、`~/MintPy` では upstream の流儀に従う
- 決定: 日本語コメント・`feature/` ブランチ・`feat:` 接頭辞はこのリポジトリだけに適用する。`~/MintPy` のコードは英語のコメント・docstring、ブランチ名とコミットメッセージは CONTRIBUTING.md と既存の履歴に合わせる
- 理由: upstream は英語のコードベースで、コミットも `load_data: ...` のような形式。notes の規約のまま PR を出すと差し戻される。日本語での流れの説明は会話か notes 側で行えば「自分で説明できるコードだけ採用する」方針は保てる

## 2026-10-11: コントリビュートの題材は「内容に踏み込んだもの・既存 issue への貢献」を優先する
- 決定: ライブラリの新しい版で非推奨になった書き方の対応（警告つぶし）は題材にしない。見つけた問題は notes の Issue に記録し、upstream に出すかは別途判断する
- 理由: 非推奨対応はオーナーも把握していると考えられ、貢献としての価値が低い。InSAR の処理内容に踏み込んだものや、既存 issue の再現・原因調査のほうが upstream の役に立つ
- 却下した代替案: 統合テストで見つけた警告 3 件をそのまま PR にする → 1 件（#1）は実際に落ちる条件を再現できたので記録は残すが、優先はしない

## 2026-10-11: 統合テストはユーザーが手動で実行する
- 決定: `tests/smallbaselineApp.py` は Claude Code では実行せず、コマンドだけ記録する
- 理由: テストデータのダウンロードを伴い約 15 分かかる（CLAUDE.md の「大量データのダウンロードはユーザーが実行」）
- 追記（同日）: 実測は DL 7 分 21 秒 + 計算 56 秒。所要時間は想定より短いが、ダウンロードを伴う点は変わらないので方針は維持
