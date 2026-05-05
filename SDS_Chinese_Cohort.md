# SDS Chinese Cohort

`SDSlog` 用于保存这个项目的事实记录。项目总览看这里，按时间的摘要看 [SDS_log.md](SDS_log.md)，详细记录看 [logs/index.md](logs/index.md)，关键结果和关键图分别看 [logs/result_index.md](logs/result_index.md) 与 [plots/plot_index.md](plots/plot_index.md)。

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
| Genome-wide SDS | 三条 gamma 轨道的 compare 已完成；当前只有 old/default mainline 保留，phlash-derived `scaled_ne0_100000` 与 `Gravel_CHB@100k` 都因 diffuse tail 被否决 | [2026-05-03_ncn_gamma_track_compare_review.md](logs/sds/2026-05-03_ncn_gamma_track_compare_review.md) |
| Single-SNP sensitivity | `chr4:99317841:T:C` 的 gamma sensitivity 已登记，`q97.5` 已补完 | [2026-04-16_chr4_99317841_gamma_sensitivity.md](logs/sds/2026-04-16_chr4_99317841_gamma_sensitivity.md) |
| Ne0 panel | q2.5/q25/q50 context + scaled `Ne(0)` panel 已准备好；`100k` only 的 chunked gamma submitter 和 local smoke 已验证 | [2026-05-02_ncn_region_ne0_chunked_gamma_acceleration.md](logs/sds/2026-05-02_ncn_region_ne0_chunked_gamma_acceleration.md) |
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
- 当前 recent `Ne` 的 gamma compare 已出结论：phlash-derived `scaled_ne0_100000` 与 `Gravel_CHB@100k` 都不能作为 mainline；当前唯一保留轨道是 old/default `g_file.txt`。
- 当前更看重 `norm_SDS`、rank stability、已知 peak 的方向一致性；raw `rSDS` 主要作为 diagnostic，不作为主要 paper-facing statistic。
- 当前 phlash-derived `scaled_ne0_100000` run 只保留 sensitivity/reference 身份；mainline gamma anchor 已切回 old/default [g_file.txt](../sds/g_file.txt) 的 provenance 轨道。
- 当前 `Gravel_CHB@100k` 的 gamma generation 已改成 chunked parallel path：`910` 个 chunk jobs、`91` 个 aggregate jobs、`1` 个 finalize job，避免再次卡在串行 gamma 模拟。
- 当前 cross-track compare 支持正式废弃 phlash 与 `Gravel_CHB@100k`：它们都产生了明显 diffuse tail 和过量的极端 common-variant hits；old/default 仍有 chr1 两个强峰，但整体 null shape 最接近 real biology。
- 当前 `Ne(0)=100k` 的 gamma generation 优先走 chunked parallel path；forced regeneration 时按 `10 frequencies x 10 chunks/frequency = 100 chunk jobs` 展开，再汇总成 `10` 个 canonical full pieces。
- Chinese-specific peak 的正式定义应和既有文献的 threshold 对齐；弱信号更适合用 tSDS / summary-statistics 层面的 aggregate evidence，而不是直接给单点下结论。

## 6. Immediate Gaps

- 需要把 meta analysis 的输入、输出、overlap 结果文件路径补进 `logs/meta_overlap` 和 `logs/result_index.md`。
- 需要把 iHS / EHH 的显著区域表、gene label 输出和关键图补进 `SDSlog`。
- 需要把 Relate + CLUES2 的关键结果文件、图和参数补进 `SDSlog`。
- 需要在最终写文章前核对 genome-wide postprocess summary 中阈值记录的差异。
- 需要对 old/default 下 chr1 的 `125.1 Mb` 和 `143.2 Mb` 两个窗口做 locus-level audit，再决定哪些 peak 可以进入 paper-facing biological interpretation。
