# Logs Index

| Topic | Scope | Latest log | Status | Notes |
| --- | --- | --- | --- | --- |
| `cohort_qc` | sample QC、population split、freeze decision | No detailed log yet | backfill needed | 先补 1st / 2nd QC 和 NCN / SCN split 的文件路径 |
| `demography` | phlash、SMC++、Relate、recent `Ne` comparison | [2026-04-23_ncn_recent_ne_benchmark.md](demography/2026-04-23_ncn_recent_ne_benchmark.md) | active | 首批 benchmark 文件和关键图已登记 |
| `sds` | genome-wide scan、postprocess、single-SNP sensitivity、region panel | [2026-05-25_gravel_chb_tip_branch_ne_panel_launch.md](sds/2026-05-25_gravel_chb_tip_branch_ne_panel_launch.md) | active | 当前新增 `Gravel_CHB` 小型 `Ne(0)` tip-branch 诊断 panel（`60k-270k`, `DAF=0.05/0.07/0.10`, `100 reps`），用于确认低频边界下 `Ne(0)` 对 gamma shape 的影响。控制变量总表见 [2026-05-22_sds_control_variable_report_for_teacher.md](sds/2026-05-22_sds_control_variable_report_for_teacher.md)；demographic-model 单一真值见 [README_demographic_models.md](sds/README_demographic_models.md) |
| `ihs_ehh` | iHS、EHH、gene label、peak follow-up | [README.md](ihs_ehh/README.md) | backfill needed | 方向已确定，文件和图未系统登记 |
| `meta_overlap` | meta、GWAS overlap、tSDS | [README.md](meta_overlap/README.md) | backfill needed | 现有结论已写入总览，缺文件级记录 |
| `relate_clues2` | Relate、CLUES2、selection trajectory | [README.md](relate_clues2/README.md) | backfill needed | 需要 paper-facing 结果与图 |

## Priority Backfill

1. `cohort_qc`: 把最终纳入样本表、排除规则和南北划分文件路径补进来。
2. `ihs_ehh`: 把显著位点表、gene label 输出和关键图补进来。
3. `meta_overlap`: 把 Nyuwa overlap / meta 的输入输出路径补进来。
