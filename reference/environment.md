# 環境スナップショット

最終更新: 2026-10-11

| | |
|---|---|
| ソースの HEAD（`~/MintPy`） | `9e0d46b2`（`upstream/main` と一致） |
| インストール済みの版 | `1.6.4.post3`（editable install） |

状況が変わったらこのファイルを上書き更新する。構築の経緯は
[`logs/2026-10-11-env-setup.md`](../logs/2026-10-11-env-setup.md)。

## マシン構成

| 項目 | 値 |
|---|---|
| OS | WSL2 (Ubuntu 24.04.2 LTS, kernel 5.15 microsoft-standard-WSL2) |
| CPU / メモリ | 8 コア / 15 GB |
| git | 2.43.0 |
| conda | Miniforge / conda 26.7.2 / mamba 2.5.0 / `~/miniforge3` |
| チャネル | `conda-forge` のみ |

## パスの配置

```
~/miniforge3/              conda 本体（全プロジェクト共通）
  └── envs/mintpy/         MintPy の開発環境
~/MintPy/                  fork をクローンしたもの（ソース。editable install の参照先）
  └── src/mintpy/          実際に import されるのはここ
~/MintPy-notes/            このメモ
~/MintPy-tutorial/         公式チュートリアル（insarlab/MintPy-tutorial を clone。ノートブックの実行結果はここに残る）
~/data/                    統合テストのデータ置き場（どちらのリポジトリにも入れない）
  ├── test/                       チュートリアルの作業場所（ARIA 版: SanFranSenDT42、4.9 GB）
  ├── FernandinaSenDT128.tar.xz   277 MB。残しておくと再実行時にダウンロードを飛ばせる
  ├── FernandinaSenDT128/         展開先 1.2 GB。再実行のたびに消して展開し直される
  ├── smallbaselineApp.log        統合テスト（FernandinaSenDT128）の全出力
  ├── WellsEnvD2T399.tar.xz       280 MB。GAMMA のテストデータ（upstream #850 の再現用）
  ├── WellsEnvD2T399/             展開先
  └── smallbaselineApp-WellsEnvD2T399.log  統合テスト（WellsEnvD2T399）の全出力
```

## conda 環境 `mintpy`

作成コマンド:

```bash
mamba create --yes --name mintpy -c conda-forge \
    --file ~/MintPy/requirements.txt \
    gdal libgdal-netcdf pre-commit pyfftw pytest setuptools_scm
mamba run -n mintpy python -m pip install -e ~/MintPy
mamba install -n mintpy -c conda-forge ipykernel     # 2026-10-19 追加: VS Code でチュートリアルのノートブックを動かすため
mamba install -n mintpy -c conda-forge ipympl        # 2026-10-25 追加: ノートブックの %matplotlib widget（対話的な図）のため
```

### 実際に入った版（主要なもの）

| パッケージ | 版 | 入手元 | 備考 |
|---|---|---|---|
| python | 3.14.8 | conda-forge | 上限指定なし。非常に新しい |
| numpy | 2.5.4 | conda-forge | |
| scipy | 1.18.1 | conda-forge | |
| h5py / hdf5 | 3.16.0 / 1.14.6 | conda-forge | |
| gdal | 3.13.3 | conda-forge | `libgdal-netcdf` も同版 |
| proj / pyproj | 9.9.0 / 3.8.0 | conda-forge | |
| matplotlib | 3.11.2 | conda-forge | |
| cartopy | 0.26.0 | conda-forge | |
| pyresample / pykdtree | 1.35.0 / 1.4.3 | conda-forge | |
| pyaps3 | 0.3.7 | conda-forge | 対流圏補正（ERA5） |
| pysolid | 0.3.4 | conda-forge | 固体地球潮汐 |
| dask / dask-jobqueue | 2026.8.0 / 0.9.0 | conda-forge | |
| cvxopt | 1.3.3 | conda-forge | |
| scikit-image | 0.26.0 | conda-forge | |
| shapely | 2.2.0 | conda-forge | |
| asf_search | 14.0.3 | conda-forge | |
| fsspec | 2026.9.0 | conda-forge | `[http]` extra は conda では無視される |
| **aiohttp** ほか 6 個 | 3.14.5 | **PyPI** | `fsspec[http]` の不足分を `pip install -e` が補った |
| libopenblas | 0.3.34 | conda-forge | BLAS 実装 |
| pyfftw | 0.15.1 | conda-forge | 開発用（`tests/requirements.txt`） |
| pytest | 9.1.1 | conda-forge | 開発用 |
| pre-commit | 4.6.2 | conda-forge | 開発用。システムの `/usr/bin/pre-commit`（3.6.2）とは別物 |
| ipykernel | 7.4.0 | conda-forge | 2026-10-19 追加。VS Code のノートブック用（依存込みで 24 パッケージ、既存の更新なし） |
| ipympl | 0.10.0 | conda-forge | 2026-10-25 追加。`%matplotlib widget` 用（依存込みで 4 パッケージ、既存の更新なし） |
| setuptools_scm | 10.3.4 | conda-forge | 開発用 |
| mintpy | 1.6.4.post3 | editable | `~/MintPy/src` を参照 |

## 開発で使うコマンド

```bash
# 環境に入る（または各コマンドの前に `mamba run -n mintpy` を付ける）
mamba activate mintpy

# upstream に追従する（CONTRIBUTING.md の `git pull upstream/main` は誤り）
cd ~/MintPy
git checkout main
git pull upstream main
git fetch upstream --tags      # タグが無いとバージョンが壊れる

# 作業ブランチ
git checkout -b <分かりやすい名前>     # 名前はマージメッセージに残る

# コミット前のチェック（git commit 時にも自動で走る）
pre-commit run --all-files

# 単体テスト（CI の "Unit Test" と同じ。数秒で終わる）
python ~/MintPy/tests/objects/euler_pole.py
python ~/MintPy/tests/objects/ionex.py
python ~/MintPy/tests/asc_desc2horz_vert.py
python ~/MintPy/tests/dem_error.py
pytest ~/MintPy/tests/

# 統合テスト（1 データセット: DL 約 280 MB + 計算 約 1 分。--dset を省略すると全 7 データセット）
mkdir -p ~/data
python ~/MintPy/tests/smallbaselineApp.py --dir ~/data --dset FernandinaSenDT128 2>&1 | tee ~/data/smallbaselineApp.log
```

## 確認済みの状態

| 項目 | 結果 | 確認日 |
|---|---|---|
| `pre-commit run --all-files` | 13 Passed / 1 Skipped（json 対象なし） | 2026-10-11 |
| 単体テスト 4 スクリプト | すべて OK | 2026-10-11 |
| `pytest tests/` | 4 passed | 2026-10-11 |
| 統合テスト `smallbaselineApp.py --dset FernandinaSenDT128` | PASS（DL 7 分 21 秒 + 計算 56 秒。PASS は「エラーなく完走」の意味で数値照合はしない）。全出力は `~/data/smallbaselineApp.log`。警告 3 件は既知（ログ参照） | 2026-10-11 |
| 統合テスト `smallbaselineApp.py --dset WellsEnvD2T399`（GAMMA） | PASS（DL 15 分 2 秒 + 計算 25.5 秒）。全出力は `~/data/smallbaselineApp-WellsEnvD2T399.log` | 2026-10-13 |
