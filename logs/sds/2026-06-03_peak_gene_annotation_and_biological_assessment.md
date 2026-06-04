# SDS 显著峰的基因注释与生物学评估

Date: 2026-06-03
Topic: sds
Type: summary

## Question

当前 NCN Manhattan 上最显著的峰（chr1:125.1M, chr1:143.2M）以及 SCN hybrid-policy 修复后出现的三个超极端窗口（chr1:125.1M, chr9:61.6M, chr16:46.4M）分别落在哪些基因？是否有已知的正选择证据？是否是真实的生物学信号？

---

## 1. chr1:125.1 Mb peak (peak125)

### 1.1 精确位置

- NCN olddefault + rebuilt input significant region: `chr1:125,148,025-125,176,827`
- Top SNP: `chr1:125,170,712:A:T`
- Cytogenetic band: **1q12 (gvar)** — pericentromeric heterochromatin

### 1.2 基因内容

**零个蛋白编码基因。** 该区域完全被 Ensembl 着丝粒注释覆盖（延伸到 125,184,587），包含 99 个重复特征/30kb，几乎全是 **HSATII (Human Satellite II)** 序列。dbSNP 中的所有变异都被归类为 `intergenic_variant`。

### 1.3 最近的有生物学意义的基因

| 基因 | 位置 (GRCh38) | 距峰 | 功能 | 选择证据 |
|---|---|---|---|---|
| SRGAP2C | 121,184,811-121,392,874 | ~4 Mb | 人类特有旁系同源；树突棘发育 | 松弛选择 |
| FAM72B | 121,167,646-121,185,539 | ~4 Mb | 神经发育 | 未明确 |

### 1.4 评估

**极大概率是 mapping artifact。** 峰精确落在 1q12 着丝粒异染色质内，无任何基因，信号模式（数百个极端 SDS SNV 密集聚集）与全基因组其他区域质上不同。1q12 卫星拷贝数在个体间高度多态，不同拷贝数的个体产生 mis-map 的 reads。

---

## 2. chr1:143.2 Mb peak (peak143)

### 2.1 精确位置

- NCN olddefault + rebuilt input significant region: `chr1:143,210,672-143,284,926`
- Top SNP: `chr1:143,272,460:T:C`
- Cytogenetic band: **1q12 (gvar) / 1q21.1 boundary** — pericentromeric heterochromatin distal edge

### 2.2 基因内容

**零个蛋白编码基因。** 90 kb 窗口内有 365 个重复特征，几乎全是 HSATII。dbSNP 变异全部是 `intergenic_variant`。

### 2.3 结构复杂性背景：1q21.1 proximal / TAR susceptibility 邻近区

**关键背景**：peak143 落在 1q12 异染色质远端边界，**紧邻 1q21.1 proximal segmental duplication block**。

1q21.1 proximal 区域（GRCh38 约 144M-146M）是人类基因组中**结构最复杂的区域之一**：
- 包含 BP1-BP4 四个断点（breakpoint），由 clustered segmental duplications 定义
- 容易发生 non-allelic homologous recombination (NAHR)，介导 recurrent deletion/duplication
- **TAR syndrome susceptibility region**（GRCh38: chr1:145,686,999-146,048,495，BP2-BP3 间）含 RBM8A
- 已知基因包括：RBM8A, POLR3C, LIX1L, RNF115, PIAS3, BCL9, PEX11B, CHRNA7, HYDIN 等

peak143 (143.2M) 距 TAR/RBM8A 区域约 **2 Mb**，处于 1q12 satellite DNA 与 1q21.1 segmental duplication block 的过渡地带。

### 2.4 最近的有生物学意义的基因

| 基因 | 位置 (GRCh38) | 距峰 | 功能 | 选择证据 |
|---|---|---|---|---|
| H3-7 | 143,894,544-143,905,977 | ~0.7 Mb | Histone H3.7；染色质结构 | 不明确 |
| NBPF15 | 144,421,390-144,461,676 | ~1.2 Mb | Olduvai (DUF1220) 域；脑大小和认知 | 人类进化 copy gain 正选择 |
| RBM8A | ~145,900,000 (GRCh38) | ~2.7 Mb | RNA binding motif protein 8A；TAR syndrome 关键基因；mRNA 剪接和无义介导的 decay | TAR syndrome 1q21.1 deletion 的致病基因 |
| NOTCH2NLA | 146,145,929-146,229,073 | ~2.9 Mb | 人类特有；增强 Notch 信号、皮层扩张 | 持续适应性进化 |

### 2.5 评估

**不能轻率宣称选择信号的"干净单基因位点"。** 评估分两层：

1. **峰本身仍在无基因的 1q12 异染色质内**：peak143 精确位置（143,209,700-143,286,983）内无蛋白编码基因，以 HSATII 重复为主。信号模式（数百个极端 SNV 密集聚集）仍然更像 artifact 而非经典选择信号。

2. **但紧邻的结构复杂区增加了生物学解释的不确定性**：
   - 1q21.1 proximal 的 segmental duplication 区域本身也是 mapping 困难区域（paralogous sequences 导致 reads 错配）
   - NAHR 介导的 recurrent CNV 在该区域频繁发生，拷贝数多态性可能确实受到选择压力
   - 如果存在真实选择成分，最可能的靶点是 ~2 Mb 外的 RBM8A / 1q21.1 proximal gene cluster 或 ~1.2 Mb 外的 NBPF15/Olduvai 基因

**结论**：peak143 不应作为 paper-facing 的正选择证据。即使有真实生物学成分，它也不是一个可以清晰归因到单一基因或单一选择事件的信号。该区域的信号更可能是 segmental duplication + satellite DNA 的 mapping 复杂性与可能的真实 population-level structural variation 的混合。

Sources:
- [ClinGen 1q21.1 TAR region (ISCA-37428)](https://search.clinicalgenome.org/kb/gene-dosage/region/ISCA-37428)
- [Proximal microdeletions and microduplications of 1q21.1 (PMC3376272)](https://pmc.ncbi.nlm.nih.gov/articles/PMC3376272/)
- [OMIM 612474 - Chromosome 1q21.1 deletion syndrome](https://omim.org/entry/612474)

---

## 3. chr9:61.6-61.8 Mb (SCN hybrid-policy 修复后新出现的超极端窗口)

### 3.1 精确位置

- SCN hybrid-policy 诊断窗口: `chr9:61,600,000-61,800,000`
- Top hit: `chr9:61,696,894:A:C`, rSDS=-1.3045, norm_SDS=-17.6, p=2.2e-69
- Cytogenetic band: **9q13 (gneg)**

### 3.2 基因内容

**零个蛋白编码基因。** 该区域是基因沙漠，仅含：
- FAM242D（lncRNA，无已知功能）
- RN7SL722P（7SL RNA 假基因）
- Y_RNA（DNA 复制相关）

### 3.3 最近的蛋白编码基因

| 基因 | 距区域 | 功能 | 选择证据 |
|---|---|---|---|
| CNTNAP3C | ~147 kb upstream | 突触发育；自闭症相关 | 未明确 |
| SPATA31A7 | ~404 kb upstream | 精子发生 + UV 修复；人类中 copy number 扩张 | **类人猿中最强正选择信号之一**（拮抗多效性） |
| SPATA31A5 | ~680 kb upstream | 同上 | 同上 |

### 3.4 评估

**来源存疑。** 该窗口是 `cap_to_boundary` 修复后才出现的 recovered-only 区域（coarse baseline 中 row_count=0）。raw rSDS 仅 -1.3（moderate），norm_SDS 被高 MAF bin 的极小 SD 放大到 -17.6。可能是 boundary-distance 填充引入的系统性偏差。SPATA31 家族确实有强选择证据，但距离 404 kb，不足以直接将峰归因于该基因。

---

## 4. chr16:46.4-46.5 Mb (SCN hybrid-policy 修复后新出现的超极端窗口)

### 4.1 精确位置

- SCN hybrid-policy 诊断窗口: `chr16:46,400,000-46,500,000`
- Top hit: `chr16:46,452,087:A:C`, -log10p=24.42
- Cytogenetic band: **16q11.2 (gvar)**

### 4.2 基因内容

**零个蛋白编码基因。** 包含：
- ANKRD26P1（假基因）
- 5 个 OCT4-NANOG-H3K27ac-H3K4me1 胚胎干细胞增强子

### 4.3 最近的蛋白编码基因

| 基因 | 距区域 | 功能 | 选择证据 |
|---|---|---|---|
| SHCBP1 | ~78 kb | 细胞分裂 / 纺锤体 | 不明确 |
| VPS35 | ~156 kb | retromer 复合体核心；Parkinson's 病 | **Voight et al. (2006) 在中国人群正选择扫描中识别** |
| ORC6 | ~189 kb | DNA 复制起始 | 同上 |
| MYLK3 | ~202 kb | 心肌功能 | 不明确 |
| NETO2 | ~578 kb | 谷氨酸受体辅助亚基 | 不明确 |

### 4.4 评估

**部分间接证据。** chr16:46.4M 附近区域确实在已发表的中国人正选择扫描中被识别（Voight et al. 2006, PMC1310643），VPS35/ORC6/SHCBP1 是候选基因。但当前 SDS 峰本身落在基因沙漠，且是 boundary-policy 修复后新恢复的区域，norm_SDS inflation 机制明确（raw rSDS moderate，被极小 bin SD 放大）。需要进一步验证是否是 cap_to_boundary 的系统性偏差。

---

## 5. 总结

| 峰 | 基因 | 真实正选择可能性 | 建议 |
|---|---|---|---|
| chr1:125.1M | **0 个**（着丝粒 HSATII） | **极低** — pure pericentromeric mapping artifact | **Mask 掉**，不应进入 paper |
| chr1:143.2M | **0 个**（1q12/1q21.1 过渡带） | **低** — 峰本身无基因，但紧邻 1q21.1 proximal segmental duplication / TAR(RBM8A) 结构复杂区（~2 Mb），有 recurrent CNV 和 NAHR，不能排除混合了真实 structural variation 选择 | **不作为 paper-facing 正选择证据**；不是"干净单基因位点"，如需讨论只能作为 structurally complex region 的注记 |
| chr9:61.6M | **0 个**（基因沙漠） | **低** — boundary repair artifact 可能性高 | 暂不解释；需验证 cap_to_boundary 偏差 |
| chr16:46.4M | **0 个**（基因沙漠 + 增强子） | **低-中** — 有已发表选择证据但峰不在基因上 | 需验证 cap_to_boundary 偏差 + 用 iHS/EHH 正交验证 |

### 建议的 masking 规则

在最终 paper-facing Manhattan 和 significant region table 中：
1. 排除所有 UCSC `cytoBand` 中 `gieStain='acen'`（着丝粒）的区域
2. 排除所有 `gieStain='gvar'`（heterochromatin）的区域，尤其是 1q12
3. 可选：排除端粒 ±500 kb
4. 可选：排除 UCSC `segmentalDuplication` track 中 large clustered duplications（如 1q21.1 BP1-BP4）
5. 对 recovered-only（boundary-policy 修复后新出现）的窗口做额外的 norm_SDS 分布审计

## Key files

- NCN significant regions: `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_newinput_20260514/NCN/NCN.significant_regions.tsv`
- SCN hybrid-policy diagnostics: `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_hybridpolicy_chunk20_20260531b/SCN/diagnostics/`
- GRCh38 cytoBand: UCSC genome browser `cytoBand` track

## Conclusion

- chr1:125.1M 峰落在纯 1q12 着丝粒异染色质内，无任何基因或结构复杂性背景，**确认为 mapping artifact**。
- chr1:143.2M 峰本身也在无基因的 1q12 异染色质内，但**紧邻 1q21.1 proximal segmental duplication / TAR susceptibility 区域**（RBM8A 等基因在 ~2 Mb 外）。该坐标段落在结构复杂区，BP1-BP4 断点间的 segmental duplication 容易发生 NAHR 介导的 recurrent CNV。这个背景本身不说明是 artifact，但说明它**不是一个可以轻率宣称选择信号的"干净单基因位点"**——即使有真实成分，也是 segmental duplication mapping 复杂性与可能的 population-level structural variation 的混合。
- SCN hybrid-policy 修复后新出现的 chr9:61.6M 和 chr16:46.4M 峰也落在基因沙漠；chr16 有间接的已发表选择证据但距离 ~150 kb。
- 所有四个峰的共同特征：**不在蛋白编码基因上**。这强烈支持在 paper-facing 分析中增加 pericentromeric/subtelomeric masking 以及 segmental duplication region masking。

## Next step

- 将 chr1:125.1M 和 143.2M 从 paper-facing candidate list 中移除
- 在 input pipeline 中实现着丝粒 / heterochromatin mask
- 对 chr9/chr16 修复窗口的 norm_SDS inflation 做进一步诊断
