# 模拟零分布按 464 条单倍型重建，并用全基因组 PCA 重跑主成分梯度

Date: 2026-11-14
Topic: relate_clues2
Type: test
Question: 之前模拟零分布用的是 928 条单倍型、而真实 PM 臂只有 464 条，两者不可直接比；另外主成分梯度用的是旧的"只用 chr1"的 PC。这两处都要修正，并给真实位点算可信的经验 p 值。

## 输入路径

- 队列基因型：`/data/home/grp-wangyf/xuyuan/raw/9k/snp_singleton_qc_chr19_v1/snp_qc_pass.vcf.gz`
- 全基因组 PC（新）：`/data/home/grp-wangyf/xuyuan/SDSworkspace/population_structure/tests/v1w_sample_pcs.tsv`（由 `_scratch/v1w/pcs.npy` 导出，9.2 万位点、chr19 排除，按 metadata 行序）
- 真实两臂的 CLUES 结果：`.../structure_aware/work/real_cal/`
- 模拟数据（旧 928 hap）：`.../structure_aware/work/sim_cal.*`
- 模拟数据（新 464 hap）：`.../structure_aware/work/sim464.*`

## 输出路径

- 464 条单倍型的模拟 ARG：`.../structure_aware/work/sim464.relate.anc` / `.mut`
- 模拟自己的 coal：`.../structure_aware/work/sim464_calibrated.coal`
- 464 条单倍型的中性零分布：`.../structure_aware/work/null464/`
- 真实位点对零分布的经验 p 值：`.../structure_aware/results/s8_pvalues.tsv`
- 用新 PC 重跑的主成分梯度：`.../population_structure/tests/v2_lead_pc_background.tsv`（脚本 `v2_lead_pc_gradient.py`、`v2b_background_null.py`、`v2c_compare.py`）

## 参数

- 新模拟：4 个群体排成一线、每群 Ne=3000、迁移率 0.08、长度 2 Mb、重组/突变率 1.25e-8、每群 116 个二倍体；**只输出两端两个群体**，共 232 个二倍体 = **464 条单倍型**（与 PM 臂一致）。seed=303，F_ST(两端)=0.00258。
- 模拟自己的 coal：`RelateCoalescentRate --mode CoalRateForTree -m 1.25e-8 --dist sim464.dist -i sim464.relate -o sim464_calibrated`
- 中性位点：频率 3–8%，每个都走 `AncMutForSubregion → SampleBranchLengths（200 棵）→ prepare_clues_inputs → RelateToCLUES_fast → inference.py --df 100 --tCutoff 600 --timeBins 50 200`；**SampleBranchLengths 与 CLUES2 用同一份 `sim464_calibrated.coal`**。
- 经验 p 值：`p = (1 + #{null ≥ lead}) / (1 + n_null)`。
- 新 PC 梯度：7 个位点的携带状态对**新的全基因组 PC1–10** 做逻辑回归；背景为每个位点 400 个频率匹配（±0.004）的中性位点。

## 人口模型

- `Demographic model=sim_stepping_stone_k4_m0.08_out23_464hap`；`Gamma shape source=不适用（CLUES2 用 coal）`；`Present-day Ne(0)=每群 3000（模拟设定）`；`Model role=mainline（中性零分布）`；`Model note=单倍型数已与 PM 臂一致（464）`
- `Demographic model=cohort_empirical_PM_NCN / PM_SCN`；`Present-day Ne(0)=近期约 3.7 万（coal 在 49.6 代处的速率反推）、全时段中位数约 6.3k/7.5k`；`Model role=sensitivity（真实位点）`；`Model note=coal 第三行是逐时段瞬时合并率，不是长期平均；此前把中位数写成 Ne(0) 是错的`
- `Demographic model=Gravel_EAS`；`Model role=rejected（用于模拟数据时）`

## 脚本 / 命令 / 作业号

- `structure_aware/bin/s4_sim_ss.py --out_demes 0,3`（新增"只输出指定群体"选项）
- `structure_aware/bin/s4_run_relate_sim.sh`；LSF 作业 **154887**（smp，已完成）
- `structure_aware/bin/s4_null_site_464.sh`（464 条单倍型中性位点流程）
- `structure_aware/bin/s8_null_pvalue.py`（经验 p 值）
- `tests/v2_lead_pc_gradient.py`、`v2b_background_null.py`、`v2c_compare.py`（改为读取新 PC）

## 结论

1. **单倍型数不匹配已修正**：新模拟输出 464 条单倍型（与 PM 臂的 464 完全一致，site_summary 里 n_haplotypes=464）；旧模拟是 928 条。中性位点用模拟自己的 coal，SampleBranchLengths 与 CLUES2 同用一份。
2. **零分布明显收紧**：464 条单倍型下，截至 39 个中性位点，logLR 中位 1.08、95 分位 2.66、**最大仅 2.92**（928 条单倍型的旧零分布最大到 11.98）。样本量减半后信息量下降，尾部变短，这是预期方向。
3. **经验 p 值（n=39，最小可分辨 0.025）**：
   - PM_NCN：DPP9 9.42（p=0.025）、MIR7-3 9.91（p=0.025）、UHRF1 3.73（p=0.025）、PLIN3 1.89（p=0.20）、DNM2 1.37（p=0.35）、ARRDC5p 62.3（p=0.025，但该位点朝向不一致，排除）、ARRDC5m（朝向不一致，排除）。
   - PM_SCN：DPP9 5.53（p=0.025）、MIR7-3 3.99（p=0.025）、DNM2 2.63（p=0.125）、PLIN3 3.39（p=0.025）、UHRF1 1.88（p=0.20）。
   - **注意**：多个位点都顶到 0.025 的分辨下限，说明零分布尾部还没采够；150 个中性位点仍在跑，n 增加后 p 值会变小、分辨更好。当前只能下"DPP9/MIR7-3 明显高于中性、DNM2/PLIN3 部分高于中性"的定性结论。
4. **新 PC 重跑主成分梯度**：换成全基因组 PC（9.2 万位点、chr19 排除）后，结论与原 chr1-PC 基本一致——DNM2（超出比例 0.023）、MIR7-3（0.039）最靠边，DPP9（0.53）、PLIN3（0.76）、UHRF1（0.96）在背景内。说明原来的 PC 梯度结论不是 chr1 特异的，但表述必须改成"全基因组 PC"。
5. **coal 解释更正**：coal 第三行是逐时段的瞬时合并率；PM_NCN/PM_SCN 的**近期** Ne 约 3.7 万（49.6 代处），全时段中位数才是 6.3k/7.5k。此前把中位数当 Ne(0) 的说法错了，已在本文更正。

## 下一步

- 等 150 个中性位点跑完，重算 p 值（n≈100–150 时最小可分辨约 0.007–0.01），并补上 Holm/Bonferroni 校正。
- 若要进一步严格，可让模拟的近期 Ne 也与队列一致（当前模拟每群 3000，队列近期约 3.7 万），但两者都用各自匹配的 coal，属于"自校准"比较，Ne 差异不直接构成偏差。