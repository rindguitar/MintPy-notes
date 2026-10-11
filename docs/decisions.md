# 設計判断の記録

## 2026-10-11: 開発環境は専用の conda 環境 `mintpy` に、CI 相当のパッケージで作る
- 決定: `mamba create -n mintpy --file requirements.txt gdal libgdal-netcdf pre-commit pyfftw pytest setuptools_scm` → `pip install -e ~/MintPy`
- 理由: CI（`.circleci/config.yml`）と同じ範囲にしておけば、手元で通ったものが CI でも通る。環境名は isce3 と揃えてプロジェクト名にし、依存の衝突を避けるため isce3 環境とは分ける
- 却下した代替案:
  - installation.md 例示の環境名 `insar` → 何の環境か分かりにくい
  - `isce2` も追加 → ISCE-2 の出力を自分で処理する予定がなく、CI も入れていない。依存が重い
  - `requirements.txt` + pytest + pre-commit のみ → ARIA / HyP3 / GMTSAR の統合テストが動かない
  - path setup 方式（`PYTHONPATH` を通す）→ installation.md で非推奨寄り。entry point が登録されない

## 2026-10-11: 統合テストはユーザーが手動で実行する
- 決定: `tests/smallbaselineApp.py` は Claude Code では実行せず、コマンドだけ記録する
- 理由: テストデータのダウンロードを伴い約 15 分かかる（CLAUDE.md の「大量データのダウンロードはユーザーが実行」）
