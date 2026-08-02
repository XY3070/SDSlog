# Logs Index

| Topic | Scope | Latest log | Status | Notes |
| --- | --- | --- | --- | --- |
| `cohort_qc` | sample QC、population split、freeze decision | [2026-06-16_singleton_ancestor_annotation_10chr.md](cohort_qc/2026-06-16_singleton_ancestor_annotation_10chr.md) | active | stepwise counts + diagnostic plots archived; singleton AA/DA annotation (10 chr, 94.6% rate) complete; new chromosomes pending |
| `demography` | phlash、SMC++、Relate、recent `Ne` comparison | [2026-04-23_ncn_recent_ne_benchmark.md](demography/2026-04-23_ncn_recent_ne_benchmark.md) | active | 首批 benchmark 文件和关键图已登记 |
| `sds` | genome-wide scan、postprocess、LD split、VEP annotation、orthogonal evidence、SLE vs non-SLE | [2026-08-02_sle_vs_nonsle_synthesis.md](sds/2026-08-02_sle_vs_nonsle_synthesis.md) | **SLE synthesis done** | 2026-08-02：ALL 按 SLE 拆分对比 (SLE=1965 vs nonSLE=2367)。**Synthesis**：5 条已建立事实：(1) singleton burden 与 region 强相关 (R²=0.29)；(2) 忽略 interaction 时 SLE 显示较低 singleton；(3) region×phenotype interaction 高度显著 (F=107, p≈0)；(4) SZ 与 HK 方向相反 (+375 vs −49)；(5) 不存在跨 region 一致的 SLE singleton burden。**结论：SLE vs non-SLE SDS 差异不能解释为疾病 selection 差异。**全局偏移是归一化 artifact；singleton 差异是 region×phenotype interaction。另见 [comparison](sds/2026-08-02_sle_vs_nonsle_sds_comparison.md)、[diagnostic](sds/2026-08-02_sle_vs_nonsle_confounder_diagnostic.md)、[synthesis](sds/2026-08-02_sle_vs_nonsle_synthesis.md)。 |
| `ihs_ehh` | iHS、EHH、gene label、peak follow-up | [2026-07-28_orthogonal_evidence_final_report.md](../sds/2026-07-28_orthogonal_evidence_final_report.md) | **iHS done, EHH failed** | Genome-wide iHS 22 chr complete (selscan + pyrho CHB). EHH all 15 signals failed (selscan stoi crash: SHAPEIT5 VCF duplicate positions). |
| `meta_overlap` | meta、GWAS overlap、tSDS | [README.md](meta_overlap/README.md) | backfill needed | 现有结论已写入总览，缺文件级记录 |
| `relate_clues2` | Relate、CLUES2、selection trajectory | [2026-07-31_relate_chr19_v7_success.md](../sds/2026-07-31_relate_chr19_v7_success.md) | **chr19 pilot done** | Relate chr19 tree inference complete (v7, 9 attempts to fix). 3.7GB .anc + 57MB .mut ready for RelateSelection. ALL phasing in progress for genome-wide Relate. |
| `population_structure` | ADMIXTURE、badMIXTURE、PCA-UMAP | [2026-06-26_population_structure_analysis.md](sds/2026-06-26_population_structure_analysis.md) | active | badMIXTURE 揭示离散模型 misfit (Row ACF ~0.86)；PCA-UMAP 证实 common/rare variants 连续梯度可补充 |

## Priority Backfill

1. `ihs_ehh`: 把显著位点表、gene label 输出和关键图补进来。
2. `meta_overlap`: 把 Nyuwa overlap / meta 的输入输出路径补进来。
3. `relate_clues2`: 把 trajectory / model compare 的图和路径补进来。
