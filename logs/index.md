# Logs Index

| Topic | Scope | Latest log | Status | Notes |
| --- | --- | --- | --- | --- |
| `cohort_qc` | sample QC、population split、freeze decision | No detailed log yet | backfill needed | 先补 1st / 2nd QC 和 NCN / SCN split 的文件路径 |
| `demography` | phlash、SMC++、Relate、recent `Ne` comparison | [2026-04-23_ncn_recent_ne_benchmark.md](demography/2026-04-23_ncn_recent_ne_benchmark.md) | active | 首批 benchmark 文件和关键图已登记 |
| `sds` | genome-wide scan、postprocess、single-SNP sensitivity、region panel | [2026-06-06_chr1_qc_controlled_sds_test.md](sds/2026-06-06_chr1_qc_controlled_sds_test.md) | **rebuild testing** | 2026-06-03 的系统性反思把问题收敛到 region mask / b_file、SNP input contract、gamma provenance；2026-06-05-06 已完成 `SDSlog/`、`SDSpipeline/`、`SDSworkspace/` 三层结构、跨服务器 path contract、external/input root adoption 和 processed legacy cleanup。第一性原理 blueprint 见 [2026-06-05_first_principles_rebuild_workflow_compare.md](sds/2026-06-05_first_principles_rebuild_workflow_compare.md)。2026-06-06 新增 chr1 controlled QC test：同一 baseline input 下逐一测试 mask、variant QC、singleton-density QC、excess-singleton sample、observability 和 cumulative arm；结果显示 `peak125/peak143` 可由 mask 或 singleton-density QC 控制，variant-only / observability-only 不足，详见最新 log。 |
| `ihs_ehh` | iHS、EHH、gene label、peak follow-up | [README.md](ihs_ehh/README.md) | backfill needed | 方向已确定，文件和图未系统登记 |
| `meta_overlap` | meta、GWAS overlap、tSDS | [README.md](meta_overlap/README.md) | backfill needed | 现有结论已写入总览，缺文件级记录 |
| `relate_clues2` | Relate、CLUES2、selection trajectory | [README.md](relate_clues2/README.md) | backfill needed | 需要 paper-facing 结果与图 |

## Priority Backfill

1. `cohort_qc`: 把最终纳入样本表、排除规则和南北划分文件路径补进来。
2. `ihs_ehh`: 把显著位点表、gene label 输出和关键图补进来。
3. `meta_overlap`: 把 Nyuwa overlap / meta 的输入输出路径补进来。
