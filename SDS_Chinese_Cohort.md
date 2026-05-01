# SDS Chinese Cohort

`SDSlog` 用于保存这个项目的事实记录。项目总览看这里，按时间的摘要看 [SDS log.md](SDS%20log.md)，详细记录看 [logs/index.md](logs/index.md)，关键结果和关键图分别看 [logs/result_index.md](logs/result_index.md) 与 [plots/plot_index.md](plots/plot_index.md)。

## 1. Project Goal

1. 用 SDS 找到 Chinese-specific 的 recent positive selection regions。
2. 用 tSDS 和 GWAS summary statistics 检查 polygenic selection signal。
3. 用 iHS / EHH、Relate + CLUES2、meta analysis、archaic introgression enrichment 解释强峰和候选 loci。

## 2. Current Data Snapshot

- 2nd QC 后保留 3971 个 WGS 样本，当前工作记录里 NCN / SCN 为 `1255 + 2716 = 3971`。
- 当前最近期 demography benchmark 的 NCN eligible sample count 为 `1255`，见 [2026-04-23_ncn_recent_ne_benchmark.md](logs/demography/2026-04-23_ncn_recent_ne_benchmark.md)。
- 当前主要工作根目录：
  - `sds/data/vcf`
  - `sds/data/processed/sds_input`
  - `sds/data/processed/sds_output`
  - `benchmark/demography`

## 3. Active Tracks

| Topic | Current state | Latest record |
| --- | --- | --- |
| Demography | NCN 的 phlash / SMC++ / Relate benchmark 文件和关键图已集中登记 | [2026-04-23_ncn_recent_ne_benchmark.md](logs/demography/2026-04-23_ncn_recent_ne_benchmark.md) |
| Genome-wide SDS | NCN genome-wide normalized output、significant regions、Manhattan plot 已登记 | [2026-04-09_ncn_genomewide_postprocess.md](logs/sds/2026-04-09_ncn_genomewide_postprocess.md) |
| Single-SNP sensitivity | `chr4:99317841:T:C` 的 gamma sensitivity 已登记，`q97.5` 已补完 | [2026-04-16_chr4_99317841_gamma_sensitivity.md](logs/sds/2026-04-16_chr4_99317841_gamma_sensitivity.md) |
| Ne0 panel | q2.5/q25/q50 context + scaled `Ne(0)` panel 已准备好 | [2026-04-26_ncn_region_ne0_scenario_panel.md](logs/sds/2026-04-26_ncn_region_ne0_scenario_panel.md) |
| iHS / EHH | 方向已定，结果文件和图还没有系统回填到 `SDSlog` | [logs/ihs_ehh/README.md](logs/ihs_ehh/README.md) |
| Meta overlap | 已有阶段性结论，但输入输出文件路径还没有系统回填到 `SDSlog` | [logs/meta_overlap/README.md](logs/meta_overlap/README.md) |
| Relate + CLUES2 | 已有代码与处理目录，但 paper-facing 结果还没有系统回填到 `SDSlog` | [logs/relate_clues2/README.md](logs/relate_clues2/README.md) |

## 4. Key Results In Hand

| Area | What is already archived | Entry |
| --- | --- | --- |
| Demography benchmark | benchmark manifest、recent-ne inputs manifest、phlash model、SMC++ / Relate key plots | [logs/result_index.md](logs/result_index.md) |
| Genome-wide SDS | `NCN.normalized.tsv`、`NCN.significant_regions.tsv`、Manhattan points、Manhattan plot | [2026-04-09_ncn_genomewide_postprocess.md](logs/sds/2026-04-09_ncn_genomewide_postprocess.md) |
| chr4 single SNP sensitivity | `summary.tsv`、`final_report.md`、scenario-level `rSDS` summary | [2026-04-16_chr4_99317841_gamma_sensitivity.md](logs/sds/2026-04-16_chr4_99317841_gamma_sensitivity.md) |
| Region Ne0 panel | `scenario_manifest.tsv`、`scenario_aliases.json`、parallel submit plan | [2026-04-26_ncn_region_ne0_scenario_panel.md](logs/sds/2026-04-26_ncn_region_ne0_scenario_panel.md) |

## 5. Working Decisions

- Singleton counts 在南北人群之间有系统差异，SDS 应分人群计算，不做简单 pooled scan。
- 当前 ancestry split 记录沿用现有设定：Neural-admixture unsupervised `k=4`，以 `comp 3 > 0.35` 定义北方成分。
- 当前 recent Ne sensitivity 的工作面板以 phlash `q50` 为缩放模板，主面板使用 `Ne(0)=100k, 152k, 250k, 500k, 750k, 1e6`；`q2.5/q25/q50` 作为 posterior context；`2.5e6/5e6` 作为 high-side diagnostic；`q97.5` 只做 smoke。
- 当前更看重 `norm_SDS`、rank stability、已知 peak 的方向一致性；raw `rSDS` 主要作为 diagnostic，不作为主要 paper-facing statistic。
- Chinese-specific peak 的正式定义应和既有文献的 threshold 对齐；弱信号更适合用 tSDS / summary-statistics 层面的 aggregate evidence，而不是直接给单点下结论。

## 6. Immediate Gaps

- 需要把 meta analysis 的输入、输出、overlap 结果文件路径补进 `logs/meta_overlap` 和 `logs/result_index.md`。
- 需要把 iHS / EHH 的显著区域表、gene label 输出和关键图补进 `SDSlog`。
- 需要把 Relate + CLUES2 的关键结果文件、图和参数补进 `SDSlog`。
- 需要在最终写文章前核对 genome-wide postprocess summary 中阈值记录的差异。
