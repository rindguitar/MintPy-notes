# 2026-11-22 upstream #850 へのコメントの下書き（未投稿）

**状態: 下書き。** メンターの確認で go が出たら、`gh issue comment` で投稿する（下の「投稿する本文」の部分だけを使う）。
メンションと upstream の URL は入れていない。#850 にコメントすれば、起票者と過去のコメント者には通知が届く。

数字とコードは、`~/MintPy` の `b9bc6214`（v1.6.4-7、2026-11-22 時点の upstream/main）・scipy 1.18.1 で実際に動かして確かめたもの。

## 投稿する本文

<!-- BEGIN COMMENT -->
I looked into this with the current `main` (b9bc6214) and the WellsEnvD2T399 test dataset.

**Current behavior**

The script no longer crashes, but for a subset dataset it writes `latitude`/`longitude` to shifted pixels in `geometryRadar.h5`. The shift equals `SUBSET_YMIN`/`SUBSET_XMIN` of the radar-coded file (11/89 pixels for this dataset, about 16 km on the ground).

**To reproduce** (in `WellsEnvD2T399/mintpy`, after running `tests/smallbaselineApp.py --dset WellsEnvD2T399`)

```bash
cp inputs/geometryRadar.h5 geometryRadar_test.h5
lookup_geo2radar.py inputs/geometryGeo.h5 -w geometryRadar_test.h5
```

```python
from mintpy.utils import readfile, utils as ut

atr = readfile.read_attribute('velocity.h5')
coord = ut.coordinate(atr, lookup_file='inputs/geometryGeo.h5')
coord.open()
print(coord.radar2geo(127, 128)[0:2])   # 41.1718, -114.8809

lat = readfile.read('geometryRadar_test.h5', datasetName='latitude')[0]
lon = readfile.read('geometryRadar_test.h5', datasetName='longitude')[0]
print(lat[127, 128], lon[127, 128])     # 41.1583, -114.6942 -> about 16 km off
print(lat[138, 217], lon[138, 217])     # 41.1725, -114.8817 -> matches, at (127 + 11, 128 + 89)
```

**Cause**

As pointed out above, `rangeCoord`/`azimuthCoord` in `geometryGeo.h5` are in the radar coordinates before subsetting. `objects/coord.py` and `objects/resample.py` subtract `SUBSET_YMIN`/`SUBSET_XMIN` of the radar-coded file before using them, but `lookup_geo2radar.py` does not. Because of the shift, 16 of the 100 tiles have no points. The IndexError above most likely came from `griddata` on such an empty tile; SciPy 1.18.1 returns NaN for it instead, so the script now finishes without any error.

**Proposed fix**

I'd like to submit a PR that:
1. subtracts `SUBSET_YMIN`/`SUBSET_XMIN` of the `--write` file from the non-zero `azimuthCoord`/`rangeCoord` in `run_lookup_geo2radar()`, in the same way as `coord.py` and `resample.py`;
2. replaces `tqdm` with `mintpy.utils.ptime.progressBar`, as suggested above.

Does this sound good to you?
<!-- END COMMENT -->

## 対訳（確認用）

| 英語 | 日本語訳 |
|---|---|
| I looked into this with the current `main` (b9bc6214) and the WellsEnvD2T399 test dataset. | 現在の `main`（b9bc6214）と、テストデータ WellsEnvD2T399 を使って、この件を調べました。 |
| **Current behavior** | **現在の挙動** |
| The script no longer crashes, but for a subset dataset it writes `latitude`/`longitude` to shifted pixels in `geometryRadar.h5`. | スクリプトはもう落ちませんが、切り出したデータでは、`geometryRadar.h5` の中のずれた画素に `latitude` / `longitude` を書き込みます。 |
| The shift equals `SUBSET_YMIN`/`SUBSET_XMIN` of the radar-coded file (11/89 pixels for this dataset, about 16 km on the ground). | ずれの量は、レーダー座標のファイルの `SUBSET_YMIN` / `SUBSET_XMIN` と同じです（このデータでは 11 / 89 画素で、地上ではおよそ 16 km）。 |
| **To reproduce** (in `WellsEnvD2T399/mintpy`, after running `tests/smallbaselineApp.py --dset WellsEnvD2T399`) | **再現方法**（`tests/smallbaselineApp.py --dset WellsEnvD2T399` を実行したあと、`WellsEnvD2T399/mintpy` の中で） |
| （コード 1）`geometryRadar.h5` をコピーし、そのコピーに `lookup_geo2radar.py` を実行する | 元のファイルを書き換えないため、コピーに対して実行する |
| （コード 2）`ut.coordinate` で画素 (127, 128) の緯度経度を求め、書き込まれた値と比べる | MintPy 自身の関数の答えを基準にする |
| `# 41.1718, -114.8809` | 基準（`ut.coordinate` の答え） |
| `# 41.1583, -114.6942 -> about 16 km off` | 同じ画素に書かれた値。約 16 km ずれている |
| `# 41.1725, -114.8817 -> matches, at (127 + 11, 128 + 89)` | 11 行・89 列先の画素に、基準と一致する値がある |
| **Cause** | **原因** |
| As pointed out above, `rangeCoord`/`azimuthCoord` in `geometryGeo.h5` are in the radar coordinates before subsetting. | 上のコメントで指摘されているとおり、`geometryGeo.h5` の `rangeCoord` / `azimuthCoord` は、切り出す前のレーダー座標の番号です。 |
| `objects/coord.py` and `objects/resample.py` subtract `SUBSET_YMIN`/`SUBSET_XMIN` of the radar-coded file before using them, but `lookup_geo2radar.py` does not. | `objects/coord.py` と `objects/resample.py` は、使う前にレーダー座標のファイルの `SUBSET_YMIN` / `SUBSET_XMIN` を引いていますが、`lookup_geo2radar.py` は引いていません。 |
| Because of the shift, 16 of the 100 tiles have no points. | このずれのせいで、100 個の小区画のうち 16 個には点が 1 つも入りません。 |
| The IndexError above most likely came from `griddata` on such an empty tile; SciPy 1.18.1 returns NaN for it instead, so the script now finishes without any error. | 上で報告されている IndexError は、おそらくそうした空の区画で `griddata` が落ちたものです。SciPy 1.18.1 では代わりに NaN を返すので、今はエラーなしで最後まで終わってしまいます。 |
| **Proposed fix** | **修正の提案** |
| I'd like to submit a PR that: | 次の内容の PR を出したいと考えています。 |
| 1. subtracts `SUBSET_YMIN`/`SUBSET_XMIN` of the `--write` file from the non-zero `azimuthCoord`/`rangeCoord` in `run_lookup_geo2radar()`, in the same way as `coord.py` and `resample.py`; | 1. `run_lookup_geo2radar()` の中で、0 でない `azimuthCoord` / `rangeCoord` から、`--write` で指定したファイルの `SUBSET_YMIN` / `SUBSET_XMIN` を引く。やり方は `coord.py` や `resample.py` と同じ。 |
| 2. replaces `tqdm` with `mintpy.utils.ptime.progressBar`, as suggested above. | 2. 上のコメントでの提案どおり、`tqdm` を `mintpy.utils.ptime.progressBar` に置き換える。 |
| Does this sound good to you? | この進め方でよいでしょうか？ |

### 言い回しについての補足

- 「As pointed out above」「as suggested above」の「above」は、#850 の 2022 年のコメント（原因の指摘をしたスクリプト作者、tqdm の置き換えを提案したメンテナ）を指す。名前を出さず、メンションもしない
- 「most likely」は、古い scipy で落ちることを実際には試していないので付けている
- 「about 16 km」は、地上での合計のずれ（南へ 1.7 km、東へ 15.7 km）を丸めたもの

## 投稿のしかた（go が出たら）

```bash
# 本文だけを取り出して投稿する（gh はユーザーのアカウントで投稿する）
F=$(mktemp)
awk '/<!-- BEGIN COMMENT -->/{f=1; next} /<!-- END COMMENT -->/{exit} f' logs/2026-11-22-issue-850-comment-draft.md > "$F"   # 最初の終わりの目印で止める
grep -c '@[A-Za-z]' "$F"   # 0 であること
gh issue comment 850 -R insarlab/MintPy -F "$F"
```
