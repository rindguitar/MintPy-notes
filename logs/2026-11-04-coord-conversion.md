# 2026-11-04 coord_conversion.ipynb と、#850 の「ずらす」が MintPy の設計どおりかの確認

## 目的

公式の `applications/coord_conversion.ipynb`（`ut.coordinate` によるレーダー座標 ⇔ 地理座標の変換）を手元のデータで動かし、
upstream #850 のずれを MintPy 自身の関数を基準に確かめる。あわせて、切り出しの開始位置の分「ずらす」ことが MintPy の設計と食い違っていないかをコードで確認する。

## 結論（先に）

- `ut.coordinate` を基準にすると、`lookup_geo2radar.py` の出力は同じ画素で約 15.7 km ずれ、(11, 89) 画素ずらすと約 0.1 km まで一致した。**自作スクリプトなしで #850 を示せる**
- **「ずらす」のは MintPy の決まりどおり。** 本体は 4 か所で同じ補正をしていて、`lookup_geo2radar.py` だけが補正していない
- 修正のときは、**`geometryGeo.h5` ではなく `-w` のファイル（レーダー側）の `SUBSET_*` を使う**、0（データなし）は引かない、の 2 点に注意

## ノートブックの結果

| データ | 対応表 | `radar2geo` → `geo2radar` | `debug_mode=True` の図 |
|---|---|---|---|
| FernandinaSenDT128（ISCE 型） | `inputs/geometryRadar.h5` | (225, 300) → (224, 299) | 出る（近い画素を探す処理を見せる） |
| WellsEnvD2T399（GAMMA 型） | `inputs/geometryGeo.h5` | (127, 128) → (127, 128) | 出ない（表の値を読むだけで探さないため。正常） |

ノートブックは作者の環境のパスなので、`proj_dir`・`geom_file`・`y1, x1` を書き換えて使う（ユーザーが実行）。

## `ut.coordinate` を基準にした #850 の確認（ユーザーが実行）

`~/data/check850/geometryRadar.h5` は `geometryRadar.h5` のコピーに `lookup_geo2radar.py` を実行したもの。

```
ut.coordinate の答え        : 41.1718, -114.8809
同じ画素 (127, 128)        : 41.1583, -114.6942 | 基準との差 北 -1.50 km, 東 +15.60 km
11 行・89 列ずらした画素: 41.1725, -114.8817 | 基準との差 北 +0.08 km, 東 -0.07 km
```

残りの約 0.1 km は、1 画素（約 90 m）程度の差。うち半画素分（北 +46 m、東 −35 m）は符号も一致していて、
`lookup_geo2radar.py` が地理格子の緯度経度を画素の角（`Y_FIRST + i * Y_STEP`）で計算し、`ut.coordinate` が画素の中心（`+ 0.5`）で計算する違いで説明できる。残りは最近傍の補間による。

## 「ずらす」は MintPy の設計どおりか（コードの確認、`9e0d46b2`）

### 本体の 4 か所が同じ補正をしている

| 場所 | 処理 | コメント・出力 |
|---|---|---|
| `objects/coord.py` `geo2radar()`（:326-331, :346-347） | 表の値から `SUBSET_YMIN` / `XMIN` を引く | `# if source data file is subsetted before` |
| `objects/coord.py` `radar2geo()`（:424-426） | 画素番号に `SUBSET_XMIN` / `YMIN` を足してから表を探す | 上の逆向き |
| `objects/resample.py`（:635-638、pyresample の経路） | 0 でない値から引く | `input data file was cropped before.` |
| `objects/resample.py`（:786-789、scipy の経路） | 同上 | 同上 |

どれも使っているのは**データ側（レーダー側）のファイル**（`src_metadata` / `src_meta`）の属性。

### 属性の定義

| 場所 | 内容 | 意味 |
|---|---|---|
| `utils/attribute.py` `update_attribute4subset()`（:235-238） | `sub_x[0] + 元の SUBSET_XMIN` | 切り出しを重ねても、常に元の画像からの位置になる |
| `utils/attribute.py` マルチルックの更新（:107-111） | `(SUBSET_XMIN - box[0]) // lks_x` | マルチルック後の画素の尺度に直される |
| `objects/stackDict.py` | `rangeCoord` / `azimuthCoord` を `xstep` / `ystep` で割る | 表の値も同じ尺度になる |

→ 決まりは「**表の値は切り出し前の画像の番号（マルチルック後の尺度）。使う側が、データ側の `SUBSET_*` を引く**」。

### `lookup_geo2radar.py` の位置づけ

- 出力の配列の大きさは `-w` のファイルの `LENGTH` / `WIDTH`（切り出し後の 254 × 257）で、同じファイルの `height` と画素が対応する前提。切り出し前の番号のまま格納することはできないので、ずらさないのは設計上の意図ではなく未対応
- スクリプト作者も #850 で「subset は未対応」と書き、メンテナは「対応を足すか、非対応のメッセージを」と回答している

## 修正のときの注意

1. **`-w` のファイル（`geometryRadar.h5`）の `SUBSET_*` を使う。** `geometryGeo.h5` にも `SUBSET_XMIN = 371` があるが、これは地理格子の番号なので使ってはいけない
2. **0（データなし）は引かない。** スクリプトは後で `xx0 != 0` で除外するので、先に全部引くと 0 が −89 になり、除外されなくなる。本体も `dest_y != 0.` の画素だけ引いている
3. 属性が無いデータでは何もしない（`if 'SUBSET_XMIN' in ...`）

## 範囲外（今回は直さない）

- 地理格子の緯度経度が画素の角で計算されている（半画素、約 46 m）
- 無効な画素を NaN で書くので、`geocode.py` が出力を対応表として読めない（`find_valid_lat_lon()` は 0 を無効とみなす）
- `multilook.py` でレーダー側だけ後から縮小した場合は、`SUBSET_*` と表の値の尺度がそろわない可能性がある（本体の `coord.py` / `resample.py` も同じ条件。未確認）
- 確認したのは GAMMA の WellsEnvD2T399 だけ。ROI_PAC は同じ経路のはずだが未確認

## 追記（2026-11-05）: 切り出しの開始位置 (x = 89, y = 11) の由来

`~/data/smallbaselineApp-WellsEnvD2T399.log` と `load_data.py` の `read_subset_box()` で確認した。

1. テスト設定の `mintpy.subset.lalo = 41.02:41.32, -115.10:-114.65` で、切り出す範囲を緯度経度で指定している
2. `coord.bbox_geo2radar()` が対応表を使って、緯度経度の範囲をレーダー画像の画素の範囲に変換する → `box to read for datasets in y/x: (179, 23, 693, 531)`（x0, y0, x1, y1。GAMMA が出力した 1558 × 1292 の画像の番号）
3. `mintpy.multilook.xstep / ystep = 2` で画素を 2 × 2 にまとめるので、番号も 2 で割って切り捨てる（`utils/attribute.py`）→ x: 179 // 2 = 89、y: 23 // 2 = 11
4. 属性に `SUBSET_XMIN = 89`、`SUBSET_YMIN = 11`（`XMAX = 346`、`YMAX = 265`）として記録される

地理側も同じ流れで、対応表の範囲 (742, 316, 2268, 1405) が 2 で割られて `SUBSET_XMIN = 371`、`SUBSET_YMIN = 158` になる。
