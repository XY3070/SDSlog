# SDSlog

`SDSlog` 是这个项目的事实记录仓库。这里记录 4 类内容：

1. 项目总览与方法决策。
2. 每个关键测试 / 生产结果的 detailed log。
3. 关键结果文件的 absolute path。
4. 关键图的仓库副本、命名和原始来源。

## Working Rules

1. 每个关键测试或生产结果都必须有一条 `logs/` 记录。
2. 每条记录都必须写清楚 `Input paths`、`Output paths`、`Parameters`、`Conclusion`、`Next step`。
3. 关键图必须 copy 到 `plots/`，并在 [plots/plot_index.md](plots/plot_index.md) 里登记原始 absolute path。
4. 关键结果文件默认只登记路径，不把大文件复制进 `SDSlog`。
5. 语言以中文为主，必要 terminology 用 English；只写 concise and precise 的事实，不写含混术语。
6. 对所有 SDS 结果，必须记录 `Demographic model`、`Gamma shape source`、`Present-day Ne(0)`、`Model role`；标准名字统一见 [logs/sds/README_demographic_models.md](logs/sds/README_demographic_models.md)。

## Entry Points

- 项目总览：[SDS Chinese Cohort.md](SDS%20Chinese%20Cohort.md)
- 时间线摘要：[SDS log.md](SDS%20log.md)
- 详细日志索引：[logs/index.md](logs/index.md)
- 关键结果索引：[logs/result_index.md](logs/result_index.md)
- 关键图索引：[plots/plot_index.md](plots/plot_index.md)
- 日志模板：[logs/_log_template.md](logs/_log_template.md)

## Directory Layout

- `logs/cohort_qc`: cohort freeze、sample inclusion / exclusion、population split
- `logs/demography`: phlash / SMC++ / Relate / `Ne(0)` sensitivity
- `logs/sds`: genome-wide SDS、postprocess、single-locus sensitivity、region panel
- `logs/ihs_ehh`: iHS、EHH、gene label、follow-up plots
- `logs/meta_overlap`: meta analysis、GWAS overlap、tSDS follow-up
- `logs/relate_clues2`: Relate、CLUES2、single-locus selection trajectory
- `plots/<topic>`: 对应 topic 的关键图副本

## Naming

- Detailed log: `logs/<topic>/YYYY-MM-DD_<slug>.md`
- Plot file: `plots/<topic>/YYYYMMDD_<cohort>_<analysis>_<keyparam>_<slug>_v01.<ext>`
- Result ID: `DEMO-...`, `SDS-...`, `IHS-...`, `META-...`, `CLUES-...`
- Plot ID: `PLT-DEMO-...`, `PLT-SDS-...`

## Update Procedure

1. 先完成测试或生产。
2. 把关键结果文件路径写进 [logs/result_index.md](logs/result_index.md)。
3. 把关键图 copy 进 `plots/`，并更新 [plots/plot_index.md](plots/plot_index.md)。
4. 若是 SDS 结果，先在 [logs/sds/README_demographic_models.md](logs/sds/README_demographic_models.md) 里确认 model / gamma / `Ne(0)` 口径，再写 result index 和 plot index。
5. 用 [logs/_log_template.md](logs/_log_template.md) 新建一条 detailed log。
6. 如有新的方法学决策或项目主线变化，再更新 [SDS Chinese Cohort.md](SDS%20Chinese%20Cohort.md) 和 [SDS log.md](SDS%20log.md)。
