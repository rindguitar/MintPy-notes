# 2026-11-24 upstream #850 の修正（手元）と、PR・コメントの下書き（未投稿）

**状態: 下書き。** メンターの go が出るまで、fork への push・PR の作成・#850 へのコメントはしない。
下書きは notes #5 にコメントしてメンターと共有する。
[2026-11-22 のコメントの下書き](2026-11-22-issue-850-comment-draft.md)は、このファイルの短い版に置き換えた（再現手順と数字は PR の本文へ移した）。

## 修正の内容（`~/MintPy`、ブランチ `lookup_geo2radar_subset`）

upstream/main `b9bc6214` から切ったブランチに、コミットを 2 つ作った（手元のみ。fork には空のブランチだけ push 済み）。

| コミット | 内容 |
|---|---|
| `20f41e74` lookup_geo2radar: support subset dataset | 0（データなし）を除いたあとの `xx` / `yy` から、`-w` のファイルの `SUBSET_XMIN` / `SUBSET_YMIN` を引く（コメント 3 行 + コード 2 行）。属性が無いファイルでは 0 を引くので変わらない |
| `d0edab06` lookup_geo2radar: replace tqdm with ptime.progressBar | 進捗の表示を `tqdm` から MintPy 自前の `ptime.progressBar` に置き換える（並列 1 のときと 2 以上のときの両方） |

`tqdm` は `requirements.txt` に無く、pyaps3 → cdsapi を通して間接的に入るだけだった。最初は「入っていない環境では import で落ちる」とコミットメッセージに書いたが、実際には必ず入るので、事実どおりの書き方に直した（push 前なので手元のコミットを書き換えた）。
コミットメッセージの末尾の `Co-Authored-By` 行は、ユーザーの判断で消した（PR 本文の「Generated with Claude Code」の行も同じく消した）。

## 検証の結果

WellsEnvD2T399（`tests/smallbaselineApp.py --dset WellsEnvD2T399` の出力）で、`inputs/geometryRadar.h5` のコピーに `lookup_geo2radar.py inputs/geometryGeo.h5 -w <コピー>` を実行し、書き込まれた緯度経度を `ut.coordinate(...).radar2geo()` の答えと比べた。

| | 修正前 | 修正後 |
|---|---|---|
| ランダムな 300 画素での距離（中央値 / 95%） | 15.4 / 16.9 km | 0.07 / 0.17 km |
| 画素 (127, 128) の距離 | 15.67 km | 0.10 km |
| `latitude` の NaN（65,278 画素のうち） | 11,428 | 1,028 |
| 標高による独立チェック（中央値） | 115.2 m | 2.5 m |

- 画素の大きさは方位方向で 162 m なので、修正後の残りの差はおよそ 1 画素以内
- 切り出しの属性（`SUBSET_*`）を消したコピーでは、修正前と修正後の出力が完全に同じ
- `--parallel 1` と `--parallel 2` の出力が同じ。2 つ目のコミット（tqdm の置き換え）の前後でも出力が同じ。進捗バーが出て、エラーなし
- pre-commit（`--all-files`）: 13 Passed、1 Skipped。単体テスト: CI と同じ 4 本すべて成功、`pytest tests/` 4 passed
- ただし、どのテストも `lookup_geo2radar.py` を通らない。この修正の確認は上の手作業の比較が担っている

### 気づいた別の不具合（notes #9）

修正後に残る NaN 1,028 画素は、**最後の 4 行（250〜253 行）× 257 列**だった。`split_range()` が区画の幅を `round(254/10) = 25` にするので、10 区画で 250 行しか覆えない。修正の前でも後でも、切り出しの有無にも関係なく起きる。#850 とは原因が別なので、この PR には入れず、本文で触れるだけにした。詳細は notes #9。

## PR の下書き

タイトル: `lookup_geo2radar: fix shifted lat/lon for subset dataset`

（最近の upstream の PR の形 `prep_nisar: fix the 90 deg offset in azimuthAngle` に合わせた）

### 投稿する本文

<!-- BEGIN PR BODY -->
**Description of proposed changes**

For a subset dataset (e.g. with `mintpy.subset.lalo` in `smallbaselineApp.py`), `lookup_geo2radar.py` writes `latitude`/`longitude` to shifted pixels of the `--write` file. The shift equals `SUBSET_YMIN`/`SUBSET_XMIN` of that file (11/89 pixels for the WellsEnvD2T399 test dataset, about 16 km on the ground). With recent SciPy (1.18.1 here), the script no longer crashes as reported in #850, so the shift goes unnoticed.

*Why.* `rangeCoord`/`azimuthCoord` in the geo-coded lookup table refer to the radar image before subsetting. `objects/coord.py` and `objects/resample.py` subtract `SUBSET_YMIN`/`SUBSET_XMIN` of the radar-coded file before using them, but `lookup_geo2radar.py` does not.

*Changes.*

1. Subtract `SUBSET_YMIN`/`SUBSET_XMIN` of the `--write` file from the non-zero `azimuthCoord`/`rangeCoord`, in the same way as `coord.py` and `resample.py`. Files without these attributes are not affected (the offsets default to 0).
2. Replace `tqdm` with `mintpy.utils.ptime.progressBar`, as suggested in #850. `tqdm` is not listed in `requirements.txt`.

*Verification.* On WellsEnvD2T399 (after `tests/smallbaselineApp.py --dset WellsEnvD2T399`), I ran `lookup_geo2radar.py inputs/geometryGeo.h5 -w` on a copy of `inputs/geometryRadar.h5` and compared the written `latitude`/`longitude` with `ut.coordinate(...).radar2geo()` at 300 random pixels.

| | before | after |
|---|---|---|
| distance to `radar2geo()`: median / 95th percentile | 15.4 / 16.9 km | 0.07 / 0.17 km |
| NaN pixels in `latitude` (out of 65,278) | 11,428 | 1,028 |

- The azimuth pixel size of this dataset is 162 m, so the remaining distance is within about one pixel.
- With the `SUBSET_*` attributes removed from the copy, the output is identical before and after this PR.
- `--parallel 1` and `--parallel 2` give identical output, and the second commit does not change the output.
- `pre-commit run --all-files` and the unit tests (`tests/` and the CircleCI unit-test scripts) pass. None of them run this script, so I checked it by hand as above.

The remaining 1,028 NaN pixels are the last 4 rows, which are NaN with or without this PR: `split_range()` uses `round(254/10) = 25` rows per tile, so the 10 tiles cover only 250 of the 254 rows. I left it out to keep this PR focused, and can fix it here or in a separate PR.

<details>
<summary>To reproduce (in <code>WellsEnvD2T399/mintpy</code>)</summary>

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
print(lat[127, 128], lon[127, 128])
# before: 41.1583, -114.6942 (about 16 km off; the matching value is at [127 + 11, 128 + 89])
# after:  41.1725, -114.8817
```

</details>

Feedback on the approach is welcome.

**Reminders**

- [x] Fix #850
- [x] Pass Pre-commit check (green)
- [ ] Pass Codacy code review (green)
- [ ] Pass Circle CI test (green)
- [x] Make sure that your code follows our style. Use the other functions/files as a basis.
- [ ] If modifying functionality, describe changes to function behavior and arguments in a comment below the function declaration.
- [ ] If adding new functionality, add a detailed description to the documentation and/or an example.

<!-- END PR BODY -->

### 対訳（確認用）

| 英語 | 日本語訳 |
|---|---|
| **Description of proposed changes** | **変更の説明**（テンプレートの見出し） |
| For a subset dataset (e.g. with `mintpy.subset.lalo` in `smallbaselineApp.py`), `lookup_geo2radar.py` writes `latitude`/`longitude` to shifted pixels of the `--write` file. | 切り出したデータ（たとえば `smallbaselineApp.py` で `mintpy.subset.lalo` を使った場合）では、`lookup_geo2radar.py` は `--write` で指定したファイルの、ずれた画素に `latitude` / `longitude` を書き込みます。 |
| The shift equals `SUBSET_YMIN`/`SUBSET_XMIN` of that file (11/89 pixels for the WellsEnvD2T399 test dataset, about 16 km on the ground). | ずれの量はそのファイルの `SUBSET_YMIN` / `SUBSET_XMIN` と同じです（テストデータ WellsEnvD2T399 では 11 / 89 画素で、地上ではおよそ 16 km）。 |
| With recent SciPy (1.18.1 here), the script no longer crashes as reported in #850, so the shift goes unnoticed. | 最近の SciPy（ここでは 1.18.1）では、#850 で報告されたようには落ちなくなったので、ずれに気づかれません。 |
| *Why.* `rangeCoord`/`azimuthCoord` in the geo-coded lookup table refer to the radar image before subsetting. | *原因。* 地理座標の対応表の `rangeCoord` / `azimuthCoord` は、切り出す前のレーダー画像を基準にした番号です。 |
| `objects/coord.py` and `objects/resample.py` subtract `SUBSET_YMIN`/`SUBSET_XMIN` of the radar-coded file before using them, but `lookup_geo2radar.py` does not. | `objects/coord.py` と `objects/resample.py` は、使う前にレーダー座標のファイルの `SUBSET_YMIN` / `SUBSET_XMIN` を引いていますが、`lookup_geo2radar.py` は引いていません。 |
| *Changes.* | *変更点。* |
| 1. Subtract `SUBSET_YMIN`/`SUBSET_XMIN` of the `--write` file from the non-zero `azimuthCoord`/`rangeCoord`, in the same way as `coord.py` and `resample.py`. | 1. 0 でない `azimuthCoord` / `rangeCoord` から、`--write` のファイルの `SUBSET_YMIN` / `SUBSET_XMIN` を引きます。やり方は `coord.py` や `resample.py` と同じです。 |
| Files without these attributes are not affected (the offsets default to 0). | この属性が無いファイルには影響しません（引く量が 0 になるため）。 |
| 2. Replace `tqdm` with `mintpy.utils.ptime.progressBar`, as suggested in #850. `tqdm` is not listed in `requirements.txt`. | 2. #850 での提案どおり、`tqdm` を `mintpy.utils.ptime.progressBar` に置き換えます。`tqdm` は `requirements.txt` に載っていません。 |
| *Verification.* On WellsEnvD2T399 (after `tests/smallbaselineApp.py --dset WellsEnvD2T399`), I ran `lookup_geo2radar.py inputs/geometryGeo.h5 -w` on a copy of `inputs/geometryRadar.h5` and compared the written `latitude`/`longitude` with `ut.coordinate(...).radar2geo()` at 300 random pixels. | *確認。* WellsEnvD2T399 で（`tests/smallbaselineApp.py --dset WellsEnvD2T399` を実行したあと）、`inputs/geometryRadar.h5` のコピーに `lookup_geo2radar.py inputs/geometryGeo.h5 -w` を実行し、書き込まれた `latitude` / `longitude` を、ランダムな 300 画素で `ut.coordinate(...).radar2geo()` の答えと比べました。 |
| （表）distance to `radar2geo()`: median / 95th percentile | `radar2geo()` との距離（中央値 / 95 パーセンタイル） |
| （表）NaN pixels in `latitude` (out of 65,278) | `latitude` の NaN の画素数（65,278 画素のうち） |
| The azimuth pixel size of this dataset is 162 m, so the remaining distance is within about one pixel. | このデータの方位方向の画素の大きさは 162 m なので、残りの距離はおよそ 1 画素以内です。 |
| With the `SUBSET_*` attributes removed from the copy, the output is identical before and after this PR. | コピーから `SUBSET_*` の属性を消した場合、この PR の前と後で出力は完全に同じです。 |
| `--parallel 1` and `--parallel 2` give identical output, and the second commit does not change the output. | `--parallel 1` と `--parallel 2` の出力は同じで、2 つ目のコミットは出力を変えません。 |
| `pre-commit run --all-files` and the unit tests (`tests/` and the CircleCI unit-test scripts) pass. | `pre-commit run --all-files` と単体テスト（`tests/` と CircleCI の単体テストのスクリプト）は通ります。 |
| None of them run this script, so I checked it by hand as above. | ただしどれもこのスクリプトを実行しないので、上のとおり手作業で確かめました。 |
| The remaining 1,028 NaN pixels are the last 4 rows, which are NaN with or without this PR: | 残っている NaN の 1,028 画素は最後の 4 行で、この PR があってもなくても NaN です。 |
| `split_range()` uses `round(254/10) = 25` rows per tile, so the 10 tiles cover only 250 of the 254 rows. | `split_range()` が 1 区画を `round(254/10) = 25` 行にするので、10 区画では 254 行のうち 250 行しか覆えません。 |
| I left it out to keep this PR focused, and can fix it here or in a separate PR. | この PR の焦点をしぼるために入れていませんが、この PR でも別の PR でも直せます。 |
| To reproduce (in `WellsEnvD2T399/mintpy`) | 再現方法（`WellsEnvD2T399/mintpy` の中で）。クリックで開く折りたたみ |
| （コード）`# before: ... (about 16 km off; the matching value is at [127 + 11, 128 + 89])` | 修正前: 約 16 km ずれている。基準と一致する値は 11 行・89 列先の画素にある |
| （コード）`# after:  41.1725, -114.8817` | 修正後: 基準とほぼ一致 |
| Feedback on the approach is welcome. | 方針について意見があればお願いします。 |
| **Reminders** | **確認事項**（テンプレートのチェックリスト） |
| - [x] Fix #850 | #850 を直す（マージされると #850 が自動で閉じる） |
| - [x] Pass Pre-commit check (green) | pre-commit が通る（手元で確認済み） |
| - [ ] Pass Codacy code review (green) / Pass Circle CI test (green) | Codacy・CircleCI が通る（PR を出したあとに結果が出るので、緑になったらチェックを付ける） |
| - [x] Make sure that your code follows our style. | 既存のコードの書き方に合わせた |
| - [ ] If modifying functionality, ... / If adding new functionality, ... | 機能や引数を変えた場合・機能を足した場合の項目。今回は当てはまらないので空のまま |

## #850 へのコメントの下書き（短い版）

PR を出したあとに、`#XXXX` を PR の番号に置き換えて投稿する。再現手順と数字は PR 側に書いたので、ここは「何が起きているか・原因・PR を出したこと」だけにした。

### 投稿する本文

<!-- BEGIN COMMENT -->
I looked into this with the current `main` (b9bc6214) and the WellsEnvD2T399 test dataset, and opened #XXXX to fix it.

The script no longer crashes, but for a subset dataset it writes `latitude`/`longitude` to shifted pixels in `geometryRadar.h5`. The shift equals `SUBSET_YMIN`/`SUBSET_XMIN` of the radar-coded file (11/89 pixels for this dataset, about 16 km on the ground).

As pointed out above, `rangeCoord`/`azimuthCoord` in `geometryGeo.h5` are in the radar coordinates before subsetting, and `lookup_geo2radar.py` does not subtract `SUBSET_YMIN`/`SUBSET_XMIN` of the radar-coded file as `objects/coord.py` and `objects/resample.py` do. Because of the shift, 16 of the 100 tiles have no points. The IndexError above most likely came from `griddata` on such an empty tile; SciPy 1.18.1 returns NaN for it instead, so the script now finishes without any error.

The PR also replaces `tqdm` with `ptime.progressBar`, as suggested above. The steps to reproduce and the numbers before and after the fix are in the PR.
<!-- END COMMENT -->

### 対訳（確認用）

| 英語 | 日本語訳 |
|---|---|
| I looked into this with the current `main` (b9bc6214) and the WellsEnvD2T399 test dataset, and opened #XXXX to fix it. | 現在の `main`（b9bc6214）とテストデータ WellsEnvD2T399 でこの件を調べ、直すための PR #XXXX を出しました。 |
| The script no longer crashes, but for a subset dataset it writes `latitude`/`longitude` to shifted pixels in `geometryRadar.h5`. | スクリプトはもう落ちませんが、切り出したデータでは、`geometryRadar.h5` の中のずれた画素に `latitude` / `longitude` を書き込みます。 |
| The shift equals `SUBSET_YMIN`/`SUBSET_XMIN` of the radar-coded file (11/89 pixels for this dataset, about 16 km on the ground). | ずれの量はレーダー座標のファイルの `SUBSET_YMIN` / `SUBSET_XMIN` と同じです（このデータでは 11 / 89 画素で、地上ではおよそ 16 km）。 |
| As pointed out above, `rangeCoord`/`azimuthCoord` in `geometryGeo.h5` are in the radar coordinates before subsetting, | 上のコメントで指摘されているとおり、`geometryGeo.h5` の `rangeCoord` / `azimuthCoord` は切り出す前のレーダー座標の番号で、 |
| and `lookup_geo2radar.py` does not subtract `SUBSET_YMIN`/`SUBSET_XMIN` of the radar-coded file as `objects/coord.py` and `objects/resample.py` do. | `lookup_geo2radar.py` は、`objects/coord.py` や `objects/resample.py` のようにはレーダー座標のファイルの `SUBSET_YMIN` / `SUBSET_XMIN` を引いていません。 |
| Because of the shift, 16 of the 100 tiles have no points. | このずれのせいで、100 個の区画のうち 16 個には点が 1 つも入りません。 |
| The IndexError above most likely came from `griddata` on such an empty tile; SciPy 1.18.1 returns NaN for it instead, so the script now finishes without any error. | 上で報告されている IndexError は、おそらくそうした空の区画で `griddata` が落ちたものです。SciPy 1.18.1 では代わりに NaN を返すので、今はエラーなしで最後まで終わってしまいます。 |
| The PR also replaces `tqdm` with `ptime.progressBar`, as suggested above. | 上での提案どおり、PR では `tqdm` も `ptime.progressBar` に置き換えています。 |
| The steps to reproduce and the numbers before and after the fix are in the PR. | 再現手順と、修正前後の数字は PR に書いています。 |

## 言い回しについての補足

- 「above」は #850 の 2022 年のコメント（原因を指摘したスクリプト作者、tqdm の置き換えを提案したメンテナ）を指す。名前は出さず、メンションもしない
- 「most likely」は、古い scipy で落ちることを実際には試していないので付けている
- 「about 16 km」は、地上での合計のずれ（南へ 1.7 km、東へ 15.7 km）を丸めたもの
- PR の「within about one pixel」は、95% の値 0.17 km が方位方向の画素 162 m とほぼ同じことから
- PR の本文の書き方（*Why.* / *Verification.* と、修正前後の表、再現手順は `<details>` に畳む）は、最近 upstream にマージされた外部からの PR（#1517、#1519）に合わせた
- `split_range()` の件は「この PR でも別の PR でも直せる」とだけ書き、どちらにするかはメンテナに任せる（notes #9）

## 投稿のしかた（メンターの go が出たら）

```mermaid
flowchart LR
  A["メンターの go"] --> B["fork に push<br/>git push"]
  B --> C["PR を作る<br/>gh pr create"]
  C --> D["PR 番号を入れて<br/>#850 にコメント<br/>gh issue comment"]
```

```bash
# 0. upstream が進んでいないか確かめる（進んでいたら、ブランチを最新の main に載せ直してから検証し直す）
git -C ~/MintPy fetch upstream && git -C ~/MintPy log --oneline -1 upstream/main   # b9bc6214 のままか

# 1. fork にコミットを送る
git -C ~/MintPy push origin lookup_geo2radar_subset

# 2. PR を作る（本文だけを取り出す。gh はユーザーのアカウントで投稿する）
cd ~/MintPy-notes
F=$(mktemp)
awk '/<!-- BEGIN PR BODY -->/{f=1; next} /<!-- END PR BODY -->/{exit} f' logs/2026-11-24-issue-850-pr-draft.md > "$F"   # 最初の終わりの目印で止める
grep -c '@[A-Za-z]' "$F"   # 0 であること
gh pr create -R insarlab/MintPy --base main --head rindguitar:lookup_geo2radar_subset \
  --title "lookup_geo2radar: fix shifted lat/lon for subset dataset" -F "$F"

# 3. 出てきた PR の番号を入れて、#850 にコメントする
N=<PR の番号>
G=$(mktemp)
awk '/<!-- BEGIN COMMENT -->/{f=1; next} /<!-- END COMMENT -->/{exit} f' logs/2026-11-24-issue-850-pr-draft.md | sed "s/#XXXX/#$N/" > "$G"
grep -c '#XXXX' "$G"; grep -c '@[A-Za-z]' "$G"   # どちらも 0 であること
gh issue comment 850 -R insarlab/MintPy -F "$G"
```
