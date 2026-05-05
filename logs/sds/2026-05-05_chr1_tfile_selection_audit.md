# chr1 `t_file` Selection Audit

Date: 2026-05-05
Topic: sds
Type: test
Question: 当前 `NCN chr1_t_file` 是否接近把 sample-subset 后的全部可用 SNP 都当成了 test SNP，从而改变了 chr1 的 site universe？

## Input paths

- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/NCN/chr1_t_file.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/NCN/chr1_s_file.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/legacy/chr1_t_file.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/legacy/chr1_s_file.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/legacy/chr1_sds_res_FULL.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_mainline/NCN/NCN.normalized.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_diagnostic_comparison/NCN_gamma_tracks.top_windows.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/vcf/NCN/UKBQC_NCN_chr1.vcf.gz`
- `/data/home/grp-wangyf/xuyuan/sds/data/NCN.txt`

## Output paths

- `/data/home/grp-wangyf/xuyuan/sds/tmp/chr1_legacy_vs_current_20260505_033208/summary.md`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/chr1_legacy_vs_current_20260505_033208/window_summary.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/chr1_tfile_sparse_like_legacy_20260505/chr1_t_file.txt`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/chr1_sparse_shadow_q_20260505/chr1_q.sds.tsv`

## Parameters

- `windows=W1_peak(chr1:143200001-143300000),W2_secondary(chr1:125100001-125200000),C1_quiet(chr1:35735220-36063259),C2_matched(chr1:43344983-43444982)`
- `singleton_check=distribution_only`
- `bin_size=1Mb`
- `current_chr1_t_bins=230`
- `legacy_chr1_t_bins=232`

## Scripts / commands / job IDs

- `/data/home/grp-wangyf/xuyuan/sds/scripts/audit_chr1_legacy_vs_current.py`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/thin_t_file_by_density.py`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/run_chr1_sparse_shadow_q.sh`
- `awk -F '\t' 'BEGIN{bin=1000000} {b=int(($4-1)/bin)+1; c[b]++} END{...}' current/legacy chr1_t_file.txt`
- `bcftools view -H -r 1:143200001-143300000 -S /data/home/grp-wangyf/xuyuan/sds/data/NCN.txt -m2 -M2 -v snps --force-samples /data/home/grp-wangyf/xuyuan/sds/data/vcf/NCN/UKBQC_NCN_chr1.vcf.gz | wc -l`
- `bcftools view -H -r 1:125100001-125200000 -S /data/home/grp-wangyf/xuyuan/sds/data/NCN.txt -m2 -M2 -v snps --force-samples /data/home/grp-wangyf/xuyuan/sds/data/vcf/NCN/UKBQC_NCN_chr1.vcf.gz | wc -l`
- `python /data/home/grp-wangyf/xuyuan/sds/scripts/thin_t_file_by_density.py ... --target-per-bin 1000`
- `bsub job 484650: chr1 sparse shadow q-arm rerun`

## Key plots

- None archived

## Key result files

- `SDS-024`: `/data/home/grp-wangyf/xuyuan/sds/tmp/chr1_legacy_vs_current_20260505_033208/summary.md`
- `SDS-025`: `/data/home/grp-wangyf/xuyuan/sds/tmp/chr1_tfile_sparse_like_legacy_20260505/chr1_t_file.txt`
- `SDS-026`: `/data/home/grp-wangyf/xuyuan/sds/tmp/chr1_sparse_shadow_q_20260505/chr1_q.sds.tsv`

## Conclusion

- 当前 `NCN chr1_t_file` 在 `W1` 和 `W2` 的行数分别为 `3248` 和 `4186`；当前源 `VCF` 在同一 sample subset、同一窗口下的 bi-allelic SNP 数也是 `3248` 和 `4186`，一一对应。
- 当前 `t_file` 更像 near-full SNP universe，而不是稀疏 test-SNP panel。chr1 1 Mb density 为 `mean=14431.8, min=2951, max=20657`；legacy 为 `mean=963.384, min=38, max=1001`，口径明显不同。
- `legacy chr1_sds_res_FULL.txt` 只覆盖 `chr1:1638357-15564144`，不能直接用于反驳当前 `125.1 Mb / 143.2 Mb` 两个 chr1 异常峰。
- 已生成一版 density-matched shadow `t_file`：chr1 总行数 `230000`，`W1=295`、`W2=513`、`C1=290`、`C2=92`，可直接用于后续 chr1 rerun。
- q-arm shadow rerun 已提交为 `LSF 484650`，输出日志将写到 `/data/home/grp-wangyf/xuyuan/sds/tmp/chr1_sparse_shadow_q_20260505/chr1_sparse_shadow_q.%J.out`。
- `LSF 484650` 已完成。shadow q-arm 并没有给出更干净的结果：`W2` 从 `16.1465` 仅降到 `14.4373`，`W1` 从 `16.7957` 降到 `11.8486`，但 q-arm control `150.0 Mb` 仍保持 `12.8304`；同时 top hits 仍散到 `145 Mb / 146 Mb / 206 Mb / 223 Mb`。
- 这说明“把 `t_file` 稀疏化到 legacy-like density”不是正确修复，至少不是主因；它只能改变行数和部分峰高，不能恢复一个明显更合理的 biology pattern。

## Next step

- 优先追查另一台服务器生成 `legacy t_file` 的真实脚本或 commit，确认是否做了 test-SNP thinning / density control / site prefilter。
- 停止沿 `t_file density thinning` 继续扩展。下一步应回到“全部 test SNP 都应进入 `t_file`”这个前提，重点审当前 split 后 `VCF/site QC/filtering` 与 legacy 上游位点 universe 的差异，而不是再人工稀疏化 `t_file`。
