# 2026-08-16 家系遗传图 (genetic maps) — ZhongHua 芯片位点全基因组定案

Date: 2026-08-16
Status: **COMPLETE** — 全基因组 22 chr 地图 (247 trio, ZhongHua-8 芯片位点)
Role: `mainline` — 用户主导的家族遗传图计算, 用于下游工具 cM 坐标 + LocusZoom x 轴
Parent: 2026-08-06_sle_family_locuszoom.md (旧图版 LocusZoom), 2026-06-26_microarray_fastsmc_data_preparation.md (芯片数据)

## 背景与目标

- 用 lidingyang 的家族 WGS 数据 (874 trio, 9466 样本, GRCh38) 计算遗传图
- 原设想 pyrho — **官方确认 pyrho 不支持家系数据** (仅无关个体 LD 推断) → 改用家系直接数交叉互换 (crossover counting)
- 数据: `/data/home/grp-wangyf/lidingyang/shapeit/` (SHAPEIT5 5.1.1 phased, pedigree-aware)

## 关键发现 (审计路径)

1. **面板样本重复**: plink1.9 `--genome` (PI_HAT≥0.9) + plink2 全基因组 KING (kinship≥0.45) 联合 → 14 对高置信重复 (DT/NJ ID 变体, THOMSON/VH108, NM/SH 双孩家系); 4 个 KING 边界对实为一级亲属 (kinship 0.25-0.28, PI_HAT 0.54-0.59) — chr1 单染色体 KING 高估; 874→868 家, 6 个重复孩子剔除; 0 样本交换 (亲子 kinship 0.256±0.004)
2. **1KG 家系构成**: 583 trio, 仅 44 CHS (南方汉) 为中国; 无 CHB/JPT; KHV 21 (越南) 应排除
3. **中国子集正则 bug**: `^[a-z]+$` 漏了 23 个 sz+数字家系 → 修正后 247 trio (204 前缀 + 44 CHS − 1 dup)
4. **WGS 全位点相位噪音**: 混合 9466 面板父母相位错误 → mask 14-22%, CO 计数虚高 2×, 性别比平. 换位点集是解药
5. **ZhongHua matched bed 损坏**: 61% 位点 AC=0 (lift 错误); **CSV (GRCh37) + pyliftover → hg38 = 0.14% AC=0** — CSV 可用
6. **MCMC seed/thread 敏感**: thread=10 vs 20 + seed 差异 → chr1 CO 计数 25% 波动 → 固定 `--seed 15052011 --thread 20`
7. **断点 vs 率形**: 20-trio 跨位点集断点 overlap 仅 13%, 但 5Mb window 率形等价 — 率形是正确比较层级

## 定案管线

```
InfiniumOmniZhongHua-8v1-4_A1.csv (GRCh37) 
  → pyliftover + hg19ToHg38.chain → hg38 位点 (chrN: ~90k)
  → analysis BCF 提取 (GT 与 QC vcf 一致 99.5%; QC vcf 单点索引提取不可行: 129GB, 4h+/chr)
  → 过滤: 双等位 SNP, 无 indel, 无 strand-ambiguous (A/T,C/G), F_MISSING<1%, 逐位点孟德尔 (247 trio), 无 MAF 限制
  → SHAPEIT5 phase_common --pedigree (247 trio, 729 样本) --seed 15052011 --thread 20 --map b38
  → 02_count_crossovers.py --caller persistence (掩膜 0%, 持久性 ≥1Mb, 间距 ≥5Mb, 离群剔除)
  → 03_build_maps.py (250kb 分箱, σ=1.5Mb, 总长校准 b38)
```

脚本: `SDSpipeline/scripts/genetic_maps/` (00-07 + common.sh, git committed); 管线编排 `SDSworkspace/cache/genetic_maps/pilot/{omni_chr.sh,zh_genome.sh}`

## 结果 (247 trio, 494 减数分裂)

- **全基因组 5Mb Spearman vs b38 = 0.904** (Pearson 0.895) — WGS 0.37 → Omni2.5 0.62 → 芯片 0.90
- mask 0/996 bins 全 chr; CO/mei: pat 35.8 (预期 25-35), mat 38.0 (预期 40-50, 欠调 ~5-10%)
- 性别比 1.06 — 残余母系欠调 (247 全量下部分恢复; 断点间距 pat/mat 相同 → 位置结构正常, 数量少)
- chr 模式无异常 (chr19 正常, acrocentric 无 inflation)
- 总量校准 b38 (0% dev)

## 输出 (两级)

- chr-level: `SDSworkspace/cache/genetic_maps/pilot/omni/out_zh247/events/main/chrN.crossovers.tsv` (CO_raw + flags=CO_filtered), `bins/`, `maps/main/chrN.genetic_map.txt` + `.pat/.mat` (性别特异) + `.gmap` + `stats.tsv`
- genome-level: 22 chr 图 (合并使用)
- LocusZoom: `SDSworkspace/results/production/sds_output/ALL/plots/SLE_family/chr7_50266267_zh247.locuszoom.png` (新图; rs4917014 → 72.186 cM)
- manifest: `SDSworkspace/provenance/runs/genetic_maps_20260808_18056/run_manifest.yaml`

## 遗留 / 下一步

1. 性别比 1.06 — mat 欠调 ~5-10% (HMM 对母系 switch 敏感度 / 参考差异), 地图总量已校准
2. 性别特异地图质量未单独验证 (pat/mat 图已输出, 未做 06 验证)
3. 中国子集专用地图 (chinese subset) 需用新位点集重建
4. QC vcf 原始 GT 提取基础设施不可行 (129GB 单点索引) — 若需原始 GT 需批量区间并行
