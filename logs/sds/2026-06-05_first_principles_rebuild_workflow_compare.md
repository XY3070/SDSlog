# 从第一性原理重建 SDS workflow，并与当前 pipeline 对比

Date: 2026-06-05
Topic: sds
Type: summary
Question: 如果今天不继承历史口径、只继承原始数据和项目目标，SDS 课题应该如何从头搭建 end-to-end workflow？现有 pipeline 哪些资产可以保留，哪些环节必须重写？

## Input paths

- `/data/home/grp-wangyf/xuyuan/SDSlog/SDS_log.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/SDS_Chinese_Cohort.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/index.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/2026-05-03_ncn_gamma_track_compare_review.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/2026-05-09_raw_joint_pipeline_prep_and_chr10_smoke.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/2026-05-31_scn_olddefault_genomewide_repair_conclusion.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/2026-05-31_scn_chr1_qarm_boundary_skip_root_cause.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/2026-06-03_pipeline_reflection_bfile_snp_gamma_demography.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/README_demographic_models.md`
- `/data/home/grp-wangyf/xuyuan/sds/SDS_SCRIPT_PLAN.md`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/common_env.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/generate_b_file.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/generate_s_file.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/generate_t_file.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/generate_o_file.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/run_sds_input.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/run_raw_joint_chr_inputs.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/run_sds_compute.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/run_sds_compute_chunk.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/submit_sds_compute_chunked_chr.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/postprocess_sds_results.py`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/submit_gravel_chb_gamma_parallel.py`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/run_smcpp_recent_resolution_workflow.sh`

## Output paths

- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/2026-06-05_first_principles_rebuild_workflow_compare.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/index.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/SDS_log.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/SDS_Chinese_Cohort.md`

## Parameters

- `scope=workflow design and pipeline comparison; no new compute jobs launched`
- `project_goal=Chinese-specific recent positive selection scan with SDS as primary statistic, then tSDS/GWAS, iHS/EHH, Relate/CLUES2, meta overlap, archaic introgression for follow-up`
- `current_freeze=NCN 1255 + SCN 2716 = 3971 WGS samples`
- `comparison_baselines=SDS-MODEL-001, SDS-MODEL-002, SDS-MODEL-003 plus raw-joint / rebuilt-input / boundary-repair branches`
- `proposed_clean_root=/data/home/grp-wangyf/xuyuan/sds_rebuild`

## Demographic model

- `Demographic model=workflow design only; current comparison references SDS-MODEL-001/002/003`
- `Gamma shape source=planning only; rebuild workflow requires reproducible gamma generation before production compute`
- `Present-day Ne(0)=not fixed in advance; must be inferred, validated, and versioned inside the rebuild pipeline`
- `Model role=audit-only`
- `Model note=this record defines a rebuild blueprint, not a new production gamma track`

## Scripts / commands / job IDs

- inspected scripts:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/run_sds_input.sh`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/generate_s_file.sh`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/generate_t_file.sh`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/generate_o_file.sh`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/generate_b_file.sh`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/run_raw_joint_chr_inputs.sh`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/run_sds_compute.sh`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/run_sds_compute_chunk.sh`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/submit_sds_compute_chunked_chr.sh`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/postprocess_sds_results.py`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/submit_gravel_chb_gamma_parallel.py`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/run_smcpp_recent_resolution_workflow.sh`
- inspected method logs:
  - `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/2026-05-03_ncn_gamma_track_compare_review.md`
  - `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/2026-05-31_scn_olddefault_genomewide_repair_conclusion.md`
  - `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/2026-05-31_scn_chr1_qarm_boundary_skip_root_cause.md`
  - `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/2026-06-03_pipeline_reflection_bfile_snp_gamma_demography.md`
- `job_ids=None; planning-only record`

## Key plots

- `None archived`

## Key result files

- `None; planning-only record`

## 1. Project purpose restated from the current record

这个项目的真正目标不是“跑出一张好看的 Manhattan”，而是建立一条能自洽回答下面因果链的 pipeline：

1. 在中国人群里定义可信的 recent-selection scan universe。
2. 用可复现、可校准的 SDS null model 找出 recent positive selection candidates。
3. 用正交证据判断这些 peak 是真实生物学、结构复杂区、还是 input / demography / normalization artifact。
4. 进一步回答 polygenic selection、meta overlap、archaic introgression enrichment 这些 paper-facing follow-up。

2026-06-03 的反思说明，当前瓶颈不在“代码能不能跑完”，而在 inference chain 中间三个环节断掉了：

1. `b_file` / region contract 没有先把明显不该分析的区域排掉。
2. SNP universe 对 SDS 过于宽松，导致能过 VQSR 的 bad sites 仍然进入统计量。
3. gamma / demography 主线依赖 provenance 不明的 `historical old/default gamma table`，导致最关键的 null model 不能自证。

所以，从第一性原理重建时，核心不是继续 patch 某个坏 Manhattan，而是把整条因果链重写成“每一层都能单独过关”的 workflow。

## 2. Rebuild principles

重建版 workflow 应遵守 8 条硬原则：

1. **先定义 inference contract，再跑 compute。**
   - 先写清 primary question、候选人群、正负对照、接受/拒绝标准。
   - 不允许在看到 Manhattan 之后倒推阈值和模型。
2. **先定义 analysis genome，再定义 SNP universe。**
   - centromere/pericentromere/subtelomere/assembly gap/segmental duplication/low-mappability 区域必须先变成 mask。
   - 不允许先跑 SDS、后靠 biological common sense 去删峰。
3. **demography 和 gamma 必须能从原始输入独立重建。**
   - 任何不能重新生成的 `g_file.txt` 都不能作为 production null model。
4. **同一条 site universe 驱动所有下游。**
   - SDS、iHS/EHH、Relate/CLUES2、candidate annotation 看到的是同一版 cohort freeze 和 callable genome。
5. **boundary behavior 必须变成前置 QC，而不是后置 heuristic。**
   - 不能等 chr1/chr9/chr16 早 q-arm 丢光了，才用 `relaxed035`、`cap_to_boundary` 去救。
6. **normalization 必须带稳定性约束。**
   - 不能让极小 bin SD 把 recovered-only 窗口放大成 `27-28` 的 `norm_SDS`。
7. **production acceptance 依赖预定义 controls，不依赖“图看起来顺眼”。**
   - Manhattan appearance 只能作为辅助，不是单独的 accept criterion。
8. **所有 release 都必须有 single source of truth。**
   - 一个 clean root、一个 manifest、一个 model registry、一个 release report。

## 3. Proposed clean-root layout

如果从头重建，我不会继续在 `sds/data/processed/sds_output_*` 这种多分叉目录里叠加结果；我会单独开一棵 clean root：

```text
/data/home/grp-wangyf/xuyuan/sds_rebuild/
  config/
    project_scope.yaml
    release_policy.md
    positive_controls.tsv
    negative_controls.tsv
  manifests/
    cohort_freeze.tsv
    sample_qc_metrics.tsv
    ancestry_split.tsv
    site_universe_registry.tsv
    release_registry.tsv
  masks/
    grch38_gap_centromere.bed
    pericentromeric_blacklist.bed
    subtelomeric_blacklist.bed
    segdup_lowmap_blacklist.bed
    callable_autosome.bed
    chr_arm_manifest.tsv
  demography/
    NCN/
    SCN/
    model_compare.tsv
  gamma/
    NCN/
    SCN/
    gamma_candidate_registry.tsv
  input/
    NCN/chr1/
    SCN/chr1/
  compute/
    NCN/<release_tag>/
    SCN/<release_tag>/
  postprocess/
    NCN/<release_tag>/
    SCN/<release_tag>/
  validation/
    NCN/<release_tag>/
    SCN/<release_tag>/
  orthogonal/
    ihs_ehh/
    relate_clues2/
    meta_overlap/
    archaic_introgression/
  reports/
    <release_tag>_acceptance_report.md
```

这里的关键不是目录美观，而是把“人群 freeze”、“analysis mask”、“demography/gamma candidate”、“compute release”彻底解耦，避免后续再次把 input lineage、gamma lineage、boundary repair lineage 混在一起。

## 4. Rebuild pipeline: stage by stage

### Stage 0. Project contract and acceptance criteria

目的：先固定“什么算成功”。

必须输出：

- `config/project_scope.yaml`
- `config/positive_controls.tsv`
- `config/negative_controls.tsv`
- `config/release_policy.md`

必须写清的内容：

- primary scan 是 `NCN`、`SCN` 各自独立跑，还是只把某一个作为主文图。
- paper-facing statistic 用 `norm_SDS` 还是 raw `rSDS`，以及 raw `rSDS` 只承担什么诊断角色。
- 正对照 panel 至少包含：
  - `chr6 MHC`
  - 当前项目已经持续跟踪的 `chr4` locus
- 负对照 panel 至少包含：
  - centromere/pericentromere/subtelomere masks
  - gene desert / high-repeat diagnostics
  - downsample / permutation null checks

stop/go gate：

- 没有 acceptance criteria，不进入任何 genome-wide compute。

### Stage 1. Cohort freeze and sample QC

目的：把“3971 样本”变成可以审计的 freeze，而不是口头事实。

必须输入：

- raw or frozen WGS callset
- sample metadata
- ancestry coordinates
- contamination / missingness / coverage / relatedness metrics

必须输出：

- `manifests/sample_qc_metrics.tsv`
- `manifests/cohort_freeze.tsv`
- `manifests/ancestry_split.tsv`
- `reports/cohort_freeze_report.md`

具体要求：

- 每个样本必须有：
  - inclusion/exclusion 状态
  - exclusion reason
  - final population label (`NCN` / `SCN` / excluded)
- ancestry split 必须存成表，而不是只留一版 `NCN.txt` / `SCN.txt`
- 所有后续 SDS / iHS / Relate / meta 都只能引用 freeze ID，不直接手工引用某个旧 sample list

stop/go gate：

- `NCN` / `SCN` 样本数冻结且可追溯。
- `cohort_qc` 进入 `SDSlog`，不再留空。

### Stage 2. Analysis genome and region mask

目的：先定义“哪些 genome 区域允许进入 SDS”。

必须输出：

- `masks/grch38_gap_centromere.bed`
- `masks/pericentromeric_blacklist.bed`
- `masks/subtelomeric_blacklist.bed`
- `masks/segdup_lowmap_blacklist.bed`
- `masks/callable_autosome.bed`
- `masks/chr_arm_manifest.tsv`

设计原则：

- arm boundary 不是单纯的“切成 p/q 两块”，而是“在 analysis BED 上给 compute engine 提供两段连续区间”。
- centromere 不能只靠一组 hard-coded `start/end`；必须基于一个明确版本的 GRCh38 reference annotation。
- pericentromeric heterochromatin 不能等 peak 出来后才删，必须事先进 blacklist。
- subtelomere、assembly gap、segmental duplication、低 mappability 区域也必须显式建模。

推荐实现：

1. 先合并全部 blacklist interval。
2. 再与 autosome callable BED 做差集。
3. 在每条染色体上把剩余 callable 区间分配到 `p` / `q` arm。
4. `b_file` 不再独立决定“能算什么”，而只是 `analysis BED` 的 arm-level summary。

stop/go gate：

- chr1 / chr6 的 q-arm 起点必须在最终允许分析区域之外经过 mask 审核，而不是沿用历史 `124932725`、`59829935` 这种会把着丝粒混进来的数字。
- 所有 `analysis BED` 区间都能回查到来源 mask。

### Stage 3. Site universe and variant contract

目的：在 analysis genome 上定义真正可供 SDS 观察的 site universe。

必须输出：

- `manifests/site_universe_registry.tsv`
- `input/<POP>/site_universe.chrN.tsv.gz`
- `input/<POP>/singleton_universe.chrN.tsv.gz`
- `reports/site_universe_qc.<POP>.md`

我会把 site universe 拆成两层：

1. **test SNP universe**
   - 允许进入 `t_file` 的 SNP 集
2. **singleton universe**
   - 允许进入 `s_file` 的 singleton 集

两层都必须共享的硬过滤：

- analysis BED 内
- PASS
- biallelic SNP
- sample freeze 一致
- per-site callable / missingness 达标
- 必要时加入 depth / GQ / allele balance 的 WGS 质量注释

singleton universe 额外要求：

- `AC=1`
- 缺失样本数和缺失比例双重限制
- 每个 singleton site 记录 upstream/downstream callable context，不只输出位置

test SNP universe 额外要求：

- 明确记录 `DAF`、`F_MISSING`、callability 注释
- 在进入 compute 之前先定义最低可接受频率区间
- 如果决定不对 ultra-rare variant 跑 paper-facing SDS，就在这里把它们从 production universe 中分流出去

stop/go gate：

- `t_file` 和 `s_file` 不得来自两套不一致的 region / missingness contract。
- 所有被 mask 掉的区域，在 `site_universe` 里必须是零。

### Stage 4. Demography estimation

目的：用可重建的 demography 驱动 null model，而不是反过来让 Manhattan 决定 demography。

必须输出：

- `demography/<POP>/model_inputs_manifest.tsv`
- `demography/<POP>/smcpp/`
- `demography/<POP>/phlash/`
- `demography/<POP>/relate/`
- `demography/model_compare.tsv`
- `reports/demography_acceptance.<POP>.md`

设计原则：

- `NCN`、`SCN` 各自单独估 recent demography。
- `SMC++` 作为主要可解释主线，`phlash` / `Relate` 作为 sensitivity / cross-check。
- demography 不能只比较“曲线像不像”，要比较对数据 summary 的解释力：
  - SFS
  - heterozygosity
  - singleton count distribution
  - recent branch-length proxies

最重要的决策规则：

- 如果 candidate demography 生成的 gamma 不能通过后续 null calibration，就不能进入 production compute。
- 如果没有任何 candidate demography 能通过，应该停在 demography/null 这一层，而不是继续用 provenance 不明的历史 `g_file.txt`。

### Stage 5. Gamma generation and null calibration

目的：把 demography 变成可复现的 `g_file.txt`，并证明它真的适用于当前 data contract。

必须输出：

- `gamma/<POP>/gamma_candidate_registry.tsv`
- `gamma/<POP>/<model_id>/g_file.txt`
- `gamma/<POP>/<model_id>/piece_manifest.tsv`
- `gamma/<POP>/<model_id>/simulation_summary.tsv`
- `reports/gamma_acceptance.<POP>.<model_id>.md`

gamma 层必须回答 4 个问题：

1. 这份 `g_file.txt` 能否被脚本从头重建？
2. 相同参数重复模拟时，shape 是否稳定？
3. 这份 gamma 在 chr-level smoke / null diagnostics 上会不会产生 diffuse tail？
4. 它和 empirical frequency-bin behavior 的偏差是否在可接受范围内？

我会把 acceptance criteria 写死：

- `g_file.txt` 必须来自 versioned demography input 和 versioned gamma script。
- 不能以“图更像 biology”为理由选用不可复现的 gamma。
- production candidate 至少要过：
  - small-chrom smoke
  - two-pop compare sanity
  - empirical common-bin diagnostics
  - permutation or masked negative-control checks

如果一条 gamma 只能通过“看起来比别的好”，但来源不可复现，那它只能留作 diagnostic reference，不能是 paper baseline。

### Stage 6. SDS input materialization

目的：把前 5 个 stage 的 contract 落成 `b/s/t/o` 四类 production input。

必须输出：

- `input/<POP>/chrN/chrN_b_file.txt`
- `input/<POP>/chrN/chrN_s_file.txt`
- `input/<POP>/chrN/chrN_t_file.txt`
- `input/<POP>/chrN/chrN_o_file.txt`
- `input/<POP>/chrN/chrN_manifest.json`
- `input/<POP>/chrN/chrN_audit.tsv`

重建版里，四类输入都必须带 manifest：

- sample freeze ID
- site universe ID
- mask release ID
- demography candidate ID
- gamma candidate ID
- software version / script hash

特别注意两点：

1. `o_file` 不能默认永远全 1。
   - 只有在 site universe 已经把 site-level callability 收紧到足够严格时，`all ones` 才有资格保留。
2. `s_file` 不能只存“位置列表”。
   - 至少还要能回查每个 singleton 是否靠近 mask 边界、是否受 missingness 影响。

stop/go gate：

- 任一染色体任一 arm 的 `s_file` 若整体接近 `NA` 或 boundary 附近覆盖显著缺失，直接 fail。
- 不能等到 compute 后才发现整段 q-arm 被吃掉。

### Stage 7. Compute

目的：在固定好的 input + gamma contract 上计算 raw `rSDS`。

必须输出：

- `compute/<POP>/<release_tag>/chrN_p.sds.tsv`
- `compute/<POP>/<release_tag>/chrN_q.sds.tsv`
- `compute/<POP>/<release_tag>/chrN.sds.tsv`
- `compute/<POP>/<release_tag>/logs/`
- `compute/<POP>/<release_tag>/chunk_metrics.tsv`

compute 层我会保留当前工程架构里的两个优点：

- top-level 先按 `p/q` 分 arm
- arm 内再按 row chunk 并行

但 boundary handling 规则要改：

- `upstream_nan_fraction` / `downstream_nan_fraction` 必须输出为 per-SNP diagnostics
- boundary censoring threshold 必须在 release policy 中预先定义
- 若某一整段 q-arm 早端系统性掉光，视为 input-layer failure，不允许用 post hoc relaxed policy 直接升格成新 baseline

我会把 compute 发布流程拆成三层：

1. `1-chr smoke`
2. `multi-chr pilot`
3. `genome-wide release`

推荐 pilot 面板：

- `chr1`：当前已知最容易暴露 pericentromere / boundary 问题
- `chr4`：当前项目已有持续 follow-up 的 locus
- `chr6`：当前已知 MHC 缺失问题
- `chr10` 或 `chr22`：中性 smoke 染色体

只有 pilot 四条都过，才进入全基因组 compute。

### Stage 8. Postprocess and normalization

目的：把 raw `rSDS` 变成可比较、可解释、不会被 tiny-bin artifact 放大的 summary。

必须输出：

- `postprocess/<POP>/<release_tag>/<POP>.normalized.tsv`
- `postprocess/<POP>/<release_tag>/<POP>.frequency_bins.tsv`
- `postprocess/<POP>/<release_tag>/<POP>.significant_regions.tsv`
- `postprocess/<POP>/<release_tag>/<POP>.plot_points.tsv`
- `reports/postprocess_acceptance.<POP>.<release_tag>.md`

重建版 normalization 规则：

1. 先分 production bin 和 diagnostic bin。
   - 不是所有 frequency bin 都适合进入 paper-facing `norm_SDS`。
2. 每个 bin 必须有最小样本量要求。
3. 每个 bin 的 mean / SD 必须做稳定性检查。
   - bootstrap
   - leave-one-chromosome-out
   - SD floor / shrinkage
4. recovered-only windows 不能只因为 bin SD 极小就变成超高 `norm_SDS`。

我会把 postprocess accept criteria 写成明确 fail condition：

- 若任一 production bin 的 SD 不稳定或过小，整版 normalized release 退回，不许继续解释 peak。

### Stage 9. Calibration and release gating

目的：在 biological interpretation 之前，先证明这条 scan 至少没明显坏掉。

必须输出：

- `validation/<POP>/<release_tag>/tail_counts.tsv`
- `validation/<POP>/<release_tag>/qq_pp_report.md`
- `validation/<POP>/<release_tag>/masked_region_enrichment.tsv`
- `validation/<POP>/<release_tag>/downsample_stability.tsv`
- `validation/<POP>/<release_tag>/positive_control_panel.tsv`
- `reports/<release_tag>_acceptance_report.md`

必须过的 gate：

1. masked regions 里不能富集“显著 peak”
2. 全基因组不能出现 diffuse tail
3. 频率 bin 不出现系统性 mean drift / SD blow-up
4. downsample 或 split rerun 不得完全颠覆 top peaks
5. 正对照 panel 不能整体失效

这里的重点是把“图像感觉”替换成“release gate”。

### Stage 10. Orthogonal validation and locus triage

目的：不让 SDS 单独承担所有生物学解释。

必须输出：

- `orthogonal/ihs_ehh/<release_tag>/`
- `orthogonal/relate_clues2/<release_tag>/`
- `orthogonal/meta_overlap/<release_tag>/`
- `orthogonal/archaic_introgression/<release_tag>/`
- `reports/locus_dossier/<locus_id>.md`

每个 paper-facing locus 至少要有：

- SDS summary
- region/mask annotation
- gene/nearest-gene annotation
- iHS / EHH status
- Relate / CLUES2 trajectory status
- known repeat / segmental duplication / structural complexity annotation
- population specificity check (`NCN` vs `SCN`)

如果一个 peak 只能在 SDS 上存在，且恰好落在结构复杂区或无基因 heterochromatin，它就应被自动降级为 artifact-suspect，而不是继续占用解释资源。

### Stage 11. Polygenic follow-up and reporting

目的：把 single-locus scan 接到项目总目标。

必须输出：

- `orthogonal/meta_overlap/<release_tag>/tsds_gwas_overlap.tsv`
- `orthogonal/meta_overlap/<release_tag>/meta_summary.md`
- `reports/teacher_summary/<release_tag>.md`
- `reports/paper_candidate_catalog/<release_tag>.tsv`

执行顺序：

1. 先锁定通过 release gate 的 SDS baseline
2. 再做 tSDS / GWAS overlap
3. 再做 meta analysis / archaic introgression enrichment
4. 最后整理 candidate catalog

如果 baseline 还没过 gate，就不应继续在 polygenic 或 meta 层堆新结果。

### Stage 12. Provenance and SDSlog integration

目的：让每条结论都能回溯。

必须输出：

- `manifests/release_registry.tsv`
- `reports/release_changelog.md`
- `SDSlog` 对应 detailed log
- `SDSlog/logs/result_index.md` 更新
- `SDSlog/plots/plot_index.md` 更新

最重要的规则：

- 每一个 production release 都只能对应一个 release tag。
- 每一个 release tag 都必须能定位：
  - cohort freeze
  - mask release
  - site universe
  - demography candidate
  - gamma candidate
  - compute config
  - postprocess config

## 5. Critical path if I restart today

如果今天重新开工，我不会先跑全基因组。我会按下面顺序走：

1. 补齐 `cohort_qc` 和 freeze manifest。
2. 重写 region mask / callable genome 层。
3. 用新 mask 重新定义 `site_universe`。
4. 单独解决 demography/gamma provenance，直到能生成一个可复现 production candidate。
5. 只跑 `chr1 + chr4 + chr6 + chr10/22` 四染色体 pilot。
6. pilot 通过后再发 genome-wide compute。
7. genome-wide 通过 release gate 后再启动 iHS/EHH、Relate/CLUES2、meta、introgression。

当前最危险的路径是反过来：

- 先沿用旧 gamma 或先做 boundary repair；
- 看到图不像，再回头 patch input/mask；
- 最后用 orthogonal module 给不稳的 baseline 做解释。

这条路已经被 2026-05 到 2026-06 的记录证明会不断积累技术债。

## 6. Comparison with the current pipeline

### 6.1 What the current pipeline already does well

以下工程资产值得保留：

1. `SDSlog` 的 detailed log / result index / plot index 结构已经非常有价值。
2. `run_sds_input.sh` 已支持 `--vcf`、`--sample-list`、`--out-pop`，且能检查 sample list 是否在 VCF header 里。
3. `run_raw_joint_chr_inputs.sh` 已经证明 input lineage 可以回到 raw-joint，而不是被 split-then-merge 绑死。
4. `run_sds_compute.sh` 和 `submit_sds_compute_chunked_chr.sh` 已经具备 arm-level + row-level fan-out 的工程骨架。
5. `submit_gravel_chb_gamma_parallel.py` 已经把 gamma chunking / aggregate / finalize 的大任务拆开，避免串行卡死。
6. `postprocess_sds_results.py` 已经能稳定产出 normalized table、frequency bins、significant regions 和 Manhattan points。
7. audit sidecar 的想法是对的，说明当前工程已经开始把 input provenance 外显化。

这些都应该进 rebuild 版，但只能作为工程骨架，不能自动代表方法学主线是对的。

### 6.2 What the current pipeline gets wrong at the method level

| Layer | Current pipeline | First-principles rebuild | Why this matters |
| --- | --- | --- | --- |
| Project contract | 目标在 `SDS_Chinese_Cohort.md` 里写明，但 release gate 主要靠 compare 图、tail count、经验判断 | 先写 positive/negative controls 与 accept/reject policy | 避免“看到图再定标准” |
| Cohort freeze | `NCN.txt` / `SCN.txt` 可用，但 `cohort_qc` 仍未系统回填 | 先形成可审计的 freeze manifest | 否则 sample universe 仍是隐式的 |
| Region contract | `generate_b_file.sh` 仍是 hard-coded arm boundary；当前坐标与 2026-06-03 反思中的 centromere 问题一致 | 用 authoritative gap/mask 先生成 analysis BED，再导出 arm summary | 现在是把不该分析的区域先放进来，再事后解释 |
| `t_file` contract | `generate_t_file.sh` 只做 `PASS + biallelic SNP`，没有共享 `F_MISSING` / mask / frequency contract | `t_file` 与 `s_file` 必须共用同一 site universe | 当前 test SNP 层比 singleton 层更宽松，容易混入坏位点 |
| `s_file` contract | `generate_s_file.sh` 只对 `AC=1` 和 missingness 做限制 | singleton universe 还要带 region/context annotations | 现在只能知道 singleton 在哪，不能系统解释为什么缺失 |
| `o_file` | `generate_o_file.sh` 直接输出全 1 | 要么严格证明 site universe 已足够 callable，要么输出真实 observability | 否则 observability 被隐式假定为完美 |
| Input lineage | `sds/data/processed/` 下已有大量 `olddefault_newinput`、`raw_joint`、`repair` 分叉 | clean root 下只允许一个 release lineage | 当前结果根很多，容易混淆“哪条才是主线” |
| Demography / gamma | production 仍靠 `SDS-MODEL-001` 历史 gamma；`SDS-MODEL-002/003` 因 diffuse tail 被 reject | 先重建 demography，再用可复现 gamma 过 null calibration | 当前最关键的 null model 无法自证 |
| Compute boundary policy | `skip_boundary_missing_fraction` 在 repair 中被调成 `relaxed035`、`cap_to_boundary`、`hybrid-policy` | boundary coverage 先做 QC，阈值预定义；若大片 q-arm 丢失就退回 input 层 | 现在 boundary heuristic 已经进入结果层决策 |
| Normalization | 2026-05-31 记录显示 tiny-bin SD 可把 recovered-only 窗口放大到 `27-28` | 加入 min-bin-size、bootstrap、LOCO、SD floor/shrinkage | 否则 normalization 本身能造 peak |
| Validation | MHC 缺失、pericentromeric artifact 是在结果出来后才识别 | 这些应该是 release gate 的组成部分 | 当前 pipeline 假阳性和假阴性问题发现得太晚 |
| Orthogonal modules | `ihs_ehh`、`meta_overlap`、`relate_clues2` 目前还只是 README stub 或零散结果 | 作为 baseline 通过后立刻接入 | 现在这些模块更像后补，而不是 baseline validator |
| Provenance | `SDSlog` 强，但 result roots 多、cohort_qc 缺失、model acceptance 没有单一 release report | 每次 release 出一份完整 acceptance report | 当前 provenance 更像“很多真相碎片”，不是单一交付物 |

### 6.3 Current pipeline state in one sentence

当前 pipeline 的真实状态不是“完全失败”，而是：

**工程编排已经很强，方法学主线却仍然被历史 gamma、隐式 mask、宽松 site universe 和后置 repair policy 绑住。**

## 7. Keep / rewrite / stop-use-now

### Keep

1. `SDSlog` 结构与当前 detailed logging discipline。
2. chunked gamma submitter 的 fan-out 设计。
3. `run_sds_input.sh` 的 sample-vs-VCF validation。
4. raw-joint input builder。
5. arm-level + row-level compute fan-out。
6. postprocess 脚本骨架。

### Rewrite

1. region mask and `b_file` generation
2. shared site-universe builder
3. `t_file` / `s_file` contract
4. `o_file` observability logic
5. demography acceptance and gamma selection
6. normalization stability policy
7. release-gating framework

### Stop using as paper-facing baseline right now

1. 任何仅靠 `historical old/default gamma table`、但没有 provenance closure 的新 production compute
2. 任何未经过 region mask 修订的 pericentromeric peak interpretation
3. 任何仅通过 `relaxed035` / `hybrid-policy` 修复、但未先解决 input mask 和 gamma 问题的“最终版”结论

## Conclusion

- 从第一性原理重建时，主线应该是：`cohort freeze -> analysis genome/mask -> site universe -> demography -> reproducible gamma -> pilot compute -> normalization gate -> orthogonal validation -> reporting`。当前项目卡住的根因，是这条因果链里最关键的 `analysis genome` 和 `gamma/null model` 两层都没有先固定。
- 现有 pipeline 的工程资产已经足够支撑重建，不需要全部推倒；真正要推倒重来的是方法学 contract：region mask、site universe、demography/gamma acceptance、boundary policy 和 release gate。
- 在这份 blueprint 下，下一轮工作不该再从“补一个 compute run”开始，而应从 `cohort_qc`、`mask`、`site universe`、`gamma provenance` 四件事开始。只有它们先过关，后面的 genome-wide SDS、iHS/EHH、meta 和 paper-facing candidate 解释才有意义。

## Next step

1. 用 clean-root 思路先补齐 `cohort_qc` 与 `analysis genome mask` 两层的 manifest。
2. 把 `generate_b_file.sh` 升级成基于 authoritative BED 的 mask-aware arm builder。
3. 把 `generate_t_file.sh` / `generate_s_file.sh` 改成共享同一 site universe。
4. 在不再依赖历史 `g_file.txt` 的前提下，重新做 demography/gamma candidate acceptance。
