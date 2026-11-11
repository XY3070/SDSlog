# 模拟中性数据用错了人口模型、平台标签收紧、真实队列重算

Date: 2026-11-10
Topic: relate_clues2
Type: test + production
Question: 之前"模拟中性数据"的零分布用错了人口模型（coal），结论要重做；同时把平台标签收紧到"确定"的样本，并检查真实数据和模拟数据用的 coal 是否一致。

## 输入路径

- 模拟数据：`/data/home/grp-wangyf/xuyuan/SDSworkspace/population_structure/structure_aware/work/sim_cal.relate.anc` / `.mut`（msprime 模拟 4 个群体、2 Mb）
- 模拟脚本：`/data/home/grp-wangyf/xuyuan/SDSworkspace/population_structure/structure_aware/bin/s4_sim_ss.py`
- 真实数据：`/data/home/grp-wangyf/xuyuan/SDSworkspace/external/1kg_30x/batch_matched/PM_NCN/PM_NCN.chr19.anc` / `.mut`；`.../PM_SCN/PM_SCN.chr19.anc` / `.mut`
- 样本信息：`/data/home/grp-wangyf/xuyuan/SDSworkspace/population_structure/sample_metadata.tsv`
- 通用参考模型：`/data/home/grp-wangyf/xuyuan/CLUES2/Gravel_EAS.coal`

## 输出路径

- 模拟数据自己的人口模型：`/data/home/grp-wangyf/xuyuan/SDSworkspace/population_structure/structure_aware/work/sim_calibrated.coal`
- 校准后的中性零分布（60 个位点）：`.../structure_aware/work/null_cal/`；汇总：`.../results/s4_sim_null_calibrated.tsv`
- 用错模型与用对模型的逐位点对照：`.../results/s4_coal_correction.tsv`
- 真实两群自己算的人口模型：`.../structure_aware/work/real_coal/PM_NCN_calibrated.coal`、`.../PM_SCN_calibrated.coal`
- 真实位点用新模型重算：`.../structure_aware/work/real_cal/`
- 平台标签置信度：`.../results/platform_certainty.tsv`
- 总判决：`.../results/structure_aware_VERDICT.md`
- 已删除：用错模型的旧零分布 `.../structure_aware/work/null/`（752 MB）和 `.../results/s4_sim_null.tsv`

## 参数

- msprime：4 个群体排成一条线，相邻群体迁移率 `m=0.08`，每个群体有效大小 `Ne=3000`，各取 116 个二倍体，长度 2,000,000 bp，重组率和突变率都是 `1.25e-8`，随机种子 303
- 模拟数据自己的人口模型：`RelateCoalescentRate --mode CoalRateForTree -m 1.25e-8 --dist sim_cal.dist -i sim_cal.relate -o sim_calibrated`
- 中性位点推断：`inference.py --coal sim_calibrated.coal --times ... --popFreq ... --tCutoff 600 --df 100 --CI 0.95 --timeBins 50 200`；`SampleBranchLengths` 和 `inference.py` **用同一份** `sim_calibrated.coal`
- 真实两群的人口模型：同样的 `CoalRateForTree` 流程，分别用 PM_NCN、PM_SCN 自己的 `.anc/.mut`
- 真实位点重算：`bin/s_subtree_clues.sh`，`COAL=work/real_coal/<arm>_calibrated.coal`（两步同用），输出到 `work/real_cal/`

## 人口模型

- `Demographic model=sim_stepping_stone_k4_m0.08`（模拟用）；`Gamma shape source=不适用（CLUES2 用 coal，不是 SDS 的 gamma）`；`Present-day Ne(0)=每群体 3000（模拟设定）`；`Model role=mainline（中性零分布）`；`Model note=必须用这批模拟数据自己算出的 coal，不能套 Gravel`
- `Demographic model=Gravel_EAS`（之前用错的）；`Present-day Ne(0)≈45000`；`Model role=rejected（用于模拟数据时）`；`Model note=和模拟设定 Ne=3000 差约 15 倍，会把中性位点算成"有选择"`
- `Demographic model=cohort_empirical_PM_NCN`；`Present-day Ne(0)≈6328`；`Model role=sensitivity（真实位点重算）`
- `Demographic model=cohort_empirical_PM_SCN`；`Present-day Ne(0)≈7481`；`Model role=sensitivity`

## 脚本 / 命令 / 作业号

- `bin/s4_null_site_cal.sh`（模拟中性位点的单点流程）
- `bin/s_subtree_clues.sh`（已支持传入 `COAL`，`SampleBranchLengths` 和 `inference.py` 两处同用）
- `bin/run_real_cal.sh`；LSF 作业 **880728**（smp 队列，8 核，`8*s003`）重算 14 个真实位点（7 个位点 × 2 群）
- `bin/render_verdict.py`（总判决文件改为读取校准后的零分布）

## 结论

1. **人口模型用错的问题确认并已修**：同一批模拟位点，用 Gravel 模型时 logLR 中位 4.32、s2 中位 +0.048；换成模拟数据自己的模型后，60 个中性位点的 logLR 中位 **0.79**（90 分位 2.46，最大 11.98）、s2 中位 **−0.0067**（围绕 0 对称）。逐位点例子：site_43314 的 logLR 从 9.40 降到 0.92。用错模型的旧零分布（`work/null/`）已删除。
2. **平台标签收紧**：只有 222 个公共 1KG/HGDP 样本的测序平台是文献确定的（Illumina）；`HKU_` 开头的样本**不能**当确定 Illumina；BGI 全部是推断的，**没有确定的 BGI 标签**。所以目前无法用确定标签做 Illumina 对 BGI 的比较，之前"两个平台都 NCN>SCN"的结论降级。
3. **真实数据两处 coal 一致**：发表流程 `clues_arm_plat.sh`（第 33 行采样、第 41 行推断）和结构感知驱动里，两处 `--coal` 都是同一份 Gravel，**彼此一致**。但它是通用参考模型，不是队列自己算的；因此又给 PM_NCN/PM_SCN 各算了一份队列自己的模型，并用作业 880728 重算真实位点。
4. **需要提醒的对比前提**：真实位点重算完成前，真实位点（旧 Gravel 标准）和校准零分布（模拟自己模型）**不能直接比较**；"位点是否超出中性"要等 880728 完成后再判。

## 下一步

- 等作业 880728 完成，汇总 14 个单元，得到真实位点在队列自己模型下的 logLR/s2。
- 在标准一致的前提下重新判断位点是否超出中性零分布，并更新总判决。
- 平台效应若还要继续，需要用户提供确定的 BGI 标签清单，否则只能用 222 个确定 Illumina 做单平台描述。