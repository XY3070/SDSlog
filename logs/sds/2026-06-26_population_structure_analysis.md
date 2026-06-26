# 2026-06-26 Population structure analysis: ADMIXTURE + badMIXTURE + PCA-UMAP

## Summary

对 9k Chinese cohort (4,393 samples) 完成三层次 population structure 分析：(1) ADMIXTURE unsupervised K=2..10 离散聚类；(2) badMIXTURE 残差检验——逐位点对比模型预测与真实基因型，用量化颜色随机性评估拟合质量；(3) PCA-UMAP 连续梯度——分别用 common (MAF>1%) 和 rare (MAF 0.1%–1%) variant sets 做 50-PC 降维后的 UMAP 可视化，检验离散假设之外的连续群体结构。

核心发现：ADMIXTURE 离散模型在 per-SNP 水平拟合差（badMIXTURE Row ACF ~0.86），个体间存在模型未捕获的系统性关联；PCA-UMAP 证实 common variants 已能分辨地理结构，rare variants 可能提供更精细的分层信号。

## 输入

| 文件 | 来源 | 说明 |
|---|---|---|
| Genotype (ADMIXTURE) | `SDSworkspace/runs/admixture_9k_v1_20260619/input/9k_ldpruned.{pgen,psam,pvar}` | LD-pruned (50/5/0.2), 107,927 SNPs, 4,393 samples |
| Genotype (PCA-UMAP common) | `raw/9k/snp_singleton_qc/snp_qc_pass.vcf.gz` → `runs/umap_9k/9k_snp_qc_maf01` | MAF>1%, 580,363 → LD-prune → 107,927 SNPs, 4,392 samples |
| Genotype (PCA-UMAP rare) | 同上 VCF → `runs/umap_9k/9k_snp_qc_rare_maf01` | MAF 0.1%–1%, 508,963 → LD-prune → 376,481 SNPs, 4,382 samples |
| Sample metadata | `raw/9k/sample_qc/cohort_freeze.tsv` | Region labels：Guangzhou, Shenzhen, HongKong, Shanghai, Jinan, JiNing, Unknown |

## ADMIXTURE: Unsupervised K=2..10

**脚本:** `SDSworkspace/oneoff/run_badmixture_analysis.R` (cluster plots, K evaluation, region composition)  
**Q 矩阵:** `runs/admixture_9k_v1_20260619/unsupervised/9k.{K}.Q`  
**P 矩阵:** `runs/admixture_9k_v1_20260619/unsupervised/9k.{K}.P`

### K 评估（非 CV 指标）

使用 Evanno Delta-K、50% subsample stability、normalized entropy、ambiguous proportion (maxQ<0.6) 四个指标评估最优 K。

### 输出

| 文件 | 说明 |
|---|---|
| `runs/umap_9k/badmixture/cluster_plots_per_K.pdf` | Per-K 四面板：Q heatmap (50/cluster sampled)、PCA of Q、cluster sizes、mean ancestry per cluster |
| `runs/umap_9k/badmixture/full_heatmap_K{2..10}.png` | 全样本 Q 矩阵热图（按 cluster + maxQ 排序） |
| `runs/umap_9k/badmixture/K_evaluation.png` | 六面板 K 评估：LL、Delta-K、Stability、Entropy、Ambiguous%、metrics table |
| `runs/umap_9k/badmixture/region_cluster_K*.png` | Best-K by-region 聚类组成 barplot |

## badMIXTURE: 残差检验

**脚本:** `SDSworkspace/oneoff/run_badmixture_residuals.R`  
**方法:** 对每个 K，sample 50/cluster（与先前分析相同，无重新抽样），提取 2,000 随机 SNP 的基因型 (plink2 --export A)，计算 `G_exp = 2·Q·P^T`（ADMIXTURE 模型期望），残差 `R = G_obs - G_exp`。用 badMIXTURE 蓝-白-红 divergent palette 可视化残差矩阵，以颜色随机性量化拟合质量。

### 残差质量指标

| K | ResidualVar | MAE | Row ACF | Col ACF | \|R\|>1 | Randomness* |
|---|------------|------|---------|---------|------|-------------|
| 2 | 3.036 | 1.634 | 0.861 | 0.008 | 82.1% | 0.870 |
| 3 | 3.041 | 1.635 | 0.858 | 0.012 | 82.2% | 0.871 |
| 4 | 3.047 | 1.637 | 0.855 | 0.013 | 82.3% | 0.868 |
| 5 | 3.047 | 1.637 | 0.858 | 0.010 | 82.3% | 0.867 |
| 6 | 3.050 | 1.639 | 0.859 | 0.011 | 82.4% | 0.870 |
| 7 | 3.050 | 1.639 | 0.860 | 0.010 | 82.4% | 0.870 |
| 8 | 3.049 | 1.638 | 0.859 | 0.010 | 82.4% | 0.869 |

*Randomness = |Row ACF| + |Col ACF|，越低 = 越随机 = 拟合越好。

### 解读

- **Row ACF ~0.86（极高）：** 个体间残差高度相关，ADMIXTURE 离散聚类假设无法捕获的系统性个体间关联。这是 badMIXTURE 的核心诊断信号。
- **Col ACF ~0.01（低）：** SNP 间残差基本独立，无 SNP 特异性偏倚。
- **Randomness 跨 K 平坦（~0.87）：** 增加聚类数不能改善拟合。模型在低 K 已饱和。
- **82% 大残差（|R|>1）：** per-SNP 基因型预测偏差普遍较大，部分与低 MAF SNP 的二项方差大有关。

结论：离散混合模型单独不足以描述该 cohort 的群体遗传结构，需要连续梯度方法补充。

### 输出

| 文件 | 说明 |
|---|---|
| `runs/umap_9k/badmixture/badmixture_residuals_K{2..8}.png` | 四面板残差图：主热图 (蓝-白-红)、color scale、cluster bar、mean residual per individual |
| `runs/umap_9k/badmixture/badmixture_quality_vs_K.png` | 六面板跨 K 对比：ResidualVar、MAE、Randomness、Row ACF、Col ACF、PropLarge |

## PCA-UMAP: 连续梯度探索

**脚本:** `SDSworkspace/oneoff/run_umap_9k.sh` (common), `SDSworkspace/oneoff/run_umap_9k_rare.sh` (rare)  
**Pipeline:** MAF filter → LD prune (50/5/0.2) → top 50 PCs (plink2 --pca) → UMAP (n_neighbors=30, min_dist=0.1, random_state=42)

### Common variants (MAF>1%)

| 阶段 | SNP count | Sample count |
|---|---|---|
| Input VCF | 14.56M | 4,393 |
| MAF>1% filter | 580,363 | 4,392 |
| LD prune | 107,927 | 4,392 |
| PCA (top 50) | 107,927 | 4,392 |

### Rare variants (MAF 0.1%–1%)

| 阶段 | SNP count | Sample count |
|---|---|---|
| Input VCF | 14.56M | 4,393 |
| 0.1%<MAF<1% filter | 508,963 | 4,383 |
| LD prune | 376,481 | 4,382 |
| PCA (top 50) | 376,481 | 4,382 |

Rare variants 在 LD prune 后保留 74% (376k/509k)，远高于 common variants 的 19% (108k/580k)——rare variants 间 LD 更低，保留了更多独立信号。

### UMAP 输出（每 variant set 5 图）

| Common 输出 | Rare 输出 | 着色 | 目的 |
|---|---|---|---|
| `*_umap_region.png` | `*_rare_umap_region.png` | Region (GZ/SZ/HK/SH/JN/JiN/Unknown) | 地理标签在连续流形上的分布 |
| `*_umap_admixture_k2.png` | `*_rare_umap_admixture_k2.png` | ADMIXTURE K=2 dominant cluster | 离散聚类与连续梯度的一致/冲突 |
| `*_umap_pc1.png` | `*_rare_umap_pc1.png` | PC1 (inferno colormap) | 主成分梯度结构的连续性 |
| `*_umap_pc12_bicolor.png` | `*_rare_umap_pc12_bicolor.png` | R=PC1, G=PC2 bicolor | 双 PC 结构同时可视化（纯色=PC结构清晰，泥色=噪声） |
| `*_umap.png` | `*_rare_umap.png` | 灰度（仅点分布） | 整体拓扑 |

所有输出在 `SDSworkspace/runs/umap_9k/`。

### UMAP 坐标文件

`9k_snp_qc_umap.tsv` / `9k_snp_qc_rare_umap.tsv`：含 FID、IID、UMAP1、UMAP2、Region、K2_Cluster、PC1、PC2

## 跨方法结论

1. **ADMIXTURE 离散聚类**在 cohort 层面有区分度（region × cluster 交叉可见结构），但 **badMIXTURE 残差检验** 揭示模型在 per-SNP 水平存在严重 misfit（Row ACF ~0.86），个体间有离散假设未捕获的隐秘连续关联。
2. **PCA-UMAP 连续方法** 证实 common variants 已能分辨地理结构（region-colored UMAP 可见区域分离），弥补了离散模型的盲区。
3. Rare variants (376k SNPs post-LD-prune) 提供了 3.5× more SNP markers，可能揭示更精细的 recent demographic events。
4. badMIXTURE randomness score 跨 K 平坦 → 增加聚类数不改善拟合质量，模型选择不应仅依赖 CV/Delta-K。

## 相关脚本

| 脚本 | 路径 |
|---|---|
| ADMIXTURE cluster + K eval | `SDSworkspace/oneoff/run_badmixture_analysis.R` |
| badMIXTURE 残差 | `SDSworkspace/oneoff/run_badmixture_residuals.R` |
| UMAP common variants | `SDSworkspace/oneoff/run_umap_9k.sh` |
| UMAP rare variants | `SDSworkspace/oneoff/run_umap_9k_rare.sh` |
| ADMIXTURE single-figure | `SDSpipeline/scripts/plot_ancestry_one_figure.py` |
