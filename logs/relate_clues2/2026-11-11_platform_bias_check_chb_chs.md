# 平台假象排查：位点朝向、窗口测序质量、窗口稀有突变，用 CHB/CHS 做同平台对照

Date: 2026-11-11
Topic: relate_clues2
Type: test
Question: 我们队列里 DPP9/MIR7-3 在 NCN 里偏高的选择信号，是不是测序平台/检出批次造成的？用三个角度排除，并用 1KG 的 CHB/CHS 做"同一台机器"的对照。

## 输入路径

- 队列基因型：`/data/home/grp-wangyf/xuyuan/raw/9k/snp_singleton_qc_chr19_v1/snp_qc_pass.vcf.gz`（只有基因型 GT）
- 队列样本质量：`/data/home/grp-wangyf/xuyuan/raw/9k/sample_qc/sample_qc_metrics.tsv`、`/data/home/grp-wangyf/xuyuan/SDSworkspace/population_structure/sample_metadata.tsv`
- 1KG 对照（同一台机器）：`/data/home/grp-wangyf/xuyuan/SDSworkspace/external/1kg_30x/qc/CHB.chr19.qc.vcf.gz`、`.../CHS.chr19.qc.vcf.gz`
- 1KG 位点结果（已有）：`/data/home/grp-wangyf/xuyuan/SDSworkspace/population_structure/replication_1kg_chb_chs/s2_tmrsa_ALL_CHB_CHS.tsv`、`.../reproduction_verdict.tsv`
- 队列位点结果（本轮重算）：`/data/home/grp-wangyf/xuyuan/SDSworkspace/population_structure/structure_aware/work/real_cal/`

## 输出路径

- 队列窗口质量与稀有突变：`/data/home/grp-wangyf/xuyuan/SDSworkspace/population_structure/structure_aware/results/s6_window_qc_sfs.tsv`
- 随机窗口参照：`.../results/s6_baseline_windows.tsv`、`.../results/s6_qc_sfs_summary.tsv`
- CHB/CHS 窗口质量：`.../results/s7_chb_chs_qc_control.tsv`
- 两对样本的对照表：`.../results/s7_platform_bias_table.tsv`
- 两份判决文件：`.../results/s6_platform_screen_VERDICT.md`、`.../results/s7_platform_bias_VERDICT.md`

## 参数

- 比较的两群人：`PM_NCN` / `PM_SCN` 各 233 人（来源批次构成相同）；1KG 对照 CHB 103 人、CHS 105 人
- 窗口：每个位点上下游 50 kb 和 100 kb
- 随机窗口参照：整条 chr19 上随机取 20 个 100 kb 窗口（避开位点 ±300 kb）
- 统计量：基因型缺失率、平均深度 DP、平均碱基质量 GQ、祖先/衍生型翻转标记、窗口内"只出现一次/两次的突变"比例
- 脚本：`structure_aware/bin/s6_window_qc_sfs.py`、`bin/s6b_baseline.py`、`bin/s7_chb_chs_control.py`

## 人口模型

- 位点 s2 来自 CLUES2：队列用 `PM_NCN/PM_SCN` 自己算的 coal（见 2026-11-10 记录），CHB/CHS 用 `Gravel_EAS.coal`。两套口径不同，所以 s2 绝对值不可直接比。
- `Gamma shape source=不适用（CLUES2 用 coal）`；`Present-day Ne(0)=见 coal 反推（队列约 6300/7500，1KG 用 Gravel）`；`Model role=audit-only`；`Model note=本记录只做平台/批次排查，不改选择结论`

## 结论（人话版）

1. **位点朝向一致**：7 个位点在 NCN 和 SCN 里，哪个是祖先型、哪个是衍生型，判断完全相同（都不需要翻转）。内部偶发的分支翻转只占 1–5%，没有哪一群系统性更高。UHRF1 是唯一两群 s2 正负相反的位点，但它的朝向判断相同、数值大小也不同，不是标签反了，而是 SCN 这边频率在下降。
2. **测序质量没有断崖**：两群的测序深度和碱基质量几乎一样（38.6 / 93.8），没有"南方深度/质量突然掉一截"的情况。
3. **缺失率和稀有突变是整条染色体的背景差，不是位点周围特有**：NCN 的基因型缺失率整体偏高（全基因组 0.0040 对 0.0034），"只出现一次/两次的突变"比例也略高；但整条染色体随机窗口里同样偏高（随机 20 窗的缺失率差 +0.00081±0.00068、稀有突变比差 +0.88±1.46），这 7 个位点窗口并没有超出这个范围。
4. **同平台对照**：CHB 和 CHS 同一台机器、同一套流程、样本量接近（103 对 105），它们的测序质量完全一样（缺失率 0/0，深度 35.6 对 35.4，质量 94.3 对 94.2）。**即使这样，位点仍然有差异**（DPP9 差异是随机波动的 7.49 倍，DNM2 是 5.91 倍）。说明流程在"零平台差异"的一对上照样能检出人群差异，位点信号不是平台造出来的。
5. **总的判断**：没有找到明显的平台假象。位点差异不能归因于这几个位点周围的测序质量/缺失率/稀有突变异常。但 NCN 整体缺失率偏高是**全队列范围的背景混杂**，需要继续控制；按批次配平的对照臂并没有把它完全消掉。

## 必须保留的限制

- 队列的 VCF 只有基因型，没有逐样本逐位点的深度/质量；窗口内的深度/质量只能用整条 chr19 的样本均值代替。
- 随机窗口参照只有 20 个，判断"是否异常"的精度有限。
- 确定的测序平台标签只有 222 个 Illumina 样本，没有确定的 BGI；所以这里的"平台"只能理解为检出批次差异，不能落到具体仪器。
- CHB/CHS 的质量文件基因型是全的（缺失率=0），缺失率绝对值不能跨数据集比。

## 下一步

- 若要继续追平台，需要用户提供确定的 BGI 标签清单。
- 若要控制"NCN 整体缺失率偏高"这个背景差，可以按缺失率/深度做样本配平后重跑位点比较。