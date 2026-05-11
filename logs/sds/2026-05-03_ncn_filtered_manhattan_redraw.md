# NCN filtered Manhattan redraw

Date: 2026-05-03
Topic: sds
Type: production
Question: 按 `sds/scripts/report_cmplot_manhattan.R` 的展示规则，只显示 `p < 0.0005` 且频率过滤通过的点后，三条 gamma 轨道的 Manhattan 图应如何重画并留档？

## Input paths

- `/data/home/grp-wangyf/xuyuan/sds/scripts/report_cmplot_manhattan.R`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/postprocess_sds_results.py`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_mainline/NCN/NCN.manhattan_points.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k/NCN/NCN.manhattan_points.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scaled_ne0_100000/NCN/NCN.manhattan_points.tsv`

## Output paths

- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_mainline/NCN/cmplot/NCN.cmplot_p5e4_maf001.manhattan.png`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k/NCN/cmplot/NCN.cmplot_p5e4_maf001.manhattan.png`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scaled_ne0_100000/NCN/cmplot/NCN.cmplot_p5e4_maf001.manhattan.png`
- `/data/home/grp-wangyf/xuyuan/SDSlog/plots/sds/20260503_NCN_sds_olddefault_mainline_manhattan_p5e4_v01.png`
- `/data/home/grp-wangyf/xuyuan/SDSlog/plots/sds/20260503_NCN_sds_gravel_chb_ne100k_manhattan_p5e4_v01.png`
- `/data/home/grp-wangyf/xuyuan/SDSlog/plots/sds/20260503_NCN_sds_phlash_scaled_ne100k_manhattan_p5e4_v01.png`

## Parameters

- `reference_script=/data/home/grp-wangyf/xuyuan/sds/scripts/report_cmplot_manhattan.R`
- `plot_p_threshold=0.0005`
- `plot_daf_min=0.01`
- `plot_daf_max=0.99`
- `plot_maf_threshold=0.01`
- `bonferroni_threshold=6.952622467e-09`
- `olddefault_points_plotted=28033`
- `gravel_chb_ne100k_points_plotted=12428`
- `phlash_scaled_ne100k_points_plotted=35146`

## Demographic model

| Track | Demographic model | Gamma shape source | Present-day `Ne(0)` | Role | Note |
| --- | --- | --- | --- | --- | --- |
| `olddefault_filtered_plot` | `SDS-MODEL-001` | `/data/home/grp-wangyf/xuyuan/sds/g_file.txt` | `implicit / unknown from historical file provenance` | `mainline` | 过滤版主图 |
| `gravel_filtered_plot` | `SDS-MODEL-003` | `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/gravel_chb_present100000.g_file.txt` | `100000` | `rejected` | 过滤后仍是 rejected track |
| `phlash_filtered_plot` | `SDS-MODEL-002` | `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/scaled_ne0_100000/scaled_ne0_100000.minimal_g.tsv` | `100000` | `rejected` | 过滤后仍是 rejected track |

## Scripts / commands / job IDs

- `awk 'BEGIN{FS=OFS="\t"} NR==1 || ($2+0)>3.30102999566 {print}' .../NCN.manhattan_points.tsv > .../cmplot/NCN.cmplot_p5e4_maf001.manhattan_points.tsv`
- `/data/home/grp-wangyf/intern/miniforge3/envs/sds/bin/python /data/home/grp-wangyf/xuyuan/sds/scripts/postprocess_sds_results.py --input-dir ... --output-prefix .../cmplot/NCN.cmplot_p5e4_maf001 --plot-points-tsv .../NCN.cmplot_p5e4_maf001.manhattan_points.tsv --bonferroni-threshold 6.952622467e-09 --plot-p-threshold 0.0005`

## Key plots

- [20260503_NCN_sds_olddefault_mainline_manhattan_p5e4_v01.png](../../plots/sds/20260503_NCN_sds_olddefault_mainline_manhattan_p5e4_v01.png)
- [20260503_NCN_sds_gravel_chb_ne100k_manhattan_p5e4_v01.png](../../plots/sds/20260503_NCN_sds_gravel_chb_ne100k_manhattan_p5e4_v01.png)
- [20260503_NCN_sds_phlash_scaled_ne100k_manhattan_p5e4_v01.png](../../plots/sds/20260503_NCN_sds_phlash_scaled_ne100k_manhattan_p5e4_v01.png)

## Key result files

- `SDS-021`: `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_mainline/NCN/cmplot/NCN.cmplot_p5e4_maf001.postprocess_summary.tsv`
- `SDS-022`: `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k/NCN/cmplot/NCN.cmplot_p5e4_maf001.postprocess_summary.tsv`
- `SDS-023`: `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scaled_ne0_100000/NCN/cmplot/NCN.cmplot_p5e4_maf001.postprocess_summary.tsv`

## Conclusion

- 这次重画严格沿用了 `report_cmplot_manhattan.R` 的展示阈值：`p < 0.0005`，并保留当前 pipeline 已有的 frequency gate，即 `DAF` 在 `(0.01, 0.99)` 且 `MAF > 0.01`。
- `SDS-MODEL-001` 的过滤版 Manhattan 仍然最合理；`SDS-MODEL-002` 与 `SDS-MODEL-003` 在只看 `p < 0.0005` 的前提下依然保留明显 diffuse tail 和多染色体高悬散点。
- 过滤版图更适合直接做人工审图与汇报，不替代 full normalized / significant-region 结果表。

## Next step

- 后续人工审图统一优先看这套 `cmplot_p5e4_maf001` 版本。
- locus-level audit 继续聚焦 `SDS-MODEL-001` 下 chr1 的 `125.1 Mb` 和 `143.2 Mb` 两个窗口。
