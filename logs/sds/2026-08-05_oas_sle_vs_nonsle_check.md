# 2026-08-05 OAS1/OAS3 (chr12) normSDS: SLE vs non-SLE check

Date: 2026-08-05
Status: **COMPLETE** — no significant case-control difference
Role: `mainline` (targeted follow-up on user question)
Parent entries: 2026-08-02_sle_vs_nonsle_sds_comparison.md, 2026-08-02_sle_vs_nonsle_confounder_diagnostic.md, 2026-08-02_sle_vs_nonsle_synthesis.md, 2026-07-28_orthogonal_evidence_final_report.md (chr12_112949kb_sig1)

## Question

The ALL=4383 scan signal `chr12_112949kb_sig1` (OAS3, block 112,877,945-113,020,120, lead chr12:112971371) sits in the OAS1/OAS3 cluster. User asks: does normSDS at OAS3 and OAS1 differ between SLE (N=1965) and non-SLE (N=2367)?

## Inputs / setup

- **Demographic model:** Gravel CHB (`g_file.Gravel_EAS.txt`); Ne(0) implicit/unknown.
- **Input lineage:** `raw/9k/snp_singleton_qc_chr{N}_v1/snp_qc_pass.vcf.gz` → group-specific singleton redefinition (SLE vs nonSLE singletons NOT inherited from ALL).
- **Sample policy:** SLE=1965, non-SLE=2367, ALL=4383 (same pipeline, separate runs, Aug-1/2).
- **Data:** `sle_vs_nonsle_comparison/threeway_merge.tsv` (SLE ∩ nonSLE ∩ ALL = 4,452,790 variants, per-variant normSDS + delta) — reused, no new compute.
- **Region (Ensembl GRCh38):** OAS1 112,905,856-112,933,219 (+); OAS3 112,938,051-112,976,460 (+); OAS2 112,978,387-113,011,723 (+). Cluster body 112,900,000-113,020,000; ±50 kb flank window 112,850,000-113,060,000 (388 variants).
- Scripts: `SDSworkspace/oneoff/oas_case_control_sds.py`, `SDSworkspace/oneoff/oas_region_plot.py` (one-off; no pipeline change).

## Results

| Region | n | mean SLE | mean nonSLE | mean ALL | mean Δ(SLE−nonSLE) | z vs genome-wide Δ | paired Wilcoxon p | frac SLE>nonSLE | n(norm>2) ALL/SLE/nonSLE |
|---|---|---|---|---|---|---|---|---|---|
| cluster ±50kb | 388 | +0.092 | +0.107 | +0.086 | −0.015 | −0.12 | 0.825 | 0.518 | 94 / 88 / 90 |
| cluster body | 288 | +0.070 | +0.054 | +0.043 | +0.016 | +0.40 | 0.788 | 0.531 | 51 / 47 / 46 |
| OAS1 body | 91 | +0.036 | +0.003 | +0.044 | +0.034 | +0.38 | 0.946 | 0.517 | 2 / 2 / 2 |
| OAS3 body | 88 | +0.047 | +0.003 | −0.026 | +0.043 | +0.46 | 0.871 | 0.545 | 16 / 12 / 11 |
| OAS2 body | 55 | −0.174 | −0.160 | −0.215 | −0.014 | −0.04 | 0.795 | 0.582 | 12 / 12 / 12 |

Genome-wide Δ reference (Aug-2): mean −0.0085, SD 1.0616. All region z-scores within noise (−0.12 to +0.46).

**Lead SNP chr12:112971371 (OAS3 3'UTR):** SLE=6.00, nonSLE=6.67, ALL=6.13 — extreme normSDS in BOTH groups; Δ=−0.67 (nonSLE slightly higher, single variant).

**Spatial (20 kb bins):** every bin with ALL>2 variants is mirrored ~1:1 in SLE and nonSLE (e.g. 113.00-113.02 Mb: 22/22/22; 112.88-112.90: 15/15/15). Region correlations: r(SLE,ALL)=0.966, r(nonSLE,ALL)=0.912, **r(SLE,nonSLE)=0.816 vs genome-wide 0.435** — the OAS cluster is MORE concordant between groups than the genome average.

## Interpretation

- **No case-control difference at OAS1/OAS3/OAS2.** The strong ALL-cohort selection signal (lead normSDS ≈ 6) is fully shared: SLE and nonSLE carry the same high-normSDS variants at nearly identical frequency of extreme scores.
- Directional hints (OAS3 mean Δ +0.04, OAS1 median SLE +0.70 vs nonSLE −0.50) are not significant (all p>0.75, z<0.5) and not consistent across the cluster — treat as noise.
- Caveats (from Aug-2 confounder diagnostic, unchanged): SLE/nonSLE differ in regional composition (Shenzhen enrichment in SLE) and singleton burden; per-group normSDS are separately calibrated. A null finding here is robust to these confounds only in the sense that no effect survives even without correction.

## Round 2 (same day): p-value comparison + two-cohort LocusZoom

User follow-up: normSDS 无差异，p-value 是否有差异？并绘制两个 cohort 的 LocusZoom（区分 cohort 的点）。

**p-value 定义:** `p_bothside = 2·min(norm.cdf(z), norm.sf(z))`，z=normSDS — p 是 |normSDS| 的严格单调变换（postprocess_sds_results.py:544-549）。

**区域整体 (n=388 joint):** median -log10p SLE 0.649 vs nonSLE 0.702；mean Δ(-log10p) = −0.083；Wilcoxon p=0.81（无差异）；KS p=2.8e-7（分布形状不同）；Spearman(-log10p) = 0.555。Bonferroni 显著数 2 vs 3。

**Per-gene body:**

| Region | n | frac(SLE>nonSLE) -log10p | paired Wilcoxon p | 驱动因素 |
|---|---|---|---|---|
| OAS1 body | 91 | 0.879 | **5.9e-12** | SLE |z| 更极端 (mean|z| 1.09 vs 0.77)，**sign corr(SLE,NS)=−0.82** |
| OAS3 body | 88 | 0.648 | 0.28 (ns) | 混合，无一致方向 |
| OAS2 body | 55 | 0.291 | 0.007 | nonSLE |z| 更极端 (1.61 vs 1.35) |

**Lead SNP 112971371 (OAS3 3'UTR):** SLE p=1.97e-9 (rank 2/390), nonSLE p=2.52e-11 (rank 1/396), ALL p=8.6e-10 — 均过基因组 Bonferroni (1.1e-8)；nonSLE p 比 SLE 小 ~78×。SLE 区域 rank-1 变异为 112927816 (OAS1 3'端, p=1.45e-9; SLE z=6.05, nonSLE z=6.44)。

**解读:** OAS1 body 的 p 差异是**幅度差异**（SLE 的 |normSDS| 系统性更高），且同一变异 rSDS 符号两组反相。**注意：极化 (AA/DA) 完全相同（同一参考 fasta + 同一 ancestral state）——反相的是 rSDS 符号，不是极化方向。** 机制（已验证）：rSDS = (观测派生 singleton 数 − 模型期望)/SD，期望基于各组自己的 DAF 和样本量；组间 DAF 不同（OAS1 例: 0.218 vs 0.253）→ 期望不同 → 偏差符号可翻转。微观证据: OAS1 body 91 变异中 raw rSDS sign corr = **−0.824**（原始 rSDS 层面已反相，非归一化 artifact），两组 bin_id 相同率 **0%**（连 MAF bin 都不重叠 → per-group 归一化完全不可比）。故幅度/符号差异由组间样本构成（区域组成，Aug-2）驱动，不能解读为 selection 差异。

**SLE 更强的方向性假设检验（用户假设: SLE OAS 功能过剩 → SLE z 更大）— 不支持:**

| 检验 | OAS family union (n=253) | cluster±50kb (n=388) |
|---|---|---|
| 单侧 Wilcoxon SLE>nonSLE on z | p=0.41 (ns) | p=0.59 (ns) |
| 单侧 Wilcoxon on −log10p | p=0.007* | p=0.60 (ns) |
| mean Δz (SLE−nonSLE) | +0.023 | −0.015 |
| n(z>0) SLE / nonSLE | 45% / 45% | 48% / 49% |
| n(z>2) SLE / nonSLE | 29 / 28 | 88 / 90 |
| mean \|z\| SLE / nonSLE | 1.24 / 1.15 | 1.61 / 1.62 |

*仅幅度（|z|）单侧显著，但 OAS1 sign 反相（r=−0.82）→ 双向更极端而非正方向更强，且 z>2 计数持平 (29 vs 28) — 与"功能过剩→正向选择"预测不符。Lead SNP 反而 nonSLE 更强 (z 6.67 vs 6.00, p 2.5e-11 vs 1.97e-9)。

**两 cohort LocusZoom:** `plot_locuszoom_two_cohort.py`（one-off）— 复用 canonical `plot_locuszoom.py` 的 load_region_data/compute_ld/load_recombination_rate/fetch_gene_annotations/plot_gene_track；SLE=实心三角 ▲、nonSLE=空心圆 ○（同 LD 颜色 r² vs lead, joint 4383 VCF），Bonferroni 线，recomb + gene track（1 lane, 7 genes）。LD 5519 pairs, recomb 0.1-5.3 cM/Mb。输出 `sle_vs_nonsle_comparison/oas_region/locuszoom_two_cohort_OAS.png`（1656×915, 150dpi）→ 归档 `SDSlog/plots/sds/2026-08-05_oas_case_control/`。p 对比表 `oas_region/pvalue_comparison.tsv`。

## Round 3 (same day): selection-strength test — Method 1 (cohort×DAF interaction regression)

User 提供 3 种检验方法，要求选最优。**选定 Method 1**：线性回归 cohort×DAF 交互（形式化、连续 DAF、全部变异、可 region-vs-rest 对照）。弃 Method 2（bin 放宽到重叠 — 宽度 ad hoc、双重校准、power 低）；弃 Method 3（ΔDAF vs ΔSDS 相关 — 基因组基线已有 r≈0.16, chr12=0.159 [Aug-2 diagnostic]，构成 artifact 方向，区域 n 下歧义）。

**模型** (stacked per-variant-per-cohort, chr12 joined 215,730 variants → 431,460 rows; OLS + cluster-robust SE by variant, 手写 sandwich — statsmodels 不在 sds env)：
`normSDS ~ DAF + cohort + region + DAF:cohort + DAF:region + cohort:region + DAF:cohort:region`（region = OAS family body union 234 variants vs chr12-rest）

**结果：**

| 尺度 | β(DAF×cohort) | SE (cluster) | p |
|---|---|---|---|
| **chr12 全 3-way: β(DAF:cohort:region)** | **+2.623** | 0.259 | **3.7e-24** |
| OAS family 单独 | +2.442 | 0.259 | 3.6e-21 |
| cluster±50kb 单独 | +2.406 | 0.251 | 7.7e-22 |
| chr12-rest 单独 | **−0.181** | 0.0086 | 3.2e-97 |

**DAF 分层均值 (OAS family):** DAF<0.5: SLE −0.55 vs nonSLE +0.07 (Δ −0.62); DAF≥0.5: SLE +0.97 vs nonSLE −0.23 (**Δ +1.20**)。交叉模式 — SLE 在低 DAF 更低、高 DAF 更正，斜率差 +2.44/单位 DAF。

**验证:** ① 独立重实现交叉验证 cluster SE=0.2586 ✓ (homo SE 0.443 — cluster 未低估)；② LD 非独立性: family 内与 lead r²>0.1 仅 42 变异 (OAS3 3' 单块)，OAS1 驱动变异不与 lead 连锁；有效独立块 ~2-3，SE×5 后仍 p~1e-8 — 结论稳健；③ raw rSDS 同方向交叉（非归一化凭空产生，OAS1 高 DAF 块 rSDS SLE +0.12~0.14 / nonSLE −0.03~−0.05）。

**结论: 数据支持 "SLE 在 OAS 高 DAF 区 SDS 更正" 的交互模式**（方向与用户假设预测一致，region-vs-rest 极显著）。**但机制上不能排除构成/校准 artifact**（Aug-2: per-group MAF-bin 归一化 artifact + 区域构成差异；bin_id 两组重叠 0%）。raw rSDS 层面即存在的真实交叉 → 不是归一化凭空造出，但构成差异（Shenzhen 富集 SLE）可驱动 DAF 移位 → 交叉模式。判定：**假设方向性支持，需 confounder 验证（同构成样本 or 频率差异独立验证）才能论文级结论**。

**清理（用户要求删冗余）:** 删 `oas_region_sle_vs_nonsle.png`（Round-1 normSDS 散点图，已被双 cohort LocusZoom + selection_strength 图取代；本日志为 supersede 记录）。保留 stats/variants/pvalue TSV（分别回答 normSDS/p-value 问题）、locuszoom_two_cohort_OAS.png。未生成 Method 2/3 输出。

**输出:** `oas_region/selection_strength_test.tsv` (21 行, 4 尺度×7 项) + `selection_strength_OAS.png` (1934×769, 150dpi, 双 panel 散点+拟合线+内嵌系数表) → 归档 `SDSlog/plots/sds/2026-08-05_oas_case_control/`。脚本 `SDSworkspace/oneoff/oas_selection_strength_test.py`。

## Outputs

- `sle_vs_nonsle_comparison/oas_region/oas_region_stats.tsv`, `oas_region_variants.tsv`
- `oas_region_sle_vs_nonsle.png` (2-panel: per-variant normSDS scatter + Δ with running median; gene bodies shaded; lead SNP marked) — archived `SDSlog/plots/sds/2026-08-05_oas_case_control/`
- No pipeline/script change → no git commit (workspace one-offs only).
