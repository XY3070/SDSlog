Date: 2026-05-26
Topic: sds
Type: test

Question

在确认 `SCN` rebuilt input 的真实样本数是 `2716` diploid (`5432` haplotypes) 后，验证：

1. 把 `Gravel_CHB + Ne(0)=100000` 的 gamma `SAMPLE_SIZE` 从旧的 `2510` 改成匹配 `5432`，shape 会不会系统性下降？
2. 这类修正是否已经在实际 `SCN chr1` compute 的已完成 chunk 上体现为 `|rSDS|` 回落，尤其是最坏的 low-frequency common bins？

Input paths

- old gamma:
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/gravel_chb_present100000.g_file.txt`
- SCN sample-size-matched gamma:
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_ne100k_scn5432_smokegrid_20260526/gravel_chb_present100000.g_file.txt`
- old SCN chr1 SDS:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/SCN/chr1.sds.tsv`
- matched-gamma partial chunk outputs:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_scn5432matched_chr1_6_20260526/SCN/chunk_submit/chr1_gravel_chb_ne100k_scn5432matched_SCN_chr1_manual_20260526_024730/p_chunks/p_chunk_10.sds.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_scn5432matched_chr1_6_20260526/SCN/chunk_submit/chr1_gravel_chb_ne100k_scn5432matched_SCN_chr1_manual_20260526_024730/p_chunks/p_chunk_11.sds.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_scn5432matched_chr1_6_20260526/SCN/chunk_submit/chr1_gravel_chb_ne100k_scn5432matched_SCN_chr1_manual_20260526_024730/p_chunks/p_chunk_12.sds.tsv`

Output paths

- full-grid gamma outdir:
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_ne100k_scn5432_smokegrid_20260526`
- matched-gamma compute outdir:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_scn5432matched_chr1_6_20260526`
- finalized matched-gamma summaries:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_scn5432matched_chr1_6_20260526/SCN/SCN.frequency_bins.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_scn5432matched_chr1_6_20260526/SCN/SCN.normalized.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_scn5432matched_chr1_6_20260526/SCN/diagnostics/SCN.gravel_chb_ne100k_scn5432matched.chrlist.chrom_tail_counts.tsv`
- manifest:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/gravel_chb_ne100k_scn5432matched_chrlist_20260526_024730_29111/manifest.tsv`

Parameters

- demographic model: `Gravel_CHB`
- `Ne(0)=100000`
- old gamma sample size: `2510`
- matched gamma sample size: `5432`
- chunk compare window: `chr1 p_chunk_10-12`

Scripts / jobs

- scripts:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/submit_gravel_chb_gamma_parallel.py`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/submit_gravel_chb_ne100k_chrlist.sh`
- full-grid gamma jobs:
  - chunks: `487075`
  - aggregates: `487076`
  - finalize: `487077`
- matched-gamma chr rerun chain:
  - `chr1_final = 487082`
  - `chr6_final = 487087`
  - `postprocess = 487088`
  - `diagnostics = 487089`

Result

## 1. Gamma shape shifts

与旧 `2510` 版相比，`5432` 版在整个 `0.05-0.95` 频带都系统性下降，大约是旧值的 `~0.72x`：

- `0.05-0.10`: mean ratio `0.7178`
- `0.10-0.20`: mean ratio `0.7175`
- `0.20-0.50`: mean ratio `0.7239`
- `0.50-0.95`: mean ratio `0.7278`

低频开头几行：

- `0.05`: `0.710343 -> 0.498386` (`0.7016x`)
- `0.06`: `0.689180 -> 0.494679` (`0.7178x`)
- `0.07`: `0.678997 -> 0.490684` (`0.7227x`)
- `0.08`: `0.674269 -> 0.484923` (`0.7192x`)
- `0.09`: `0.674550 -> 0.484737` (`0.7186x`)
- `0.10`: `0.672147 -> 0.488488` (`0.7268x`)

## 2. Partial `chr1` overlap compare

先看 `p_chunk_10-12` 的 `30000` 个 overlap 位点：

- old `|rSDS|`:
  - mean `1.3654`
  - median `0.7746`
  - `p90 = 3.2184`
  - `>=5`: `946`
  - `>=10`: `214`
- matched gamma `|rSDS|`:
  - mean `1.2654`
  - median `0.7200`
  - `p90 = 3.1148`
  - `>=5`: `838`
  - `>=10`: `150`
- `new < old` 的位点数：`21951 / 30000`

再只看 common-frequency overlap (`MAF > 0.01`，共 `4244` 个位点)：

- old `|rSDS|`:
  - mean `0.6253`
  - median `0.0617`
  - `p90 = 0.2166`
  - `>=5`: `93`
  - `>=10`: `93`
- matched gamma `|rSDS|`:
  - mean `0.3480`
  - median `0.0599`
  - `p90 = 0.2070`
  - `>=5`: `53`
  - `>=10`: `53`
- `new < old` 的位点数：`2534 / 4244`

按 common-frequency bins 再拆：

- `(0.01, 0.02]`, `n=410`
  - old mean `4.6573`
  - new mean `2.0848`
  - `>=5`: `80 -> 41`
  - `new < old`: `236`
- `(0.02, 0.03]`, `n=393`
  - old mean `1.2197`
  - new mean `0.9263`
  - `>=5`: `12 -> 11`
  - `new < old`: `278`
- `(0.03, 0.05]`, `n=382`
  - old mean `0.2173`
  - new mean `0.2169`
  - `>=5`: `1 -> 1`
  - `new < old`: `163`
- `(0.05, 1.0]`, `n=3059`
  - old mean `0.0595`
  - new mean `0.0573`
  - `>=5`: `0 -> 0`
  - `new < old`: `1857`

## 3. Finalized `chr1/chr6` compare after postprocess

上面的 overlap 只是先验证方向。等 `chr1/chr6` 的 matched-gamma rerun 正式收口后，再按与旧 `SCN` 完全同口径的 `chr1.sds.tsv / chr6.sds.tsv` 比较，结果如下。

### chr1

- old:
  - `common_n = 550839`
  - `|rSDS| mean = 0.867438`
  - `|rSDS| median = 0.0699`
  - `>=5 = 15629`
  - `>=10 = 15629`
- matched gamma:
  - `common_n = 550839`
  - `|rSDS| mean = 0.525509`
  - `|rSDS| median = 0.0679`
  - `>=5 = 10240`
  - `>=10 = 10240`

按频率区间拆：

- `(0.01,0.02]`, `n=68039`
  - old mean `5.031056`
  - new mean `2.551380`
  - `>=5: 12723 -> 7649`
- `(0.02,0.03]`, `n=37773`
  - old mean `2.385851`
  - new mean `1.914183`
  - `>=5: 2500 -> 2214`
- `(0.03,0.05]`, `n=48438`
  - old mean `0.370437`
  - new mean `0.346045`
  - `>=5: 403 -> 373`
- `(0.05,0.50]`, `n=396589`
  - old mean `0.069207`
  - new mean `0.067605`
  - `>=5: 3 -> 4`

### chr6

- old:
  - `common_n = 476899`
  - `|rSDS| mean = 0.784691`
  - `|rSDS| median = 0.0731`
  - `>=5 = 11850`
  - `>=10 = 11850`
- matched gamma:
  - `common_n = 476899`
  - `|rSDS| mean = 0.486096`
  - `|rSDS| median = 0.0710`
  - `>=5 = 7922`
  - `>=10 = 7922`

按频率区间拆：

- `(0.01,0.02]`, `n=53542`
  - old mean `5.031406`
  - new mean `2.633255`
  - `>=5: 9810 -> 6101`
- `(0.02,0.03]`, `n=29119`
  - old mean `2.134833`
  - new mean `1.719230`
  - `>=5: 1699 -> 1512`
- `(0.03,0.05]`, `n=41207`
  - old mean `0.363159`
  - new mean `0.336593`
  - `>=5: 331 -> 302`
- `(0.05,0.50]`, `n=353031`
  - old mean `0.078458`
  - new mean `0.076188`
  - `>=5: 10 -> 7`

### Tail counts

旧版是全基因组 diagnostics，新版这次只做了 `chr1/chr6`，所以只比较这两条染色体：

- `chr1`
  - `>=20: 207 -> 199`
  - `>=50: 71 -> 40`
  - `>=100: 36 -> 8`
- `chr6`
  - `>=20: 178 -> 163`
  - `>=50: 67 -> 45`
  - `>=100: 45 -> 13`

Conclusion

- `SCN` 的 gamma contract 中，`SAMPLE_SIZE=2510` 是真实错配；改成匹配 `5432` 后，gamma shape 会在全频带系统性下降。
- 这种修正不是纸面上的：在已经完成的 `chr1` chunk overlap 上，`|rSDS|` 已经真实回落。
- 改善最明显的正是之前最坏的 low-frequency common bin：
  - `(0.01,0.02]` 的 mean `|rSDS|` 从 `4.6573` 降到 `2.0848`
  - `>=5` 的位点从 `80` 降到 `41`
- 正式收口后的 `chr1/chr6` compare 进一步坐实：
  - `sample-size mismatch` 是 `SCN` 异常 Manhattan 的真实原因之一；
  - 修正后最坏的 low-frequency common bins 在两条染色体上都明显回落；
  - `>=50` / `>=100` 的极端 tails 也同步收缩。
- 但它还不是全部答案：
  - `chr1/chr6` 的 `(0.01,0.02]` mean `|rSDS|` 虽然从 `~5.03` 压到了 `~2.55/2.63`，仍然偏高；
  - 因此 `SCN` 后续还需要全基因组 matched-gamma rerun，确认是否还有额外的 low-frequency model mismatch。

Next step

- 把 matched `SCN` gamma 扩到全基因组 `chr1-22`，看 genome-wide Manhattan 是否整体回到 plausible biology。
- `NCN` 继续单独拆，因为它本来就匹配 `2510`，所以其坏 Manhattan 还需要另一层解释。
