Date: 2026-06-14
Topic: sds
Type: audit

Question

把 2026-05-03 的 same-input gamma compare 进一步整理成一个更适合汇报的 MAF sensitivity 证据包：

- 同一组 `NCN` 输入、同一组三条 gamma track；
- 重新生成 `CMplot` 图；
- 点画大一点；
- x 轴染色体标签改成只在第一条显示 `chr1`，其余直接用数字；
- 只保留两版对照：`MAF > 0.01` 和 `MAF > 0.05`。

Input paths

- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_mainline/NCN/NCN.normalized.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scaled_ne0_100000/NCN/NCN.normalized.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k/NCN/NCN.normalized.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_mainline/NCN/NCN.postprocess_summary.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scaled_ne0_100000/NCN/NCN.postprocess_summary.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k/NCN/NCN.postprocess_summary.tsv`

Workspace outputs

- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/audit/ncn_gamma_track_compare_20260503_maf_sensitivity_20260614/README.md`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/audit/ncn_gamma_track_compare_20260503_maf_sensitivity_20260614/maf_sensitivity.summary.tsv`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/audit/ncn_gamma_track_compare_20260503_maf_sensitivity_20260614/maf_sensitivity.summary.md`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/audit/ncn_gamma_track_compare_20260503_maf_sensitivity_20260614/ncn_gamma_track_compare_maf001_maf005_bigpoints.png`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/audit/ncn_gamma_track_compare_20260503_maf_sensitivity_20260614/maf001/olddefault_maf001_bigpoints.png`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/audit/ncn_gamma_track_compare_20260503_maf_sensitivity_20260614/maf001/phlash_scaled_ne100k_maf001_bigpoints.png`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/audit/ncn_gamma_track_compare_20260503_maf_sensitivity_20260614/maf001/gravel_chb_ne100k_maf001_bigpoints.png`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/audit/ncn_gamma_track_compare_20260503_maf_sensitivity_20260614/maf005/olddefault_maf005_bigpoints.png`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/audit/ncn_gamma_track_compare_20260503_maf_sensitivity_20260614/maf005/phlash_scaled_ne100k_maf005_bigpoints.png`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/audit/ncn_gamma_track_compare_20260503_maf_sensitivity_20260614/maf005/gravel_chb_ne100k_maf005_bigpoints.png`

Provenance

- `/data/home/grp-wangyf/xuyuan/SDSworkspace/provenance/runs/ncn_gamma_track_compare_20260503_maf_sensitivity_20260614/run_manifest.yaml`

Conclusion

- `olddefault` remains the cleanest baseline.
- `phlash_scaled_ne100k` still produces a heavier tail than `olddefault`.
- `Gravel_CHB_ne100k` remains the most diffuse of the three.
- Tightening the display filter from `MAF > 0.01` to `MAF > 0.05` sharply reduces the plotted set and makes the contrast easier to see.
- The new figure can be cited directly as a controlled, CMplot-based follow-up to the earlier same-input gamma compare.
