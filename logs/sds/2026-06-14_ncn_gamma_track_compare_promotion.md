Date: 2026-06-14
Topic: sds
Type: audit

Question

把 2026-05-03 的 same-input gamma compare 进一步推广到 `SDSworkspace`，并生成一张三联 `CMplot` 证据图，用来作为“不同 gamma file 对 Manhattan 形状影响很大”的另一组论据。

Input paths

- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_mainline/NCN/cmplot/NCN.cmplot_p5e4_maf001.manhattan.png`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scaled_ne0_100000/NCN/cmplot/NCN.cmplot_p5e4_maf001.manhattan.png`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k/NCN/cmplot/NCN.cmplot_p5e4_maf001.manhattan.png`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_diagnostic_comparison/NCN_gamma_tracks.summary.md`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_diagnostic_comparison/NCN_gamma_tracks.chrom_tail_compare.tsv`

Canonical workspace paths

- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/audit/ncn_gamma_track_compare_20260503/README.md`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/audit/ncn_gamma_track_compare_20260503/olddefault_cmplot.manhattan.png`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/audit/ncn_gamma_track_compare_20260503/phlash_scaled_ne100k_cmplot.manhattan.png`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/audit/ncn_gamma_track_compare_20260503/gravel_chb_ne100k_cmplot.manhattan.png`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/audit/ncn_gamma_track_compare_20260503/ncn_gamma_tracks_cmplot_montage.png`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/audit/ncn_gamma_track_compare_20260503/NCN_gamma_tracks.summary.md`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/audit/ncn_gamma_track_compare_20260503/NCN_gamma_tracks.chrom_tail_compare.tsv`

Provenance paths

- `/data/home/grp-wangyf/xuyuan/SDSworkspace/provenance/runs/ncn_gamma_track_compare_20260503/run_manifest.yaml`

Conclusion

- 这是更适合做汇报的 same-input gamma control 组：
  - `olddefault` 极值点显著更少；
  - `phlash_scaled_ne100k` 和 `Gravel_CHB_ne100k` 都显著更差；
  - 这说明在输入固定时，gamma provenance 本身就能改变 Manhattan 的整体形状。
- 由于对应 `CMplot` 版本本来就存在，没有必要重新计算，只需要做证据封装与归档。

Next step

- 如果你要继续扩充论据，我建议下一个方向不是再找更多 `Gravel_CHB@100k` 图，而是把这组 same-input compare 和前面的 tip-branch panel 并列成一个小段落，分别证明：
  - `Ne(0)` 主要改 scale
  - gamma file choice 主要改 Manhattan 形状
