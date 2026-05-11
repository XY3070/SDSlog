# NCN gamma-track compare review

Date: 2026-05-03
Topic: sds
Type: summary
Question: `SDS-MODEL-001`、`SDS-MODEL-003`、`SDS-MODEL-002` 三条 gamma 轨道里，哪一条的 norm SDS Manhattan plot 最接近 real biology，哪两条应废弃？

## Input paths

- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_mainline/NCN/NCN.normalized.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_mainline/NCN/NCN.significant_regions.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_mainline/NCN/NCN.frequency_bins.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k/NCN/NCN.normalized.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k/NCN/NCN.significant_regions.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k/NCN/NCN.frequency_bins.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scaled_ne0_100000/NCN/NCN.normalized.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scaled_ne0_100000/NCN/NCN.significant_regions.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scaled_ne0_100000/NCN/NCN.frequency_bins.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_diagnostic_comparison/NCN_gamma_tracks.summary.md`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_diagnostic_comparison/NCN_gamma_tracks.chrom_tail_compare.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_diagnostic_comparison/NCN_gamma_tracks.top_windows.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_diagnostic_comparison/NCN_gamma_tracks.top_hits.tsv`

## Output paths

- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/2026-05-03_ncn_gamma_track_compare_review.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/plots/sds/20260503_NCN_sds_olddefault_mainline_manhattan_v01.png`
- `/data/home/grp-wangyf/xuyuan/SDSlog/plots/sds/20260503_NCN_sds_gravel_chb_ne100k_manhattan_v01.png`
- `/data/home/grp-wangyf/xuyuan/SDSlog/plots/sds/20260503_NCN_sds_phlash_scaled_ne100k_manhattan_v01.png`

## Parameters

- `compare_job=484562`
- `diagnose_jobs_v2=484559,484560,484561`
- `common_variant_count=7191531`
- `thresholds=-log10(p)>=8,20,50,100`
- `cluster_window_bp=100000`
- `olddefault_tail_counts=583,205,62,2`
- `gravel_tail_counts=4545,3839,2714,1498`
- `phlash_tail_counts=9717,3337,1129,483`
- `olddefault_dense_windows_ge20=2`
- `gravel_dense_windows_ge20=20`
- `phlash_dense_windows_ge20=20`
- `olddefault_top_chr1_window=143200001-143300000 hits=117 max=115.7976199`
- `olddefault_second_chr1_window=125100001-125200000 hits=86 max=146.0615389`

## Demographic model

| Track | Demographic model | Gamma shape source | Present-day `Ne(0)` | Role | Note |
| --- | --- | --- | --- | --- | --- |
| `olddefault` | `SDS-MODEL-001` | `/data/home/grp-wangyf/xuyuan/sds/g_file.txt` | `implicit / unknown from historical file provenance` | `mainline` | 当前唯一保留的主线 |
| `gravel_chb_ne100k` | `SDS-MODEL-003` | `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/gravel_chb_present100000.g_file.txt` | `100000` | `rejected` | `Gravel_CHB` shape，biology compare 未通过 |
| `phlash_scaled_ne100k` | `SDS-MODEL-002` | `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/scaled_ne0_100000/scaled_ne0_100000.minimal_g.tsv` | `100000` | `rejected` | phlash `q50` shape，biology compare 未通过 |

## Scripts / commands / job IDs

- `job 484559`: `sds_olddefault_diag_v2`
- `job 484560`: `sds_gravel_chb_diag_v2`
- `job 484561`: `sds_phlash100k_diag_v2`
- `job 484562`: `sds_gamma_track_compare_v2`
- `python3 /data/home/grp-wangyf/xuyuan/sds/scripts/compare_sds_diagnostics.py --track olddefault=... --track gravel_chb_ne100k=... --track phlash_scaled_ne100k=... --output-prefix /data/home/grp-wangyf/xuyuan/sds/data/processed/sds_diagnostic_comparison/NCN_gamma_tracks`

## Key plots

- [20260503_NCN_sds_olddefault_mainline_manhattan_v01.png](../../plots/sds/20260503_NCN_sds_olddefault_mainline_manhattan_v01.png)
- [20260503_NCN_sds_gravel_chb_ne100k_manhattan_v01.png](../../plots/sds/20260503_NCN_sds_gravel_chb_ne100k_manhattan_v01.png)
- [20260503_NCN_sds_phlash_scaled_ne100k_manhattan_v01.png](../../plots/sds/20260503_NCN_sds_phlash_scaled_ne100k_manhattan_v01.png)

## Key result files

- `SDS-017`: `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_mainline/NCN/NCN.normalized.tsv`
- `SDS-018`: `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_mainline/NCN/NCN.significant_regions.tsv`
- `SDS-019`: `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/gravel_chb_present100000.g_file.txt`
- `SDS-020`: `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_diagnostic_comparison/NCN_gamma_tracks.summary.md`

## Conclusion

- `SDS-MODEL-001` 明显优于另外两条线。Manhattan 主体接近“多数点贴近基线、只有少数局部 cluster”的形状；全基因组 common-variant extreme-hit totals 只有 `583 / 205 / 62 / 2`，对应 `>=8 / >=20 / >=50 / >=100`。
- `SDS-MODEL-003` 和 `SDS-MODEL-002` 都不符合 real biology。两者都有明显 diffuse tail 和大量高悬散点；`SDS-MODEL-003` 的 extreme-hit totals 达到 `4545 / 3839 / 2714 / 1498`，`SDS-MODEL-002` 达到 `9717 / 3337 / 1129 / 483`。
- `SDS-MODEL-001` 还没有完全过关：最主要的剩余问题是 chr1 上两个强局部峰，分别在 `143.2-143.3 Mb` 和 `125.1-125.2 Mb`。它们是局部 cluster，不是全基因组 diffuse manifestation，但仍需 locus-level audit 后才能作为 paper-facing biology signal 使用。
- low-MAF normalization 也支持废弃 `SDS-MODEL-002` / `SDS-MODEL-003`：`[0.01,0.02)` bin 中 `SDS-MODEL-001` 的 `sd=0.277`，而 `SDS-MODEL-002` 的 `sd=6.026`、`mean=1.675`，`SDS-MODEL-003` 的 `sd=0.638`，都显示 recent-demography gamma 对低频 bin 造成了明显扭曲。

## Next step

- 冻结 `SDS-MODEL-001` 为当前 mainline，`SDS-MODEL-002` 与 `SDS-MODEL-003` 都降级为 rejected sensitivity。
- 对 `SDS-MODEL-001` 下 chr1 `125.1 Mb` 和 `143.2 Mb` 两个窗口做 locus-level audit，优先检查是否和已知 ADH 之外的输入特征、局部 singletons、recombination 或 sample split artifact 相关。
- 后续 Manhattan、GWAS overlap、candidate locus 解读统一基于 `SDS-MODEL-001`，不再用 `SDS-MODEL-002` 或 `SDS-MODEL-003` 版本做主图。
