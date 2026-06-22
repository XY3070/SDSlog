# 2026-06-17 chr4 SDS with singleton_qc_v1 + EAS gamma + frozen pipeline

## Summary

完成 chr4 SDS 计算，使用 `raw/9k/snp_singleton_qc_chr4_v1/` 数据。确定了 EAS gamma（`gamma_shapes.Gravel_EAS.n100` triplet 1 shape）为唯一权威 gamma，并将 pipeline 约定冻结到 `SDSpipeline`。

## 输入

| 文件 | 来源 | 说明 |
|---|---|---|
| `chr4_s_file.txt` (28M, 4393 人) | `filtered_singletons.vcf.gz` AC=1, masked | 排除 132 个 exclusion/density-outlier 区间 |
| `chr4_t_file.txt` (3.2G, 387,325 SNPs) | `snp_qc_pass.vcf.gz` AF 0.05-0.95, F_MISSING≤0.05, masked | 同上 mask |
| `chr4_o_file.txt` | 全 1 | 4393 样本 |
| `chr4_b_file.txt` | GRCh38 chr4 centromere | p: 1-49712060, q: 51743952-190214555 |
| Gamma | `g_file.Gravel_EAS.txt` | `gamma_shapes.Gravel_EAS.n100` triplet 1 shape, deduplicated |

## Mask

合并 `singleton_exclusion_regions.bed` (127 chr4 区间) + `singleton_density_outlier_regions.bed` (20 区间) → 132 merged regions。通过 `bcftools view -T "^mask.bed"` 在 s_file 和 t_file 生成时排除。

## 计算

- 队列: normal (auto-sense, score 8.8, 50 free slots)
- chunk_rows=10000, array_parallel=37, chunk_job_slots=1
- LSF jobs: p-arm 490729, q-arm 490730, merge 490731/490732, final 490733

## Gamma 调查

发现 `gamma_shapes.Gravel_EAS.n100` 有 6 列：两个 triplet `(DAF, col2, col3)`。

| 解释 | 验证 | 结论 |
|---|---|---|
| (DAF, shape, scale) | mean²/var = shape 恒等式成立；CV=0.73 合理 | **正确** |
| (DAF, mean, sd) | shape=11.4, CV=0.30 过紧 | 排除 |

col2 = shape 参数，已被 `compute_SDS.py` 正确使用。DAF=0.15 和 0.85 有 4 行重复（两个 triplet 交叉），已去重取均值。

## 旧 run 对比

| | 旧 run (Jun 16, `tmp/chr4_output/`) | 新 run (Jun 17) |
|---|---|---|
| Gamma | `Gravel_EAS.n100.2col.txt` (同一 EAS gamma) | `g_file.Gravel_EAS.txt` (同) |
| 输入 VCF | `raw/9k/snp_singleton_qc/snp_qc_pass.vcf.gz` (genome-wide, 80GB) | `snp_singleton_qc_chr4_v1/snp_qc_pass.vcf.gz` (chr4, 2.4GB) |
| 样本数 | 1255 | 4393 |
| Mask | 无 | 132 regions |
| SNPs | 389,773 | 379,533 |
| rSDS range | [-0.74, 0.77] | [-0.64, 2.13] |

主体分布一致 (SD≈0.10)。差异在于新 VCF 包含一个 artifact SNP `chr4:8184879` (99.8% 杂合, HWE MIDP=3.4e-1286)，该 SNP 在旧 genome-wide VCF 中不存在。

## QC pipeline HWE bug

`chr4:8184879` 的 HWE p=3.4e-1286 远低于阈值 1e-6，但出现在 `variant_pass_hwe.txt` 中。根因未确定，疑似 resume 模式下文件不一致。此 SNP 应在 QC 阶段被过滤但未过滤。

## 结果

| 文件 | 路径 |
|---|---|
| SDS output | `SDSworkspace/results/production/sds_output/NCN/chr4.sds.tsv` |
| Parquet | `SDSworkspace/results/production/sds_output/NCN/chr4.sds.parquet` |
| Normalized | `SDSworkspace/results/production/sds_output/NCN/NCN_chr4.normalized.tsv` |
| Frequency bins | `SDSworkspace/results/production/sds_output/NCN/NCN_chr4.frequency_bins.tsv` |
| Manhattan | `SDSworkspace/results/production/sds_output/NCN/NCN_chr4_manhattan.png` |
| Significant regions | `SDSworkspace/results/production/sds_output/NCN/NCN_chr4.significant_regions.tsv` |

rSDS: n=379,533, mean=0.0178, SD=0.0997, range=[-0.64, 2.13]

Top 信号: chr4:8,184,879 (rSDS=2.13, artifact), chr4:8,254,567 (1.24), chr4:103,597,722 (1.10), chr4:27,948,139 (0.80)

## Pipeline 冻结

以下约定已硬编码到 `SDSpipeline/`:

1. **t_file AF 过滤**: `generate_t_file.sh:29` — `AF>=0.05 && AF<=0.95`
2. **t_file HWE 过滤**: `generate_t_file.sh:28-29` — `HWE>=1e-6`（排除 paralog/CNV 杂合子过剩 artifact；t_file 检查发现 246/387325 SNPs 应被过滤，其中 211 个 obs_het≈0.97-0.99）
3. **Norm 分层**: `postprocess_sds_results.py` — MAF<0.1 bin=0.05, MAF≥0.1 bin=0.1 (硬编码，无开关)
4. **matplotlib**: 已从 `postprocess_sds_results.py` 删除
5. **CMplot**: `report_cmplot_manhattan.R` — PNG only (无 PDF), p-threshold 默认 1 (不过滤)
6. **Gamma**: `g_file.Gravel_EAS.txt` 为所有 pop 默认 gamma (`common_env.sh:find_default_g_file` 最高优先级)
