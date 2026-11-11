# Logs Index

| Topic | Scope | Latest log | Status | Notes |
| --- | --- | --- | --- | --- |
| `cohort_qc` | sample QC、population split、freeze decision | [2026-06-16_singleton_ancestor_annotation_10chr.md](cohort_qc/2026-06-16_singleton_ancestor_annotation_10chr.md) | active | stepwise counts + diagnostic plots archived; singleton AA/DA annotation (10 chr, 94.6% rate) complete; new chromosomes pending |
| `demography` | phlash、SMC++、Relate、recent `Ne` comparison | [2026-04-23_ncn_recent_ne_benchmark.md](demography/2026-04-23_ncn_recent_ne_benchmark.md) | active | 首批 benchmark 文件和关键图已登记 |
| `sds` | genome-wide scan、postprocess、LD split、VEP annotation、orthogonal evidence、SLE vs non-SLE | [2026-08-25_chr19_member_level_clump_audit.md](sds/2026-08-25_chr19_member_level_clump_audit.md) | active | member-aware edge + complete-link clumping 完成：R1=5 个多lead局部块+9 singleton；R2=3 个多lead局部块+5 singleton；最终独立性等待 Relate。 |
| `ihs_ehh` | iHS、EHH、gene label、peak follow-up | [2026-10-12_st3_ihs_sensitivity_and_postprocess.md](ihs_ehh/2026-10-12_st3_ihs_sensitivity_and_postprocess.md) | iHS sensitivity + ST3 CSV complete; EHH status unchanged | Full-marker scan retained; EPO=1 pooled DAF normalization; 184-row ST3 CSV with REF/ALT/AA/DA. EPO=0 signed direction intentionally blank. |
| `meta_overlap` | meta、GWAS overlap、tSDS | [README.md](meta_overlap/README.md) | backfill needed | 现有结论已写入总览，缺文件级记录 |
| `relate_clues2` | Relate、CLUES2、selection trajectory | [2026-11-11_platform_bias_check_chb_chs.md](relate_clues2/2026-11-11_platform_bias_check_chb_chs.md) | active | 平台假象排查完成：位点朝向两群一致；无 DP/GQ 断崖；NCN 缺失率/稀有突变偏高但为全染色体背景差；1KG CHB/CHS 同平台对照在零质量差下仍检出 DPP9(z=7.49)/DNM2(z=5.91) 差异 → 无明显平台假象。真实位点已用队列 coal 重算（job 880728 DONE）。 |
| `population_structure` | ADMIXTURE、badMIXTURE、PCA-UMAP | [2026-06-26_population_structure_analysis.md](sds/2026-06-26_population_structure_analysis.md) | active | badMIXTURE 揭示离散模型 misfit (Row ACF ~0.86)；PCA-UMAP 证实 common/rare variants 连续梯度可补充 |

| `genetic_maps` | 家系遗传图 (crossover counting, ZhongHua 芯片位点) | [2026-08-16_genetic_maps_zhonghua_genomewide.md](genetic_maps/2026-08-16_genetic_maps_zhonghua_genomewide.md) | **genome-wide done** | 247 trio (204 中文 + 44 CHS), 全基因组 5Mb Spearman vs b38 = 0.904; CO/mei pat 35.8 / mat 38.0 (性别比 1.06, mat 欠调 ~5-10%); 旧记录已移至本目录 (sle_family_locuszoom, microarray_fastsmc) |
## Priority Backfill

1. `ihs_ehh`: 把显著位点表、gene label 输出和关键图补进来。
2. `meta_overlap`: 把 Nyuwa overlap / meta 的输入输出路径补进来。
3. `relate_clues2`: 把 trajectory / model compare 的图和路径补进来。
