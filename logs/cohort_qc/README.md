# cohort_qc

这里记录 Chinese cohort 的 sample QC、SNP QC、singleton QC 以及 freeze decision。

## Canonical standards

### Sample QC

- 输入先做 metadata harmonization，再做 missingness。
- `MAF >= 0.05` 是 het / KING 的前置过滤，不再用全变异集合。
- `F_MISS >= 0.05` 的样本剔除。
- heterozygosity 只看低端异常，阈值是 `mean - 5 SD` 的 `F_het`。
- KING cutoff 用 `0.0884`，对应二级亲缘及以内剔除。
- PCA 只在 post-KING 样本集上做最终判定。
- singleton burden 是最后的诊断项，不先于 relatedness / PCA 作为主过滤。

### SNP QC

- 先保留 frozen samples、biallelic PASS SNPs。
- `chrX/chrY` 要先去掉 PAR1 / PAR2。
- autosome 的 variant missingness 阈值是 `0.05`。
- autosome 的 HWE 阈值是 `p >= 1e-6`。
- sex chromosomes 的 HWE 走 bypass。
- DP / QUAL 最终门槛是 `QUAL >= 56` 且 `DP_AVG` 落在 cohort mean 的 `[1/2, 2x]`。

### Singleton QC

- 从 final SNP-QC VCF 里提取 `AC==1`。
- 去掉低置信度区域、blacklist、PAR。
- 用 `20 kb` window、`10 kb` step 做 density filter，超过 `mean + 4 SD` 的窗口剔除。
- singleton site missingness 必须同时满足：
  - `missing_count <= 10`
  - `missing_rate <= 0.005`
- 通过 site gate 后再看 sample singleton burden，异常高者作为诊断候选。

## Current files

- [raw/9k README](/data/home/grp-wangyf/xuyuan/raw/9k/README.md)
- [SDS Chinese Cohort](/data/home/grp-wangyf/xuyuan/SDSlog/SDS%20Chinese%20Cohort.md)
- [SDS log](/data/home/grp-wangyf/xuyuan/SDSlog/SDS%20log.md)
- [Raw 9k QC counts and plot archive](./2026-06-14_raw9k_qc_counts_and_plots.md)
