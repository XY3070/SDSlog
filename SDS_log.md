# SDS log

这是按时间维护的简明时间线。细节、参数、文件路径、关键图请看 [logs/index.md](logs/index.md)。

## Timeline

| Month | Summary | Detailed log |
| --- | --- | --- |
| 2025-05 | 拿到课题，开始读 SDS R 脚本、输入输出和 `s file` 准备流程；样本量当时记为 4197。 | Backfill pending |
| 2025-06 | 跑通 chr22 的 SMC++ 和 SDS smoke，开始得到 `rSDS` 与 norm；转到 `hpc2`。 | Backfill pending |
| 2025-07 | SMC++ 切到 singularity，补齐容器和集群运行经验。 | Backfill pending |
| 2025-08 | WGS preprocess 启动；做全常染色体 SMC++；开始下载 Nyuwa SDS 并学习做 meta。 | Backfill pending |
| 2025-09 | 学习用 PLINK 做 QC；提出用 phlash 替代 SMC++ 估 recent `Ne`。 | Backfill pending |
| 2025-11 | 明确增加 iHS / EHH 和 archaic introgression enrichment 方向。 | Backfill pending |
| 2025-12 | 做了第一版 meta analysis，记录里提到 overlap 点约 14k。 | Backfill pending |
| 2026-04 | 回到 NCN：完成 genome-wide SDS postprocess、chr4 单点 gamma sensitivity、region `Ne(0)` scenario panel。 | [2026-04-09_ncn_genomewide_postprocess.md](logs/sds/2026-04-09_ncn_genomewide_postprocess.md), [2026-04-16_chr4_99317841_gamma_sensitivity.md](logs/sds/2026-04-16_chr4_99317841_gamma_sensitivity.md), [2026-04-23_ncn_recent_ne_benchmark.md](logs/demography/2026-04-23_ncn_recent_ne_benchmark.md), [2026-04-26_ncn_region_ne0_scenario_panel.md](logs/sds/2026-04-26_ncn_region_ne0_scenario_panel.md) |
| 2026-05 | 正式重构 `SDSlog`：建立 README、detailed logs、result index、plot index，并归档首批关键图。 | 本次更新 |

## Notes To Keep

- 当前优先保证：每个关键测试或生产结果都有 detailed log，每张关键图都有 source path，每个关键结果文件都有 absolute path。
- 旧时间线保留，后续只补 concise summary；参数、命令、job ID、结果路径统一下沉到 `logs/`。
