# Logs Index

| Topic | Scope | Latest log | Status | Notes |
| --- | --- | --- | --- | --- |
| `cohort_qc` | sample QC、population split、freeze decision | No detailed log yet | backfill needed | 先补 1st / 2nd QC 和 NCN / SCN split 的文件路径 |
| `demography` | phlash、SMC++、Relate、recent `Ne` comparison | [2026-04-23_ncn_recent_ne_benchmark.md](demography/2026-04-23_ncn_recent_ne_benchmark.md) | active | 首批 benchmark 文件和关键图已登记 |
| `sds` | genome-wide scan、postprocess、single-SNP sensitivity、region panel | [2026-06-06_input_root_bridge_adoption.md](sds/2026-06-06_input_root_bridge_adoption.md) | **rebuild planning** | 2026-06-03 的系统性反思已经把问题收敛到三层 blocking issue：region mask / b_file、SNP input contract、gamma provenance。2026-06-05 先从项目结构下手，正式把工作区拆成 `SDSlog/`、`SDSpipeline/`、`SDSworkspace/` 三层：汇报写作、可复用 pipeline、非 git 运行空间各自独立；随后进一步把“跨服务器路径差异”和“大文件非对称可用性”固化为明确 contract：tracked repo 只保留通用逻辑，服务器本地路径只放 `config/paths.env`，一次性桥接脚本只放 `SDSworkspace/oneoff/`。2026-06-06 先把当前服务器的外部依赖入口切到 `SDSworkspace/external/`，再把 VCF / sample list 入口切到 `SDSworkspace/input/`，两者都先用 symlink bridge 形成 canonical root，再逐步考虑物理迁移。第一性原理的 workflow blueprint 仍是结构重构的上位设计，详见 [2026-06-05_first_principles_rebuild_workflow_compare.md](sds/2026-06-05_first_principles_rebuild_workflow_compare.md)；结构解耦详见 [2026-06-05_project_structure_decoupling.md](sds/2026-06-05_project_structure_decoupling.md) |
| `ihs_ehh` | iHS、EHH、gene label、peak follow-up | [README.md](ihs_ehh/README.md) | backfill needed | 方向已确定，文件和图未系统登记 |
| `meta_overlap` | meta、GWAS overlap、tSDS | [README.md](meta_overlap/README.md) | backfill needed | 现有结论已写入总览，缺文件级记录 |
| `relate_clues2` | Relate、CLUES2、selection trajectory | [README.md](relate_clues2/README.md) | backfill needed | 需要 paper-facing 结果与图 |

## Priority Backfill

1. `cohort_qc`: 把最终纳入样本表、排除规则和南北划分文件路径补进来。
2. `ihs_ehh`: 把显著位点表、gene label 输出和关键图补进来。
3. `meta_overlap`: 把 Nyuwa overlap / meta 的输入输出路径补进来。
