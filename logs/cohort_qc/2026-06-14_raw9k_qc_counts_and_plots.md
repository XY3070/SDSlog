# Raw 9k QC Counts and Plot Archive

Date: 2026-06-14
Topic: cohort_qc
Type: summary
Question: What are the stepwise sample QC / SNP QC / singleton QC counts, and where are the diagnostic plots?

## Input paths

- `/data/home/grp-wangyf/xuyuan/raw/9k/sample_qc/qc_summary_report.md`
- `/data/home/grp-wangyf/xuyuan/raw/9k/sample_qc/sample_qc_metrics.tsv`
- `/data/home/grp-wangyf/xuyuan/raw/9k/sample_qc/sample_qc_chr1_maf05/cohort_freeze.tsv`
- `/data/home/grp-wangyf/xuyuan/raw/9k/sample_qc/heterozygosity.het`
- `/data/home/grp-wangyf/xuyuan/raw/9k/sample_qc/pca.eigenvec`
- `/data/home/grp-wangyf/xuyuan/raw/9k/sample_qc/pca_postking.eigenvec`
- `/data/home/grp-wangyf/xuyuan/raw/9k/sample_qc/plot_qc_diagnostics.py`
- `/data/home/grp-wangyf/xuyuan/raw/9k/snp_singleton_qc_chr20_v3/snp_qc_stats.tsv`
- `/data/home/grp-wangyf/xuyuan/raw/9k/snp_singleton_qc_chr20_v3/snp_singleton_qc_report.md`
- `/data/home/grp-wangyf/xuyuan/raw/9k/snp_singleton_qc_chr20_v3/singleton_filter_stats.tsv`
- `/data/home/grp-wangyf/xuyuan/raw/9k/snp_singleton_qc_chr20_v3/singleton_quality.tsv`

## Output paths

- `/data/home/grp-wangyf/xuyuan/raw/9k/sample_qc/sample_qc_heterozygosity.png`
- `/data/home/grp-wangyf/xuyuan/raw/9k/sample_qc/sample_qc_pca.png`
- `/data/home/grp-wangyf/xuyuan/raw/9k/sample_qc/sample_qc_pca_postking.png`
- `/data/home/grp-wangyf/xuyuan/raw/9k/sample_qc/sample_qc_umap.png`
- `/data/home/grp-wangyf/xuyuan/raw/9k/sample_qc/sample_qc_umap_postking.png`
- `/data/home/grp-wangyf/xuyuan/raw/9k/sample_qc/qc_plot_removed_samples.tsv`
- `/data/home/grp-wangyf/xuyuan/raw/9k/sample_qc/qc_plot_summary.txt`
- `/data/home/grp-wangyf/xuyuan/SDSlog/plots/cohort_qc/sample_qc_heterozygosity.png`
- `/data/home/grp-wangyf/xuyuan/SDSlog/plots/cohort_qc/sample_qc_pca.png`
- `/data/home/grp-wangyf/xuyuan/SDSlog/plots/cohort_qc/sample_qc_umap.png`

## Parameters

- Sample QC contract: `MAF >= 0.05` before het / KING; `F_MISS < 0.05`; het low tail only; KING cutoff `0.0884`; PCA after KING; singleton burden last.
- SNP QC contract: frozen samples, biallelic PASS SNPs, PAR removed on `chrX/chrY`, autosome missingness `<= 0.05`, autosome HWE `>= 1e-6`, sex chromosome HWE bypass, final `QUAL >= 56` and `DP_AVG` within `[mean/2, mean*2]`.
- Singleton QC contract: `20 kb` window, `10 kb` step, density cutoff `mean + 4 SD`; site missingness must satisfy `missing_count <= 10` and `missing_rate <= 0.005`.

## Scripts / commands / job IDs

- `raw/9k/sample_qc/plot_qc_diagnostics.py`
- `raw/9k/sample_qc/pca.eigenvec` regenerated with:
  - `/data/home/grp-wangyf/leiyao/Tools/PLINK2/plink2 --pfile raw/9k/sample_qc/chinese --extract raw/9k/sample_qc/pruned.prune.in --pca 10 --out raw/9k/sample_qc/pca`
- `raw/9k/snp_singleton_qc_chr20_v3/snp_singleton_qc_report.md`

## Key plots

- [Het scatter](../../plots/cohort_qc/sample_qc_heterozygosity.png)
- [Pre-KING PCA scatter](../../plots/cohort_qc/sample_qc_pca.png)
- [Post-KING PCA scatter](../../plots/cohort_qc/sample_qc_pca_postking.png)
- [Pre-KING UMAP scatter](../../plots/cohort_qc/sample_qc_umap.png)
- [Post-KING UMAP scatter](../../plots/cohort_qc/sample_qc_umap_postking.png)

## Sample QC counts

| Step | Before | After | Removed |
| --- | ---: | ---: | ---: |
| Input cohort | 4929 | 4929 | 0 |
| Missingness filter | 4929 | 4919 | 10 |
| Het filter | 4919 | 4911 | 8 |
| KING filter | 4911 | 4393 | 518 |
| PCA outlier filter | 4393 | 4393 | 0 |
| Singleton burden filter | 4393 | 4393 | 0 |

Note: the `MAF >= 0.05` prefilter on chr1 did not change the final sample list; `cohort_freeze.tsv` stayed identical.
Note: KING report text also flags 526 related hits, but 8 of those overlap with het outliers; the stepwise table above records unique sample counts after each stage.

## SNP QC counts

Reference run: `chr20_v3`.

| Step | Before | After | Removed |
| --- | ---: | ---: | ---: |
| Raw variants in `chinese` | 16913111 | 16913111 | 0 |
| Frozen subset + biallelic PASS + PAR removal | 16913111 | 4276150 | 12636961 |
| Variant missingness `<= 0.05` | 4276150 | 4212451 | 63699 |
| Autosome HWE `>= 1e-6` | 4212451 | 4208768 | 3683 |
| DP / QUAL gate | 4208768 | 4202933 | 5835 |

DP / QUAL gate used mean DP `39.003273`, low cutoff `19.501637`, high cutoff `78.006546`, and `QUAL >= 56`.

## Singleton QC counts

Reference run: `chr20_v3`.

| Step | Count |
| --- | ---: |
| Raw singletons | 1086913 |
| After blacklist exclusion | 1086913 |
| Density outlier windows | 15 |
| Singletons removed by density | 9572 |
| Final singletons | 1077341 |

| Singleton site QC | Count |
| --- | ---: |
| Total singleton sites | 1077341 |
| Passing sites | 1018942 |
| Failing sites | 58399 |

## Key result files

- `sample_qc_metrics.tsv`: per-sample QC status
- `cohort_freeze.tsv`: final sample list
- `controls_freeze.txt`: final control list
- `snp_qc_stats.tsv`: variant QC step counts
- `singleton_filter_stats.tsv`: singleton region/density counts
- `singleton_quality.tsv`: singleton-site missingness and observability
- `singleton_quality_pass.tsv`: singleton sites passing the double missingness gate

## Conclusion

- Sample QC final freeze is `4393` samples.
- The `chr1` `MAF >= 0.05` rerun gives the exact same freeze as the earlier sample QC.
- The reference SNP QC run ends at `4,202,933` variants.
- The reference singleton QC run ends at `1,077,341` singletons and `1,018,942` passing singleton sites.

## PCA cluster review

- The circled post-KING PCA cluster contains `THOMSON_4`, `THOMSON_5`, `IMR-90`, `HK173C`, and `C463C`.
- All five are `HongKong` controls in the current metadata.
- They are close to each other in the first 10 PCs, but they do not cross the current per-PC `6 SD` outlier rule.
- Their approximate distance from the post-KING PC1/PC2 centroid is `0.038-0.052` in PC units.
- Their PC2 z-scores are about `2.13-3.28`; none reaches the current `6 SD` PCA outlier rule.
- Current decision: keep them for now, but treat them as a manual review cluster rather than automatic outliers.

## Positive het outliers vs KING

- High positive `F_het` samples total `15`.
- KING removed `5` of them: `THOMSON_1`, `THOMSON_3`, `VH108A`, `VH108B`, `VH108C`.
- The remaining high `F_het` samples were not dropped by KING under the current `0.0884` cutoff.

## Next step

- Use this log and the plot index as the only detailed record for counts and diagnostic figures.
