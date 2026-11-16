# Panmictic 消融 + 真实基因组频率/梯度双匹配背景

Date: 2026-11-16
Topic: relate_clues2
Type: test
Question: (1) 去掉人群结构、其余流程完全相同，是否仍会产生"显著"的中性位点？(2) 用真实基因组上频率相同、南北频率差（Δp）也相同的中性位点当背景，DPP9/MIR7-3 等 lead 还突出吗？

## 输入路径

- 队列基因型：`/data/home/grp-wangyf/xuyuan/raw/9k/snp_singleton_qc_chr19_v1/snp_qc_pass.vcf.gz`
- 队列频率/Δp：`work/annot/all_ac.tsv`、`ncn_ac.tsv`、`scn_ac.tsv`（按 NCN 1,227 / SCN 2,537 重算 AC/AN）
- 基因坐标：`work/annot/chr19_genes.bed`（Ensembl REST 抓取的 chr19 3,573 个基因）
- 重组图：`/data/home/grp-wangyf/xuyuan/SDSworkspace/external/1kg_30x/relate_work/CHB/chr19_relate.map`
- 真实臂 ARG 与 coal：`work/real_coal/PM_NCN_calibrated.coal`（PM_NCN 臂）
- 模拟：`work/sim_pan.*`（panmictic）、`work/sim464.*`（stepping-stone）

## 输出路径

- Panmictic 零分布：`work/null_pan/`（150 位点）、`results/s10_pan_vs_ss.tsv`
- 双匹配背景候选：`results/s9_background_candidates.tsv`（7 lead × 100）
- 背景 CLUES 结果：`work/bg_PM_NCN/`（每个 lead 50–55 格）、`results/s11_bg_vs_lead.tsv`
- 判决文件：`results/s12_ablation_and_matched_background_VERDICT.md`

## 参数

- **Panmictic**：`s4_sim_ss.py --m 0 --k 1 --n 232 --L 2000000 --seed 303` → 464 条单倍型，实测 F_ST=−0.002；Relate（LSF 作业 363807）→ 自身 coal → 3–8% 频率位点按同一流程跑 150 个。
- **Stepping-stone 对照**：`--m 0.08 --k 4 --n 116 --out_demes 0,3`，其余完全相同（已有 `work/null464/`，150 位点）。
- **背景筛选**：距 lead >5 Mb（离任何 lead >1 Mb）、不在基因体内、不在重组率前 10%（p90=3.43）、|AF−AF_lead|≤0.02、|Δp−Δp_lead|≤0.005；每个 lead 取最接近的 100 个，实际跑成 50–55 个。
- **背景分析**：PM_NCN 臂，队列自己的 coal，`SampleBranchLengths` 与 CLUES2 同用；`--df 100 --tCutoff 600 --timeBins 50 200`。

## 人口模型

- `Demographic model=sim_panmictic_k1_Ne3000_464hap`；`Present-day Ne(0)=3000`；`Model role=audit-only`；`Model note=无结构消融`
- `Demographic model=sim_stepping_stone_k4_m0.08_out23_464hap`；`Present-day Ne(0)=3000/群`；`Model role=mainline（零分布对照）`
- `Demographic model=cohort_empirical_PM_NCN`；`Present-day Ne(0)=近期约 3.7 万`；`Model role=sensitivity（真实背景）`

## 结论

1. **Panmictic 与 stepping-stone 零分布没有差别**（各 150 位点；logLR 中位 0.73 vs 0.77，KS p=0.53）。去掉结构不会让零分布变紧 → logLR 的噪声主要来自 CLUES2/ARG 采样流程本身，不是人群结构。
2. 无结构数据里有 **5/150（3.3%）** 中性位点超过 stepping-stone 零分布最大值 3.90，最高 5.62；但 150 个无结构位点里**没有一个达到 lead 的 9.4/9.9**。
3. **真实双匹配背景**（50–55 个/lead）下，极性正确的 5 个 lead 经验 p 为：DPP9 **0.137**、MIR7-3 **0.078**、PLIN3 0.679、UHRF1 0.321、DNM2 0.929——**没有一个显著**。DPP9 的匹配背景 logLR 中位 6.08、最高 15.81；MIR7-3 的背景最高 10.57。
4. **结论**：DPP9/MIR7-3 "过模拟零分布"不能升级为选择证据。它们的高 logLR 是"带有同样南北差（Δp）的真实位点"的普遍性质；用模拟零分布做阈值会高估显著性，真实双匹配背景才是严格对照。

## 必须保留的限制

- 真实背景只在 PM_NCN 臂上跑（未做 PM_SCN 臂）。
- 约 13% 的候选位点在臂的 ARG 中缺失被跳过，缺失可能非随机，轻微影响背景分布。
- "高重组热点"用房为前 10% 的定义，换阈值会改变入选位点。
- Panmictic 与 stepping-stone 都是每群 Ne=3000，与队列近期 Ne（约 3.7 万）不同；各自自校准，绝对数值不可互换。

## 下一步

- 在 PM_SCN 臂重复真实背景（检验是否臂特异）。
- 把阳性对照（ADH1B rs1229984 / ALDH2 rs671）走同一套背景流程，确认这把尺子能把"真强选择"与"同 Δp 的普通位点"分开。