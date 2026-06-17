# 2026-06-13 chr1 EAS arm-bin CMplot redraw

## Question

Use `CMplot` to redraw the `chr1` EAS gamma arm-normalized / custom MAF-bin Manhattan figure, and make the arm split come from `sds_rebuild/masks`.

## Input paths

- `/data/home/grp-wangyf/xuyuan/SDSworkspace/runs/chr1_qc_control_20260606/plots/eas_chr1_arm_bins.normalized.tsv`
- `/data/home/grp-wangyf/xuyuan/sds_rebuild/masks/chr_arm_manifest.tsv`
- `/data/home/grp-wangyf/xuyuan/sds_rebuild/masks/callable_autosome.bed`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/runs/chr1_qc_control_20260606/plots/render_eas_chr1_arm_bins_cmplot.R`

## Output paths

- `/data/home/grp-wangyf/xuyuan/SDSworkspace/runs/chr1_qc_control_20260606/plots/eas_chr1_arm_bins.png`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/runs/chr1_qc_control_20260606/plots/eas_chr1_arm_bins.pdf`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/runs/chr1_qc_control_20260606/plots/eas_chr1_arm_bins.cmplot_input.tsv`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/runs/chr1_qc_control_20260606/plots/eas_chr1_arm_bins.matplotlib.png`

## Implementation notes

- The old matplotlib PNG was preserved as `eas_chr1_arm_bins.matplotlib.png`.
- The redraw script uses `chr_arm_manifest.tsv` to derive the chr1 p/q arm boundaries.
- The redraw script uses `callable_autosome.bed` to drop non-callable chr1 positions before plotting.
- `CMplot` is rendered with pseudo-chromosome labels `chr1 p` and `chr1 q`.

## Result summary

- Input normalized rows: `548557`
- Rows after arm + callable mask filter and `p < 0.005`: `8494`
- Bonferroni threshold used for the horizontal line: `5.886508123e-06`
- Plot backend: `CMplot (R)`

## Revision

On 2026-06-14, the figure was overwritten to use true mask coordinates on a single chr1 axis.

- Old p/q split CMplot figure was replaced by the single-axis version in `eas_chr1_arm_bins.png`
- `eas_chr1_arm_bins.pdf` was removed
- The new title is `chr1 EAS gamma (true mask coordinates)`

## Command

```bash
/data/home/grp-wangyf/intern/miniforge3/envs/sds/bin/Rscript \
  /data/home/grp-wangyf/xuyuan/SDSworkspace/runs/chr1_qc_control_20260606/plots/render_eas_chr1_arm_bins_cmplot.R \
  --input-normalized-tsv /data/home/grp-wangyf/xuyuan/SDSworkspace/runs/chr1_qc_control_20260606/plots/eas_chr1_arm_bins.normalized.tsv \
  --output-prefix /data/home/grp-wangyf/xuyuan/SDSworkspace/runs/chr1_qc_control_20260606/plots/eas_chr1_arm_bins \
  --callable-bed /data/home/grp-wangyf/xuyuan/sds_rebuild/masks/callable_autosome.bed
```
