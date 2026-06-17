# Logs Index

| Topic | Scope | Latest log | Status | Notes |
| --- | --- | --- | --- | --- |
| `cohort_qc` | sample QC、population split、freeze decision | [2026-06-16_singleton_ancestor_annotation_10chr.md](cohort_qc/2026-06-16_singleton_ancestor_annotation_10chr.md) | active | stepwise counts + diagnostic plots archived; singleton AA/DA annotation (10 chr, 94.6% rate) complete; new chromosomes pending |
| `demography` | phlash、SMC++、Relate、recent `Ne` comparison | [2026-04-23_ncn_recent_ne_benchmark.md](demography/2026-04-23_ncn_recent_ne_benchmark.md) | active | 首批 benchmark 文件和关键图已登记 |
| `sds` | genome-wide scan、postprocess、single-SNP sensitivity、region panel | [2026-06-14_ncn_gamma_track_compare_promotion.md](sds/2026-06-14_ncn_gamma_track_compare_promotion.md) | **rebuild testing** | 2026-06-03 的系统性反思把问题收敛到 region mask / b_file、SNP input contract、gamma provenance；2026-06-05-06 已完成 `SDSlog/`、`SDSpipeline/`、`SDSworkspace/` 三层结构、跨服务器 path contract、external/input root adoption 和 processed legacy cleanup。2026-06-14 进一步把 2026-05-25 的 Gravel_CHB tip-branch `Ne(0)` panel、2026-05-03 的 same-input gamma compare，以及 2026-06-14 的 MAF sensitivity rerender 分别归档进 `SDSworkspace`，补成一组可直接引用的论据图。第一性原理 blueprint 见 [2026-06-05_first_principles_rebuild_workflow_compare.md](sds/2026-06-05_first_principles_rebuild_workflow_compare.md)。 |
| `ihs_ehh` | iHS、EHH、gene label、peak follow-up | [README.md](ihs_ehh/README.md) | backfill needed | 方向已确定，文件和图未系统登记 |
| `meta_overlap` | meta、GWAS overlap、tSDS | [README.md](meta_overlap/README.md) | backfill needed | 现有结论已写入总览，缺文件级记录 |
| `relate_clues2` | Relate、CLUES2、selection trajectory | [README.md](relate_clues2/README.md) | backfill needed | 需要 paper-facing 结果与图 |

## Priority Backfill

1. `ihs_ehh`: 把显著位点表、gene label 输出和关键图补进来。
2. `meta_overlap`: 把 Nyuwa overlap / meta 的输入输出路径补进来。
3. `relate_clues2`: 把 trajectory / model compare 的图和路径补进来。
