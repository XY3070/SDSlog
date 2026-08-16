# Handover Checkpoint — 2026-08-16 (session switch)

Date: 2026-08-16
Status: CHECKPOINT — 完整进度交接，供新 session 直接续跑
Role: mainline — Relate ALL chr19 downstream + SNP master table

## 1. 正在运行 / 挂起的任务

### 1a. Relate followup 链（505504 已结束失败，见 §2）
- **231/231 BuildTopology sections 完成** ✓（chunk_0/UKBQC_ALL_chr19_{0..230}.anc，共 11.9GB）
- **FindEquivalentBranches 完成** ✓（.mut 231）
- **InferBranchLengths 完成** ✓（per section，xargs 12 并行，infer_sec*.log 全含 "CPU Time" 完成标记）
- **CombineSections 完成** ✓ → `UKBQC_ALL_chr19/chunk_0/UKBQC_ALL_chr19_c0.anc`（12.1GB）+ `_c0.mut`（56.8MB，**5 列旧格式**）
- **断点**：CoalRateForTree 报 `Error reading following line in mut file: 0;1960 7591 3701;1;0;0;0;`——当前 `relate/bin/Relate`（4 月编译，**源码版，无 ReEstimateBranchLengths 模式**）与 mut 5 列格式不兼容下游（RelateCoalescentRate/RelateSelection 需 14 列新格式，见 v7 的 `run_work_chr19_v7/UKBQC_NCN_chr19.mut`）
- **根因**：relate 版本太老。`old_versions/relate_v1.2.4_x86_64_static.tgz` 存在（18.4MB）——v7（NCN 成功含 RelateSelection）用的就是 v1.2.4（新格式 mut + ReEstimateBranchLengths 支持）
- **卡住点**：v1.2.4 解压被 auto-mode classifier 拦截（settings.json 已配置 `autoMode.environment` 可信路径——用户已手动写入 `/data/home/grp-wangyf/xuyuan/.claude/settings.json`——**新 session 应不再拦截**）

### 1b. Master SNP table（后台跑，PID 227788）
- 脚本：`SDSworkspace/tmp/build_master_snp_table.py`
- 流程：1KG metadata（78M 位点）→ dbSNP 流式 rsID 匹配 → SDS normalized AA/DA → 输出 `orthogonal/ihs/master_snp_table.tsv`
- 日志：`SDSworkspace/tmp/master_table.log`——上次看卡在 "streaming dbSNP for rsIDs..."（26GB gz 流式 ~30-60min）——**可能已完或被杀，新 session 先查**
- **注意**：另一台服务器已自己做 dbSNP rsID——这张表主要用途 = chr:pos + AA/DA（SDS 覆盖 4.5M common variants，其余位点 AA 空）
- 后续：另一台服务器 polarization 用 EPO .fa（`input/reference/markers/homo_sapiens_ancestor_GRCh38/`，3GB，chr1-22+X 共 23 文件 ~2.9GB）——**决定迁 .fa 到那边**（不是迁结果）

## 2. Relate 下一步（新 session 直接执行）

```bash
# ① 解压 v1.2.4（分类器已配置可信，应通过）
mkdir -p /data/home/grp-wangyf/xuyuan/relate/relate_v1.2.4
cd /data/home/grp-wangyf/xuyuan/relate/relate_v1.2.4
tar xzf ../old_versions/relate_v1.2.4_x86_64_static.tgz   # 解压后找 bin/Relate 等

# ② 用 v1.2.4 跑后续（全部在 chunk_0 内 cd）
W=/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/orthogonal/relate_clues2/ALL/run_work_chr19
C=$W/UKBQC_ALL_chr19/chunk_0
cd $C
export LD_LIBRARY_PATH=/data/home/grp-wangyf/intern/miniforge3/envs/sds/lib:${LD_LIBRARY_PATH:-}
REL=/data/home/grp-wangyf/xuyuan/relate/relate_v1.2.4/bin/Relate   # v1.2.4 路径待确认

# ③ ReEstimateBranchLengths（v1.2.4 支持；先 -N 100000 或等 ALL.coal）
$REL --mode ReEstimateBranchLengths -N 100000 -i UKBQC_ALL_chr19_c0 -o UKBQC_ALL_chr19_c0_reestimated

# ④ EstimatePopulationSize → ALL.coal
/data/home/grp-wangyf/xuyuan/relate/bin/RelateCoalescentRate --mode CoalRateForTree --years_per_gen 28 --dist chunk_0.dist -i UKBQC_ALL_chr19_c0_reestimated -o UKBQC_ALL_chr19_c0_reestimated_coal
# 注：--dist 用 chunk_0.dist（MakeChunks 产物）
/data/home/grp-wangyf/xuyuan/relate/bin/RelateCoalescentRate --mode FinalizePopulationSize --years_per_gen 28 -i UKBQC_ALL_chr19_c0_reestimated_coal -o ALL
# → ALL.coal 在 chunk_0/ 下（Relate 要求输出在 cwd）

# ⑤ ReEstimate（用 ALL.coal 重跑）
$REL --mode ReEstimateBranchLengths --coal ALL.coal -i UKBQC_ALL_chr19_c0 -o UKBQC_ALL_chr19_c0_reestimated

# ⑥ RelateSelection（v1.2.4 的 RelateSelection，读新格式 mut）
$REL_SEL --mode SDS -i UKBQC_ALL_chr19_c0_reestimated -o $W/UKBQC_ALL_chr19_relate_sds
bash /data/home/grp-wangyf/xuyuan/relate/scripts/DetectSelection/DetectSelection.sh -i UKBQC_ALL_chr19_c0_reestimated -o $W/UKBQC_ALL_chr19_selection -m 1.25e-8
```
- 参考实现：`SDSworkspace/tmp/relate_post_combine.sh`（v2 版已写 cwd=chunk_0 逻辑；worker 脚本 relate_ibl_section.sh/relate_reest_section.sh 存在）
- **坑**（已踩过，勿重复）：非交互 bash `jobs -r` 空 → 并发失效 OOM（用 xargs -P）；xargs `-I {} bash -c` 单引号问题（用独立 worker 脚本）；CombineSections 需大内存（-M 64000，-M 4000 会 OOM kill）；Relate 输出必须 cwd；chunk 结构前缀 `UKBQC_ALL_chr19_c0`（非 UKBQC_ALL_chr19）；v1.2.4 之前的所有 Relate binary 无 ReEstimateBranchLengths

## 3. 已完成事项（本 checkpoint 前）

### 3a. final TSV：`signals_clump/loci_p5e5_final.tsv`（112 行，chr3 合并版在 tmp 待确认）
- **114 行完整版 staging**：`SDSworkspace/tmp/loci_p5e5_final_with_chr3.tsv`（含 chr3 2 loci：50369825 norm_iHS -0.70/RASSF1 eQTL 1e-128、132013301 norm_iHS -1.25）——**用户确认后** cp 覆盖 final（当前 final 是 112 行无 chr3）
- 40 列，Excel 列序（lead_snp → top_TFs），1KCP top3 `gene(p)` 格式，motif Δ→d ASCII，iHS 统一**负值=derived 被选**约定（用户定义的 Voight 惯例——所有来源已翻转一致：norm_iHS/NyuWa_iHS/iHS_EAS/iHS_TOPMed ×-1；iHS_CHB/CHS/JPT 本就 Johnson 惯例；**NOTE 行已删**）
- 方向验证：ADH1B/ALDH2（derived 被选）6 来源全负、chr7:50266267（ancestral 被选）全正 ✓

### 3b. iHS Manhattan（CMplot）
- `plots/ihs_manhattan/`：6 张单图 `{ALL,AFR,EUR,EAS,SAS,AMR}_ihs.png`（CMplot 4.5.1，sds env Rscript，-log10P，16×4in@300dpi）+ 合并 `ihs_manhattan_CMplot_ALL_plus_1KG_5pop.png`（PIL 垂直堆叠 4800×7200）
- 数据：1KG 5 superpop iHS 按 EPO .fa polarization（`polarize_norm_1kg_epo.py`）+ 100-bin norm（SDS DAF，负=derived）；ALL norm 文件已翻转
- 脚本：`SDSpipeline/scripts/orthogonal/polarize_norm_1kg_epo.py`、`plot_ihs_manhattan.py`（已注册 canonical_scripts.tsv，commit 6eb1a2f）

### 3c. EHH plot（114 个 loci 全量重画）
- `plots/ehh_loci_all/`（111 loci）+ `plots/SLE_family/`（3 SLE 点）
- 图例直接标碱基（`G (derived, DA)`）、标题 AA/DA/DAF、**左下角 metrics box 已删**（用户要求）
- 脚本：`SDSpipeline/scripts/orthogonal/plot_ehh_single.py`（commit 722d483 + a0c6cd2）

### 3d. chr7:50266267 (rs4917014) EHH 半衰期（ALL）
- **T (ancestral): left 14.08 / right 16.01 / mean 15.05kb；G (derived): 7.51 / 1.85 / 4.68kb；G/T = 0.31**
- 与用户 1KG 表 EAS（T 13.52 / G 5.59 / 0.41）一致——EAS/ALL 中 ancestral T 特异受选择（SDS -2.71、iHS 全正一致）

### 3e. 其它
- Relate NCN v7 参考输出：`run_work_chr19_v7/`（NCN_chr19_relate_sds.SDS、NCN_chr19_selection.sele）
- SDSlog 已有条目：2026-08-10_p5e5_catalog_annotation.md（round 2 含 SLE 点/方向修复）、2026-08-09_ihs_polarization.md、2026-08-09_phasing_complete_relate_resume.md

## 4. 关键路径速查

| 用途 | 路径 |
|---|---|
| Relate ALL workdir | `SDSworkspace/results/production/orthogonal/relate_clues2/ALL/run_work_chr19/` |
| chunk 数据 | `.../chunk_0/UKBQC_ALL_chr19_c0.{anc,mut}`（12.1GB/56.8MB） |
| relate v1.2.4 | `relate/relate_v1.2.4/`（待解压）+ `relate/old_versions/relate_v1.2.4_x86_64_static.tgz` |
| followup 脚本 | `SDSworkspace/tmp/relate_post_combine.sh`、`relate_followup.sh`、`relate_ibl_section.sh`、`relate_reest_section.sh` |
| final TSV | `signals_clump/loci_p5e5_final.tsv`（112）+ `tmp/loci_p5e5_final_with_chr3.tsv`（114 staging） |
| iHS Manhattan | `plots/ihs_manhattan/`（CMplot 6+1） |
| EHH plots | `plots/ehh_loci_all/`、`plots/SLE_family/` |
| master SNP table | `tmp/build_master_snp_table.py` → `orthogonal/ihs/master_snp_table.tsv` |
| EPO fasta | `input/reference/markers/homo_sapiens_ancestor_GRCh38/`（迁另一台用） |
| 1KG iHS 5 pop | `orthogonal/ihs/{AFR,EUR,EAS,SAS,AMR}/chr*.ihs.out`（原始）+ `.100bins.norm`（EPO 极化后） |
| settings.json | `/data/home/grp-wangyf/xuyuan/.claude/settings.json`（autoMode.environment 已配） |

## 5. 待办清单（新 session 顺序）

1. **查 master table**（tmp/master_table.log + master_snp_table.tsv 是否生成）
2. **解压 relate_v1.2.4** → 按 §2 跑 Relate 链（ReEstimate → CoalRateForTree → FinalizePopulationSize → RelateSelection SDS+Selection）→ 验证 .SDS/.sele/ALL.coal
3. **chr3 合并确认**：用户确认后 `cp tmp/loci_p5e5_final_with_chr3.tsv signals_clump/loci_p5e5_final.tsv`（114 行）
4. RelateSelection 输出 → mirror Manhattan（SDS vs Relate SDS，参考 v7 或 mirrored_manhattan.py）
5. 另一台服务器 polarization：EPO .fa 迁移清单（chr1-22+X 23 文件）
