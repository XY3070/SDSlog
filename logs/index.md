# Logs Index

| Topic | Scope | Latest log | Status | Notes |
| --- | --- | --- | --- | --- |
| `cohort_qc` | sample QC、population split、freeze decision | No detailed log yet | backfill needed | 先补 1st / 2nd QC 和 NCN / SCN split 的文件路径 |
| `demography` | phlash、SMC++、Relate、recent `Ne` comparison | [2026-04-23_ncn_recent_ne_benchmark.md](demography/2026-04-23_ncn_recent_ne_benchmark.md) | active | 首批 benchmark 文件和关键图已登记 |
| `sds` | genome-wide scan、postprocess、single-SNP sensitivity、region panel | [2026-05-11_sdslog_demography_provenance_backfill.md](sds/2026-05-11_sdslog_demography_provenance_backfill.md) | active | raw-joint `chr1` `v2` 三轨 compare 已完成；chr1 local compare 图已归档；demographic-model 单一真值已落到 [README_demographic_models.md](sds/README_demographic_models.md) |
| `ihs_ehh` | iHS、EHH、gene label、peak follow-up | [README.md](ihs_ehh/README.md) | backfill needed | 方向已确定，文件和图未系统登记 |
| `meta_overlap` | meta、GWAS overlap、tSDS | [README.md](meta_overlap/README.md) | backfill needed | 现有结论已写入总览，缺文件级记录 |
| `relate_clues2` | Relate、CLUES2、selection trajectory | [README.md](relate_clues2/README.md) | backfill needed | 需要 paper-facing 结果与图 |

## Priority Backfill

1. `cohort_qc`: 把最终纳入样本表、排除规则和南北划分文件路径补进来。
2. `ihs_ehh`: 把显著位点表、gene label 输出和关键图补进来。
3. `meta_overlap`: 把 Nyuwa overlap / meta 的输入输出路径补进来。
