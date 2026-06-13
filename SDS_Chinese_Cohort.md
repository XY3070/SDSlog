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
  - `SDSlog/`
  - `SDSpipeline/`
  - `SDSworkspace/`
  - 服务器本地实际路径由未跟踪的 `SDSpipeline/config/paths.env` 解析

## 3. Active Tracks

| Topic | Current state | Latest record |
| --- | --- | --- |
| Demography | NCN 的 phlash / SMC++ / Relate benchmark 文件和关键图已集中登记 | [2026-04-23_ncn_recent_ne_benchmark.md](logs/demography/2026-04-23_ncn_recent_ne_benchmark.md) |
| Genome-wide SDS | 三条 gamma 轨道的 compare 已完成；当前只有 `SDS-MODEL-001` 保留为 mainline，`SDS-MODEL-002` 与 `SDS-MODEL-003` 都因 diffuse tail 被否决 | [2026-05-03_ncn_gamma_track_compare_review.md](logs/sds/2026-05-03_ncn_gamma_track_compare_review.md) |
| Workflow rebuild | 已完成从第一性原理重建 SDS workflow 的 blueprint，并逐项对照现有 pipeline 该保留和该重写的环节 | [2026-06-05_first_principles_rebuild_workflow_compare.md](logs/sds/2026-06-05_first_principles_rebuild_workflow_compare.md) |
| Project structure | 已把工作区正式拆成 `SDSlog/`、`SDSpipeline/`、`SDSworkspace/` 三层，并在 `SDSpipeline` 里建立 canonical script snapshot、path contract 和 provenance template | [2026-06-05_project_structure_decoupling.md](logs/sds/2026-06-05_project_structure_decoupling.md) |
| Cross-server portability | 已把“大文件路径随服务器变化”的约束正式写成 contract：tracked repo 只保留通用脚本，服务器本地路径只放 `config/paths.env`，一次性桥接脚本只放 `SDSworkspace/oneoff/` | [2026-06-05_cross_server_portability_contract.md](logs/sds/2026-06-05_cross_server_portability_contract.md) |
| External root adoption | 当前服务器已把 `ms`、`relate`、`smcpp` image/rootfs/runtime 等外部依赖统一挂到 `SDSworkspace/external/` 下，先以 symlink bridge 方式形成 canonical external root | [2026-06-06_external_root_bridge_adoption.md](logs/sds/2026-06-06_external_root_bridge_adoption.md) |
| Input root adoption | 当前服务器已把 cohort VCF 和常用 freeze sample list 入口统一挂到 `SDSworkspace/input/` 下，先以 symlink bridge 方式形成 canonical input root | [2026-06-06_input_root_bridge_adoption.md](logs/sds/2026-06-06_input_root_bridge_adoption.md) |
| Processed legacy cleanup | 已把 `sds/data/processed` 中的 scheduler、compare、audit、experiment-input 和零引用长名 output 收口到 `legacy_*` grouped roots，并只删除空壳对象 | [2026-06-06_processed_legacy_cleanup.md](logs/sds/2026-06-06_processed_legacy_cleanup.md) |
| Variant / singleton QC | 已在 `SDSpipeline` 中补入 input QC contract、chr-level mask builder、controlled post-filter 和 window summarizer；NCN chr1 target test 证明 `peak125/peak143` 可由 mask 或 singleton-density QC 控制，variant-only 与 observability-only 不足 | [2026-06-06_chr1_qc_controlled_sds_test.md](logs/sds/2026-06-06_chr1_qc_controlled_sds_test.md) |
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
- 当前 recent `Ne` 的 gamma compare 已出结论：`SDS-MODEL-002` 与 `SDS-MODEL-003` 都不能作为 mainline；当前唯一保留轨道是 `SDS-MODEL-001`。
- 当前更看重 `norm_SDS`、rank stability、已知 peak 的方向一致性；raw `rSDS` 主要作为 diagnostic，不作为主要 paper-facing statistic。
- 当前 `SDS-MODEL-002` 只保留 sensitivity/reference 身份；mainline gamma anchor 已切回 `SDS-MODEL-001` 的 provenance 轨道，即 [g_file.txt](../sds/g_file.txt)。
- 当前 `SDS-MODEL-003` 的 gamma generation 已改成 chunked parallel path：`910` 个 chunk jobs、`91` 个 aggregate jobs、`1` 个 finalize job，避免再次卡在串行 gamma 模拟。
- 当前 cross-track compare 支持正式废弃 `SDS-MODEL-002` 与 `SDS-MODEL-003`：它们都产生了明显 diffuse tail 和过量的极端 common-variant hits；`SDS-MODEL-001` 仍有 chr1 两个强峰，但整体 null shape 最接近 real biology。
- 当前 `Ne(0)=100k` 的 gamma generation 优先走 chunked parallel path；forced regeneration 时按 `10 frequencies x 10 chunks/frequency = 100 chunk jobs` 展开，再汇总成 `10` 个 canonical full pieces。
- Chinese-specific peak 的正式定义应和既有文献的 threshold 对齐；弱信号更适合用 tSDS / summary-statistics 层面的 aggregate evidence，而不是直接给单点下结论。
- **chr1:125.1M 确认为 pericentromeric heterochromatin (1q12 HSATII) 的 mapping artifact**，不应进入 paper-facing biological interpretation，峰内无任何蛋白编码基因。**chr1:143.2M 峰本身也在无基因的 1q12 异染色质内，但紧邻 1q21.1 proximal segmental duplication / TAR susceptibility 区域**（RBM8A 等基因在 ~2 Mb 外），该坐标段落在结构复杂区（BP1-BP4 断点间 recurrent CNV），不能轻率宣称选择信号——即使有真实成分也是 mapping 复杂性与 structural variation 的混合。两个峰都不应作为 paper-facing 正选择证据。
- **b_file 生成逻辑需要修正**：当前 chr1 和 chr6 的 b_file 把着丝粒区域包含进了 arm 范围（chr1 q-arm 起始在着丝粒内部 252 kb，chr6 q-arm 深入着丝粒 2.3 Mb），导致 satellite DNA 上的 variant 进入 SDS 计算。
- **SNP input contract 需要扩展**：当前 PASS + biallelic + F_MISSING 过滤不足以排除着丝粒和极端 rare variant 的干扰；需要增加 pericentromeric/subtelomeric mask 和 DAF 下限。
- **gamma provenance 是当前必须解决的 blocking 问题**：olddefault shape (0.026-0.25) 与 Gravel_CHB@100k shape (0.63-0.71) 相差 3-27 倍，但 olddefault 来源不明且不可复现。在解决 provenance 之前，暂停依赖 olddefault 的新 compute run。
- **chr6 MHC 区域在所有 baseline 中都没有正选择信号**，与 MHC 是已知人类正选择最强区域的预期矛盾。说明 pipeline 存在"假阳性敏感、假阴性不敏感"的结构性问题。需要用 iHS/EHH 做正交验证。
- **SCN hybrid-policy rerun 后 recovered-only 窗口的 norm_SDS inflation 是 normalization 层 artifact**，不是真实生物学信号。四个最极端峰（chr1:125M, chr9:61.6M, chr16:46.4M + peak143）全部不在蛋白编码基因上。
- **2026-06-05 已输出 first-principles rebuild blueprint**：当前不再建议继续直接追加 compute run，而是按 [2026-06-05_first_principles_rebuild_workflow_compare.md](logs/sds/2026-06-05_first_principles_rebuild_workflow_compare.md) 先补 `cohort_qc`、analysis mask、shared site universe 与 gamma acceptance。
- **2026-06-05 已完成项目结构解耦**：`SDSlog/` 保留为汇报与写作 repo，新增 `SDSpipeline/` 作为可复用脚本 repo，新增 `SDSworkspace/` 作为非 git 的 input/results/runroot/provenance 空间。详见 [2026-06-05_project_structure_decoupling.md](logs/sds/2026-06-05_project_structure_decoupling.md)。
- **2026-06-05 已完成跨服务器路径 contract**：tracked pipeline 默认通过 `common_env.sh` + 本地 `config/paths.env` 解析路径；不同服务器的大文件落点、外部工具路径和一次性桥接脚本都不再写回 git tracked 代码。详见 [2026-06-05_cross_server_portability_contract.md](logs/sds/2026-06-05_cross_server_portability_contract.md)。
- **2026-06-06 已完成当前服务器 external root adoption**：本机 `ms`、`relate`、`smcpp` image/rootfs、matplotlib config、singularity config 和 system runtime 已统一通过 `SDSworkspace/external/` 暴露为 canonical path，`paths.env` 也已切换到这层入口。详见 [2026-06-06_external_root_bridge_adoption.md](logs/sds/2026-06-06_external_root_bridge_adoption.md)。
- **2026-06-06 已完成当前服务器 input root adoption**：本机 cohort VCF 与常用 freeze sample list 已统一通过 `SDSworkspace/input/` 暴露为 canonical path，`SDS_VCF_ROOT` 也已切换回 `${SDS_INPUT_ROOT}/raw/vcf`。详见 [2026-06-06_input_root_bridge_adoption.md](logs/sds/2026-06-06_input_root_bridge_adoption.md)。
- **2026-06-06 chr1 controlled QC test 已证明问题可控**：在同一 NCN chr1 baseline input 下，`mask_only` 和 `singleton_density_only` 都能把 `peak125/peak143` target windows 的 SDS 输出降为 0，同时基本保留 `control35`；`variant_only`、`excess_sample_only`、`observability_callrate_only` 不能单独解决 peak。结论：chr1 125/143 Mb 主要是 analysis-mask / singleton-density 层面的 artifact，详见 [2026-06-06_chr1_qc_controlled_sds_test.md](logs/sds/2026-06-06_chr1_qc_controlled_sds_test.md)。

## 5b. Project-Wide QC Contract

The following rules are now the project-level default for any Chinese-cohort SDS QC work, including `raw/9k/sample_qc/` and downstream freeze generation:

1. Sample QC is order-sensitive. Metadata harmonization and sample intersection come first, then missingness and heterozygosity, then relatedness, then post-relatedness PCA, then singleton/callable-region checks, then the final freeze.
2. Relatedness is capped at second degree or closer. The working cutoff is `KING 0.0884`; looser thresholds need explicit justification.
3. Heterozygosity is lower-tail only by default. Use `mean - 5 SD` for exclusion; do not drop high-heterozygosity samples unless there is an independent contamination or library-failure signal.
4. PCA outliers are judged on the post-KING sample set. PCA on the pre-KING set is diagnostic only, not the final inclusion decision.
5. Singleton metrics are primarily a QC diagnostic and callable-region control. They should not be used as a blunt global filter before the sample/relatedness screen is finished.
6. Variant/SNP QC for SDS must exclude low-confidence sequence contexts, including centromeres, pericentromeric and subtelomeric masks, low-complexity sequence, and segmental duplications.
7. SDS scans should use a common-variant universe that is stable enough for selection inference. Sample-QC statistics can use a stricter common-SNP subset; selection scans should only run on the final callable/common set.
8. Any new QC threshold must be recorded in `SDSlog/SDS_Chinese_Cohort.md` and reflected in the corresponding `sample_qc` report before it is treated as a project standard.

## 5c. Recommended QC Flow for `raw/9k/sample_qc`

This is the preferred flow for the current `raw/9k/sample_qc` directory. It matches the current data products, but it makes the dependency order explicit so the final freeze is built from the post-relatedness sample set.

| Step | Purpose | Recommended rule |
| --- | --- | --- |
| 0 | Sample/metadata harmonization | Intersect VCF samples with metadata, deduplicate sample IDs, and write missing-sample report. |
| 1 | Sample missingness | Exclude samples with `F_MISS >= 0.05`. |
| 2 | Heterozygosity | Compute on LD-pruned common SNPs; exclude only samples below `mean - 5 SD`. |
| 3 | Relatedness | Run KING on the same pruned common SNP set; keep only unrelated samples under the `0.0884` cutoff. |
| 4 | Post-KING variant reduction | Re-run LD prune on the retained samples, then regenerate PCA on that post-KING subset. |
| 5 | PCA review | Flag PCA outliers on the post-KING set; review labels against cohort metadata and ancestry before final removal. |
| 6 | Singleton QC | Count singleton burden and inspect callable-region artifacts only after sample filters are fixed. Treat singleton outliers as diagnostic unless they are independently supported by library/QC evidence. |
| 7 | Final freeze | Write `sample_qc_metrics.tsv`, `cohort_freeze.tsv`, `controls_freeze.txt`, and the summary report from the post-QC sample set. |

The main implementation note for `raw/9k/sample_qc` is that KING should remain on the LD-pruned common SNP panel, but the definitive PCA and freeze must be built from the post-KING sample set. That is the cleanest way to avoid letting relatedness distort ancestry filtering.

## 5a. Current SDS Model Policy

| Role | Demographic model | Gamma shape source | Present-day `Ne(0)` | Status |
| --- | --- | --- | --- | --- |
| Mainline (provisional, under provenance review) | `historical old/default gamma table` | `/data/home/grp-wangyf/xuyuan/sds/g_file.txt` | `implicit / unknown from historical file provenance` | **active but provenance unresolved** |
| Sensitivity / rejected | `phlash q50 scaled` | `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/scaled_ne0_100000/scaled_ne0_100000.minimal_g.tsv` | `100000` | rejected |
| Sensitivity / rejected | `Gravel_CHB` | `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/gravel_chb_present100000.g_file.txt` | `100000` | rejected |

- **2026-06-03 update**: `SDS-MODEL-001` 虽然仍是唯一产出 plausible Manhattan 的轨道，但其 gamma provenance 被识别为未解决的根本问题。olddefault shape 值与 Gravel_CHB@100k 相差 3-27 倍，来源可能是不同 Ne(0)、不同 demography 或不同 gamma generation 方法。在 provenance 明确之前，所有基于 olddefault 的 paper-facing 结论都是 provisional 的。
- `raw-joint`、`input-fix`、`parent-like` 只描述 input lineage，不是新的 demographic model。
- 所有 SDS demography provenance 的标准名字和状态统一见 [logs/sds/README_demographic_models.md](logs/sds/README_demographic_models.md)。

## 6. Immediate Gaps

### Blocking (must resolve before paper-facing conclusions)

- **Gamma provenance**: 追溯 olddefault `g_file.txt` 的确切来源（检查 SDS 原作者的 R 脚本、supplementary、邮件记录）；做 Ne(0) 敏感性扫描确认是 Ne(0) 差异还是 shape 差异；评估 constant Ne baseline。详见 [2026-06-03_pipeline_reflection_bfile_snp_gamma_demography.md](logs/sds/2026-06-03_pipeline_reflection_bfile_snp_gamma_demography.md)。
- **Workflow rebuild order**: 不再把“追加 compute”当作默认下一步；先按 [2026-06-05_first_principles_rebuild_workflow_compare.md](logs/sds/2026-06-05_first_principles_rebuild_workflow_compare.md) 固化 clean-root、freeze、mask、site universe、gamma acceptance 的先后顺序。
- **Repo/workspace adoption**: 新的 reusable 代码优先写到 `SDSpipeline/`；新的 run 优先写到 `SDSworkspace/`，并强制落 provenance manifest。
- **b_file centromere fix**: 修正 b_file 生成逻辑，显式排除 GRCh38 着丝粒 gap 和 pericentromeric heterochromatin。
- **SNP input contract expansion**: 增加 pericentromeric/subtelomeric mask 和 DAF 下限。
- **QC integration into production entrypoint**: 当前 chr1 测试已通过 `SDSpipeline` 的 controlled post-filter 完成，但还需要把 mask、variant QC、singleton-density、excess-sample 和 observability contract 集成进 production input workflow，并补全 genome-wide low-mappability/LCR/segmental-duplication masks。
- **chr1 peak masking**: 将 chr1:125.1M 和 143.2M 从 paper-facing candidate list 中移除。
- **chr6 MHC 验证**: 用 iHS/EHH 对 chr6 MHC 做正交验证，确认 SDS pipeline 对该区域是否有 power。

### Backfill (important but not blocking)

- 需要把 meta analysis 的输入、输出、overlap 结果文件路径补进 `logs/meta_overlap` 和 `logs/result_index.md`。
- 需要把 iHS / EHH 的显著区域表、gene label 输出和关键图补进 `SDSlog`。
- 需要把 Relate + CLUES2 的关键结果文件、图和参数补进 `SDSlog`。
- 需要在最终写文章前核对 genome-wide postprocess summary 中阈值记录的差异。
