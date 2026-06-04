# chr6 MHC 区域在所有 baseline 中缺乏正选择信号

Date: 2026-06-03
Topic: sds
Type: summary

## Question

chr6 MHC (28.5M-33.5M) 是人类基因组中已知正选择信号最强的区域之一（HLA 基因的 balancing selection 和 directional selection）。为什么在当前所有 SDS baseline 中，MHC 区域完全没有 Bonferroni 显著的 positive selection 信号？这与预期矛盾。

---

## 1. 跨 baseline 事实

### 1.1 显著区域

| Baseline | chr6 MHC (28.5M-33.5M) 显著区域 | chr6 整体显著区域 |
|---|---|---|
| NCN olddefault + rebuilt input | **0 个** | 5 个，全在 MHC 外（最近的是 chr6:19.9M，距 MHC ~8.6 Mb） |
| NCN olddefault mainline (historical) | **0 个** | 4 个，全在 MHC 外 |
| SCN olddefault repaired (coarse) | **0 个** | 0 个（chr6 全染色体无显著区域） |

### 1.2 MHC 区域 raw rSDS 分布

**NCN olddefault + rebuilt input**（chr6.sds.tsv, 99,016 SNPs in MHC）：
- 正尾（正选择方向）：仅 **2 个** SNP 的 rSDS > +5.0
  - top: chr6:29,982,680, rSDS=7.94, DAF=0.9996（基本固定的 singleton artifact）
- 负尾（负选择方向）：**2,619 个** singleton SNP (DAF=0.0004) 的 rSDS 在 -13 到 -14 之间
  - 这些是极端 rare variant，SDS 估计极其不稳定

**SCN olddefault repaired**（chr6.sds.tsv, 112,981 SNPs in MHC）：
- 正尾：**0 个** SNP 的 rSDS > +5.0（max rSDS = 2.60）
- 负尾：**2,684 个** singleton SNP 的 rSDS 在 -13 到 -14 之间

### 1.3 chr6 input lineage audit

根据 `2026-05-15_chr1_chr6_input_lineage_audit.md`：
- chr6:32,500,001-32,600,000（MHC 内部）：oldsplit 有 862 个 extra test SNP，**全部 non-PASS**（主要是 LowQD / LowAAScore）
- 在同一 rebuilt input 上，olddefault gamma 产出安静的 chr6，而 Gravel_CHB@100k 放大 chr6 到 extreme tail
- 结论：chr6 差异主要来自旧链保留了低质量位点

---

## 2. 为什么 MHC 没有信号？

### 2.1 可能原因分析

| 假设 | 评估 | 证据 |
|---|---|---|
| **olddefault gamma shape 在 MHC 频率范围过于保守** | 可能 | MHC 选择信号主要在 intermediate frequency，olddefault 在高 DAF 的 shape 极小（0.03-0.06），null 分布极窄，但 rSDS 本身可能就不够大 |
| **MHC 高重组率稀释 singleton 模式** | 可能 | SDS 依赖 singleton 密度差异，MHC 区域重组率极高（尤其是 HLA 基因间），singleton 分布可能更均匀 |
| **NCN 样本量不足** | 可能 | NCN 只有 1255 个样本；MHC 高度多态，等位基因频率分布复杂；power 可能确实不足 |
| **WGS variant calling 在 MHC 质量差** | 部分成立 | MHC 的 HLA 基因高度多态，短读长 mapping 在该区域质量不好；chr6:32.5M-32.6M 的 862 个 extra SNP 全是 non-PASS 说明该区域 calling 确实困难 |
| **Gravel_CHB@100k 曾有 chr6 信号但是 artifact** | 已确认 | Gravel_CHB@100k 在同一 rebuilt input 上产生 chr6 extreme tail，但这是 gamma shape 过大放大的 artifact，不是真信号 |

### 2.2 与 chr1 artifact 的对比

| 区域 | 显著 SNV 数 | 信号性质 | 评估 |
|---|---|---|---|
| chr1:125.1M (pericentromeric) | 177/551 = 32.1% | mapping artifact | 假阳性 |
| chr1:143.2M (pericentromeric) | 211/626 = 33.7% | mapping artifact | 假阳性 |
| chr6:MHC (28.5M-33.5M) | 0 | 真信号未被捕获 | 假阴性 |

当前 pipeline **对假信号敏感（着丝粒 artifact 被放大），对真信号不敏感（MHC 选择信号缺失）**。

---

## 3. 建议

### 3.1 正交验证

MHC 是否有 Chinese-specific 正选择不能仅靠 SDS 判断：
- **iHS/EHH**：MHC 是 iHS 的经典强信号区域，应优先检查
- **HLA 等位基因频率**：检查 HLA-A, HLA-B, HLA-DRB1 等在中国人群中的等位基因频率分布是否与全球人群有显著偏移
- **已发表文献对比**：Field et al. (2016) 在英国人群中用 SDS 观察到了 MHC 信号；需要理解为什么中国人群没有

### 3.2 技术层面

- 考虑 MHC 是否需要 HLA imputation + allele-specific selection test
- 检查 MHC 区域的 singleton 分布是否与全基因组有系统性差异（singleton 密度、DAF 分布等）
- 如果问题出在 gamma model，那解决 gamma provenance 问题后 MHC 信号可能会恢复

---

## Key files

- NCN chr6 SDS output: `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_newinput_20260514/NCN/chr6.sds.tsv`
- NCN significant regions: `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_newinput_20260514/NCN/NCN.significant_regions.tsv`
- SCN chr6 SDS output: `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_coarse_20260528/SCN/chr6.sds.tsv`
- chr6 lineage audit: `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/2026-05-15_chr1_chr6_input_lineage_audit.md`

## Conclusion

- chr6 MHC 在所有 baseline 中都没有 Bonferroni 显著的 positive selection 信号
- MHC 正尾几乎空白，负尾有大量不稳定的 singleton artifact
- 这与 MHC 是已知正选择最强区域的预期矛盾
- 可能原因包括：gamma model 过于保守、MHC 高重组率稀释 singleton 模式、样本量不足、variant calling 质量问题
- 当前 pipeline 存在"假阳性敏感、假阴性不敏感"的结构性问题

## Next step

- 用 iHS/EHH 对 chr6 MHC 做正交验证
- 在 gamma provenance 问题解决后重新检查 MHC 信号
- 检查 MHC 区域的 singleton 分布特征是否与全基因组有系统性差异
