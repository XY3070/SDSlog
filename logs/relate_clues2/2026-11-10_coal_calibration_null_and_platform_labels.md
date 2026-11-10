# 结构感知验证：模拟零分布 coal 校准、平台标签收紧、真实队列 coal 重算

Date: 2026-11-10
Topic: relate_clues2
Type: test + production
Question: 修正"结构感知中性零分布"用错 .coal 的问题；按确定标签收紧平台效应结论；核查真实数据 SampleBranchLengths 与 CLUES2 的 --coal 一致性，并给真实队列生成自己的校准 coal 重算 lead。

## Input paths

- 模拟 ARG：`/data/home/grp-wangyf/xuyuan/SDSworkspace/population_structure/structure_aware/work/sim_cal.relate.anc` / `.mut`（msprime 4-deme stepping-stone，2 Mb）
- 模拟参数脚本：`/data/home/grp-wangyf/xuyuan/SDSworkspace/population_structure/structure_aware/bin/s4_sim_ss.py`
- 真实臂 ARG：`/data/home/grp-wangyf/xuyuan/SDSworkspace/external/1kg_30x/batch_matched/PM_NCN/PM_NCN.chr19.anc` / `.mut`；`.../PM_SCN/PM_SCN.chr19.anc` / `.mut`
- 样本元数据：`/data/home/grp-wangyf/xuyuan/SDSworkspace/population_structure/sample_metadata.tsv`
- 通用参考 coal：`/data/home/grp-wangyf/xuyuan/CLUES2/Gravel_EAS.coal`

## Output paths

- 模拟自身 coal：`/data/home/grp-wangyf/xuyuan/SDSworkspace/population_structure/structure_aware/work/sim_calibrated.coal`
- 校准中性零分布（60 位点）：`/data/home/grp-wangyf/xuyuan/SDSworkspace/population_structure/structure_aware/work/null_cal/`；汇总表 `.../results/s4_sim_null_calibrated.tsv`
- coal 错配配对对照（8 位点）：`.../results/s4_coal_correction.tsv`
- 真实臂校准 coal：`.../work/real_coal/PM_NCN_calibrated.coal`、`.../work/real_coal/PM_SCN_calibrated.coal`
- 真实 lead 重算（运行中）：`.../work/real_cal/`
- 平台标签确定性表：`.../results/platform_certainty.tsv`
- 判决文件：`.../results/structure_aware_VERDICT.md`
- 已删除：旧错 coal 零分布 `.../work/null/`（752 MB）与 `.../results/s4_sim_null.tsv`

## Parameters

- msprime：`k=4` deme 一维 stepping-stone，`m=0.08`，`Ne=3000`/deme，`n=116` 二倍体/deme，`L=2,000,000` bp，重组率=突变率=`1.25e-8`，`seed=303`
- 模拟 coal 生成：`RelateCoalescentRate --mode CoalRateForTree -m 1.25e-8 --dist sim_cal.dist -i sim_cal.relate -o sim_calibrated`（dist 由 `RelateExtract --mode ExtractDistFromMut` 生成）
- 校准零分布 CLUES2：`inference.py --coal sim_calibrated.coal --times <site>_times.txt --popFreq <PF> --tCutoff 600 --df 100 --CI 0.95 --timeBins 50 200`；`SampleBranchLengths` 与 `inference.py` 同用 `sim_calibrated.coal`
- 真实臂 coal 生成：同 `CoalRateForTree` 流程，分别用 PM_NCN / PM_SCN 的 `.mut` 与 `.anc`
- 真实 lead 重算：`bin/s_subtree_clues.sh`，`COAL=work/real_coal/<arm>_calibrated.coal`（SBL 与 CLUES2 同用），`TRAJFLAG=""`，输出 `work/real_cal/`

## Demographic model

- `Demographic model=sim_stepping_stone_k4_m0.08`（模拟）；`Gamma shape source=n/a（CLUES2 用 coal，非 SDS gamma）`；`Present-day Ne(0)=3000/deme（模拟设定）`；`Model role=mainline（中性零分布）`；`Model note=coal 必须由该模拟自身 ARG 经 CoalRateForTree 生成，不能套 Gravel`
- `Demographic model=Gravel_EAS`（旧错配口径）；`Present-day Ne(0)≈45000（近期，由 coal 首率 1.11e-5 反推）`；`Model role=rejected（用于模拟零分布时）`；`Model note=与模拟 Ne=3000 相差约 15 倍，把中性位点算出高 logLR/正 s2`
- `Demographic model=cohort_empirical_PM_NCN`；`Present-day Ne(0)≈6328（coal 非零率中位 7.90e-5 反推）`；`Model role=sensitivity（真实 lead 重算）`
- `Demographic model=cohort_empirical_PM_SCN`；`Present-day Ne(0)≈7481（中位率 6.68e-5）`；`Model role=sensitivity`

## Scripts / commands / job IDs

- `bin/s4_null_site_cal.sh`（校准零分布单位点流程）
- `bin/s_subtree_clues.sh`（已参数化 `COAL`，SBL 第 40 行与 CLUES2 第 59 行同用 `$COAL`）
- `bin/run_real_cal.sh`；LSF job **880728**（smp，8 核，`8*s003`，RUN）重算 14 个真实 lead 单元（7 lead × 2 臂，all 子集）
- 渲染：`bin/render_verdict.py`（§2 已改读 `s4_sim_null_calibrated.tsv`）

## Conclusion

1. **coal 错配确认并修正**：同一批模拟位点，Gravel coal 给 logLR 中位 4.32 / s2 +0.048；换模拟自身 `sim_calibrated.coal` 后，n=60 中性零分布 logLR 中位 **0.79**（p90 2.46，max 11.98）、s2 中位 **−0.0067**（围绕 0 对称）。配对例：site_43314 logLR 9.40→0.92、s2 +0.100→+0.033。旧零分布（`work/null/`）已删除。
2. **平台标签收紧**：只有 222 个公共 1KG/HGDP 编号是文献确定 Illumina（NCN 82 / SCN 21 / neither 119）；`HKU_` 前缀按用户说明降级（不是确定 Illumina）；BGI 全为推断（medium），**无确定 BGI 标签**。因此目前无法用确定标签做 Illumina-vs-BGI 对照， prior"两平台都 NCN>SCN"结论降级。
3. **真实数据 coal 一致性**：发表流程 `clues_arm_plat.sh`（`COAL=Gravel_EAS.coal`，SBL 33 行 / CLUES2 41 行）与结构感知驱动两处 `--coal` 均一致（同 Gravel）。满足一致性要求；但用的是通用参考而非队列校准 coal，故已生成 PM_NCN/PM_SCN 经验 coal 并提交 job 880728 重算 lead。
4. **口径警示**：真实 lead 重算完成前，真实 lead（旧 Gravel 口径）与校准零分布（模拟自身 coal 口径）不可直接比较；"lead 是否超出中性"须待 job 880728 完成后在匹配口径下判定。

## Next step

- 等 job 880728 完成，汇总 `work/real_cal/` 的 14 个单元，得到真实 lead 在队列校准 coal 下的 logLR/s2。
- 在匹配口径下重新评估 lead 相对校准零分布的位置；更新 `structure_aware_VERDICT.md`。
- 平台效应若要继续，需用户提供确定 BGI 标签清单，否则只能用 222 个确定 Illumina 做单平台描述。