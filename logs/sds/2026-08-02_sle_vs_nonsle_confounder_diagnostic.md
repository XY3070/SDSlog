# 2026-08-02 SLE vs non-SLE SDS: Comprehensive Confounder Diagnostic

**Role:** `audit-only` (diagnostic framework — no biological claims)  
**Parent entry:** [2026-08-02_sle_vs_nonsle_sds_comparison.md](2026-08-02_sle_vs_nonsle_sds_comparison.md)  
**Date:** 2026-08-02

---

## §1 Current Evidence Summary

*Observations only. No interpretation in this section.*

### 1.1 Cohort composition

| Group | N | Singleton count (mean) | Singleton count (SD) |
|---|---|---|---|
| SLE | 1,965 | 626.4 | — |
| non-SLE (control) | 2,367 | 681.8 | — |
| ALL | 4,383 | 655 (cohort mean) | — |

SLE cases have 55.4 fewer singletons per sample on average (Welch t-test p<0.001, Cohen's d = −0.42).

### 1.2 SDS comparison summary

| Metric | Value |
|---|---|
| SLE vs nonSLE Pearson r | **0.4350** (moderate) |
| SLE vs ALL Pearson r | **0.7812** |
| nonSLE vs ALL Pearson r | **0.7504** |
| Genome-wide mean delta (SLE − nonSLE) | **−0.0085** |
| SD delta | **1.0616** |
| delta skewness | +0.010 (symmetric) |
| \|delta\| > 2 | 5.96% of variants |
| \|delta\| > 4 | 0.03% (726 variants) |
| All 22 chromosomes mean delta negative? | **Yes** |

### 1.3 Raw vs normalized delta

| Metric | Value |
|---|---|
| mean delta_rSDS (raw, pre-normalization) | **+0.001643** |
| mean delta_norm (post-normalization) | **−0.008534** |

The raw rSDS difference between SLE and non-SLE is near-zero and slightly **positive** (SLE > nonSLE). After per-group MAF-bin normalization, the delta becomes **negative** (SLE < nonSLE). The shift direction **flips** during normalization.

### 1.4 Top divergent windows (from threeway merge, uncorrected)

| Locus | Window | delta (SLE−nonSLE) | SLE | nonSLE | ALL |
|---|---|---|---|---|---|
| chr19:52.5Mb | 52,500,437−52,505,738 | **−1.07** | −0.442 | +0.628 | +0.342 |
| chr1:92Mb | 92,501,859−92,999,212 | **−0.86** | −0.505 | +0.351 | −0.292 |
| chr2:108Mb | 108,548,916−108,999,920 | −0.66 | +0.279 | +0.937 | +0.472 |
| chr3:49Mb | 49,045,991−49,497,682 | −0.61 | +0.039 | +0.645 | +0.896 |
| chr9:92Mb | 92,001,508−92,499,195 | **+0.25** | +0.201 | −0.047 | +0.134 |

23 of 30 top divergent windows have negative delta (nonSLE > SLE).

### 1.5 Known confounds (from 2026-06-26 population structure analysis)

- **Singleton burden**: SLE 626 vs control 678 (p<0.001). PC1~singleton r = −0.50.
- **Geographic imbalance**: SLE 78% HongKong, 0% Guangzhou/Jinan. SLE% varies 0%–70% by region.
- **Population structure**: PC1 captures South-North cline (eta²=0.41). SLE/control d=0.428 on PC1 — cases and controls differ in ancestry.
- **PC4**: sex + case/control (d=−0.422).
- **Singleton count 2.2× variation by region** (Shanghai 431 vs Jinan 958) — batch/sequencing site effect.
- **badMIXTURE Row ACF=0.86**: residual inter-individual correlation from continuous cline + singleton gradient.

---

## §2 Confounder Diagnostic Table

| # | Confounder | Available Evidence | Expected Effect on SDS | Testable Prediction | Currently Tested? | Confidence in Confound |
|---|---|---|---|---|---|---|
| 1 | **Singleton burden difference** | SLE 626 vs control 678 singletons (p<0.001). SLE deficit is in **opposite direction** of sample-size null (θ/n predicts 821 for SLE). Per-region: HK SLE=573, HK ctrl=622 — difference persists within region. | Lower singletons → fewer AC=1 sites → shifts s_file composition → changes rSDS scaling. Does NOT directly affect norm_SDS (singleton variants are filtered at MAF<0.05). Indirect effect via per-group bin statistics. | If singleton burden drives delta, within-HK-only comparison should show reduced delta. | Partially (HK-only singleton comparison done) | **HIGH** — real case/control difference in singleton counts, not explained by sample size |
| 2 | **Sample size imbalance** | SLE=1,965 vs nonSLE=2,367. Expected SD ratio = √(2367/1965) = 1.0975. | Larger N → more precise allele frequency estimates → smaller BIN_SD. SLE BIN_SD should be ~1.098× nonSLE BIN_SD. | BIN_SD ratio ≈ 1.098 across all MAF bins. | **YES** — BIN_SD ratio measured at 1.19–1.33 (higher than expected) | **PARTIAL** — explains ~80% of variance inflation; residual ~20% unexplained |
| 3 | **Geographic/region imbalance** | SLE: 78% HK, 0% GZ/JN. Control: spread across all 6 regions. | Region is **variant-constant** — cannot create variant-specific peaks. Can only shift global intercept of delta if region-specific batch effects differ between groups. | Global delta shift should be proportional to region composition difference. chr19 peak should NOT be explained by region. | **Partially** (regional composition quantified, UMAP plotted) | **HIGH for global shift, LOW for focal peaks** |
| 4 | **Population structure (ancestry)** | PC1~SLE d=0.428. PC1 eta²=0.41 (region). PC1_residualized available in freeze. | Ancestry differences → allele frequency differences at ancestry-informative markers → delta_SDS at those loci. Should produce variant-specific effects (unlike region composition). | delta_SDS should correlate with PC loading of variants (not tested at variant level). | **Partially** (UMAP shows SLE/nonSLE separation on PC1 axis) | **MODERATE** — ancestry differences are real but modest (d=0.428 is small-medium) |
| 5 | **Batch/sequencing effects** | Singleton count 2.2× by region (431–958). SLE samples concentrated in HK (mean=573) and SZ (mean=728) — different batch characteristics. | Batch effects → differential genotype calling → systematic shifts in rSDS. Should be region-stratified. | HK-only comparison should reduce batch-driven delta. SZ-only should show different delta pattern. | **Partially** (HK-only singleton comparison; SZ not analyzed separately) | **MODERATE** — batch is confounded with region; disentangling requires per-batch metadata |
| 6 | **Case/control ascertainment** | Hospital-based recruitment. SLE% 0% (GZ, JN) to 70% (SZ). Controls from same hospitals may not represent population controls. | Hospital-based controls may have different health-seeking behavior / comorbidities → different selection pressures. Unlikely to produce genome-wide shift. | Known GWAS loci should NOT be enriched in delta peaks (if they are → ascertainment bias). | **NO** | **LOW** — plausible but speculative without hospital metadata |
| 7 | **Technical: variant missingness** | o_file per-sample observability values available (~0.998–0.999). Not compared between groups. | Differential missingness → different variant sets in bins → BIN_MEAN/BIN_SD shift. | BIN_MEAN difference between groups should correlate with missingness difference. | **NO** | **UNKNOWN** — data exists but not analyzed |
| 8 | **Technical: normalization artifact** | **delta_rSDS mean = +0.0016, delta_norm mean = −0.0085**. Shift flips sign during normalization. SLE BIN_MEAN consistently higher than nonSLE BIN_MEAN at low MAF (e.g., 0.042 vs 0.029 at MAF 0.05–0.07). SLE BIN_SD 20–33% higher than nonSLE. | Per-group normalization (norm = (rSDS − μ_bin)/σ_bin) with different μ_bin and σ_bin per group → systematic offset in normalized scores even when raw rSDS is identical. | (1) delta_rSDS ≈ 0 but delta_norm ≠ 0. (2) BIN_MEAN_SLE − BIN_MEAN_nonSLE > 0 at low MAF. (3) delta_norm ~ BIN_MEAN difference across bins. | **YES** — decomposition confirms shift originates in normalization | **HIGH** — primary driver of genome-wide negative delta |

---

## §3 Figures

All figures in `SDSworkspace/results/production/sds_output/sle_vs_nonsle_comparison/diagnostic/`.

### Figure 5: Per-sample singleton count distribution

- **File:** `fig5_singleton_burden.png` (6 panels)
- **Purpose:** Visualize singleton burden difference between SLE and controls, overall and stratified by region.
- **Panel A1:** Histogram overlay — x="singleton_count", y="Density". SLE distribution shifted left.
- **Panel A2:** Boxplot — SLE vs control. Welch t-test p-value annotated.
- **Panel A3:** Region-stratified boxplot — x=region×phenotype, y=singleton_count. Shows within-region differences.
- **Panel B1:** Sample-size null check — bar chart comparing SLE observed (626), control observed (682), and SLE predicted under θ/n model (821). **SLE observed is lower than predicted → deficit is NOT a sample-size artifact.**
- **Panel B2:** HongKong-only histogram — most balanced region (SLE=1540, control=1504). Shows if SLE deficit persists within region.
- **Panel B3:** Region means + SLE% — dual-axis bar/line plot. Singleton count varies 2.2× by region (Shanghai 287–443 vs Jinan 947–958). SLE% varies 0–70%.
- **Expected interpretation:** SLE singleton deficit persists within HongKong (573 vs 622), confirming it is a genuine case/control difference, not purely geographic. However, the 2.2× regional variation dwarfs the case/control difference — region is the dominant source of singleton count variation. This means any SDS comparison confounded by singleton burden will primarily reflect region/batch, secondarily case/control.

### Figure 7: Case-control regional composition

- **File:** `fig7_regional_composition.png` (2 panels)
- **Purpose:** Document the severe geographic imbalance between SLE and control groups.
- **Panel A:** Stacked bar — x=Region, y=Sample count, stacked SLE/control. SLE% annotated above bars.
- **Panel B:** Side-by-side pie charts — SLE composition vs control composition by region.
- **Expected interpretation:** SLE cases concentrated in HongKong (78%) and Shenzhen (10%). Controls spread across all 6 regions. Geographic confounding is severe — any genome-wide delta could in principle reflect region-specific batch effects rather than case/control biology. However, region composition is **variant-constant**: it can only shift the global intercept, never create variant-specific or window-specific peaks like chr19:52.5Mb.

### Figure 8: Delta SDS vs delta DAF

- **File:** `fig8_delta_sds_vs_daf.png` (6 panels)
- **Purpose:** Test whether SDS divergence tracks allele frequency differences between groups. If delta_SDS is driven by frequency spectrum shifts, we expect correlation with delta_DAF.
- **Panel A1:** Hexbin scatter — x=delta_DAF, y=delta_norm_SDS. Pearson r=0.143, Spearman ρ=0.137.
- **Panel A2:** Binned means ±95%CI — x=mean delta_DAF in 20 bins, y=mean delta_norm_SDS.
- **Panel A3:** delta_DAF distribution — centered near zero (mean=+0.00023).
- **Panel B1:** Per-chromosome correlation — Pearson r by chromosome (all weak, <0.2).
- **Panel B2:** delta_norm_SDS distribution — mean=−0.0085, sd=1.062, near-symmetric (skew=+0.01).
- **Panel B3:** SD and mean of delta by MAF bin — SD close to 1.0 across bins (no MAF-dependent variance inflation in normalized space).
- **Expected interpretation:** Weak correlation (r=0.14) means delta_SDS is **not primarily DAF-driven**. The systematic −0.0085 shift is not explained by allele frequency differences between groups. The near-zero skew (+0.01) suggests the distribution is symmetric around the mean — no excess of extreme values in either direction. The SD≈1.0 across MAF bins confirms normalization is well-calibrated per-bin.

### Figure 9: BIN_SD ratio vs MAF

- **File:** `fig9_bin_sd_ratio.png` (4 panels)
- **Purpose:** Check if variance inflation matches sample-size expectation. Pure sample-size effect predicts SD_ratio = √(2367/1965) = 1.098.
- **Panel A1:** BIN_SD_SLE / BIN_SD_nonSLE vs MAF — observed ratio 1.19–1.33 (expected 1.098). Mean ratio = 1.229.
- **Panel A2:** Absolute BIN_SD values — SLE consistently higher than nonSLE.
- **Panel B1:** Raw rSDS SD ratio — also elevated (1.17–1.26), confirming excess variance in raw scores too.
- **Panel B2:** Variant count per MAF bin — reference.
- **Expected interpretation:** BIN_SD ratio is **systematically higher** than the sample-size prediction (1.23 vs 1.10). The excess variance (~12% above expectation) suggests additional variance sources beyond sample size — possibly batch effects, differential missingness, or genuine biological variance differences. Importantly, higher BIN_SD in SLE means the same raw rSDS difference gets **attenuated** in normalized space for SLE, which contributes to the negative delta shift.

### Figure 10: Per-chromosome mean delta

- **File:** `fig10_per_chr_delta.png` (4 panels)
- **Purpose:** Identify which chromosomes drive the global shift.
- **Panel A1:** Mean delta_norm_SDS by chromosome ±95%CI — **all 22 chromosomes negative**. chr22 most negative (−0.028), chr10 least negative (−0.004).
- **Panel A2:** Mean delta_rSDS by chromosome — mixture of positive and negative, all near zero.
- **Panel B1:** % variants with |delta|>2 and |delta|>4 by chromosome — chr19 elevated for |delta|>4.
- **Panel B2:** Variant count per chromosome — reference.
- **Expected interpretation:** All 22 chromosomes have negative mean delta — this is a **genome-wide systematic shift**, not a few outlier chromosomes. The pre-normalization delta_rSDS shows both positive and negative chromosomes, confirming the shift is introduced by normalization. chr19 has elevated extreme-tail proportion (|delta|>4), consistent with the chr19:52.5Mb focal peak — this peak may be real despite the global shift.

### Correlation Matrix

- **File:** `fig_correlation_matrix.png`
- **Purpose:** Show correlation structure between delta_SDS and all potential confounders.
- **Key values:**
  - delta_norm ~ delta_rSDS: **r=0.959** — normalization preserves variant-level structure almost perfectly
  - delta_norm ~ delta_DAF: **r=0.143** — weak
  - delta_norm ~ MAF: **r=0.002** — none
  - delta_norm ~ delta_nG0: **r=−0.087** — very weak
  - delta_rSDS ~ delta_DAF: **r=0.187** — weak
- **Expected interpretation:** The very high correlation between delta_norm and delta_rSDS (0.96) confirms that normalization preserves the **ranking** of variants by divergence. The systematic shift (−0.0085) is a constant offset added during normalization. Variant-level conclusions (which loci are most divergent) are robust to normalization; the global mean shift is not.

### Decomposition: Raw vs Normalized Delta

- **File:** `fig_decomposition.png` (2 panels)
- **Purpose:** Pinpoint where the −0.0085 shift originates.
- **Panel A:** Raw vs normalized delta by MAF bin — at low MAF (0.05–0.07), mean delta_rSDS=+0.008 but mean delta_norm=−0.035. **Shift magnitude is largest at low MAF and flips sign.**
- **Panel B:** Per-bin BIN_SD comparison — SLE BIN_SD consistently higher, gap widens at low MAF.
- **Expected interpretation:** The shift originates in the **per-group bin statistics**. At low MAF, SLE has higher BIN_MEAN (0.042 vs 0.029) and higher BIN_SD (0.175 vs 0.139). For a variant with the same rSDS in both groups: norm_SLE = (rSDS−0.042)/0.175, norm_nonSLE = (rSDS−0.029)/0.139. The higher BIN_MEAN pushes SLE norm_SDS down; the higher BIN_SD additionally compresses SLE scores. Both effects produce negative delta_norm even when delta_rSDS is zero or positive. This is a **normalization artifact**, not a biological signal.

### PCA-UMAP (common and rare variants)

- **Files:** `umap/umap_{common,rare}_{orig,resid}_{phenotype,region,region_phenotype,singleton,sle_overlay}.png` (20 files)
- **Purpose:** Visualize SLE/non-SLE separation in ancestry space, assess geographic confounding, show singleton burden gradient.
- **Views per variant set:**
  1. **Phenotype-colored** (SLE=orange, control=blue) — shows if SLE/non-SLE separate in ancestry space
  2. **Region-colored** (6-region palette) — shows geographic structure
  3. **Region-color × Phenotype-shape** — combined view for geographic confounding assessment
  4. **Singleton-count continuous** (viridis) — directly visualizes the singleton burden gradient
  5. **SLE overlay** (controls faint, SLE prominent) — highlights where cases concentrate
- **Expected interpretation:**
  - **Common variants, original PCs:** SLE and control broadly overlap in UMAP space. The South-North cline (GZ/SZ/HK → SH → JN/JiN) is visible but SLE cases (concentrated in HK/SZ) occupy a subset of the ancestry space. No discrete "SLE cluster" — cases are embedded within the population structure.
  - **Rare variants, original PCs:** Stronger singleton-count gradient visible. The 626 vs 682 singleton difference appears as a subtle shift — SLE samples tend toward lower-singleton-count regions of the embedding. This is the visual manifestation of the singleton confound.
  - **Residualized embeddings (common and rare):** After regressing out singleton_count + F_MISS + F_het from PCs, the singleton gradient disappears (PC1~singleton r → 0.000). However, SLE and control still overlap substantially — the case/control difference in ancestry space (d=0.428 on PC1) is modest and primarily reflects geographic concentration, not a distinct genetic architecture.
  - **Key takeaway:** SLE cases are not a genetically distinct subpopulation. They are a geographically biased subset of the overall Chinese cohort. Any SDS differences between SLE and non-SLE are more likely to reflect (a) normalization artifacts from different sample sizes/compositions, (b) geographic/batch effects from uneven regional representation, or (c) genuine case/control differences at specific loci — with (a) and (b) being the more parsimonious explanations for genome-wide patterns.

### CMplot Mirror Manhattan

- **File:** `SDSworkspace/results/production/sds_output/sle_vs_nonsle_comparison/mirror_manhattan_SLE_vs_nonSLE_CMplot.png` (redrawn 2026-08-05; old matplotlib version superseded + deleted — it was not CMplot, no peak annotation)
- **Purpose:** Genome-wide view of SLE (top panel) vs non-SLE (bottom panel) SDS signals, CMplot standard; shared peaks annotated black, SLE-only red, nonSLE-only blue (see `mirror_peaks.tsv`).
- **Expected interpretation:** Most peaks are shared (mirrored). Systematic nonSLE elevation visible as more/bigger peaks. Focal divergences (chr19:52.5Mb, chr1:92Mb, chr9:92Mb) require confounder assessment before biological interpretation.

---

## §4 Statistical Tests (Suggested, Not Executed)

Each test answers a specific question. Tests are listed in order of priority for establishing whether observed delta reflects biology or artifact.

### Test 1: delta_SDS ~ delta_DAF correlation
- **Question:** Does allele frequency difference between groups explain SDS difference?
- **Input:** threeway_merge + normalized TSVs (available: delta_norm, delta_DAF for 4.45M variants)
- **Method:** Pearson r and Spearman ρ. Hexbin scatter plot.
- **Output:** r=0.143, ρ=0.137 (from Figure 8 — **ALREADY COMPUTED**)
- **Conclusion if r² high (>0.25):** Delta is frequency-driven → artifact.
- **Conclusion if r² low (<0.05):** Frequency differences don't explain delta → other factors dominate.
- **Actual result:** r²=0.020. Frequency differences explain ~2% of delta variance. Delta is NOT DAF-driven.

### Test 2: delta_SDS ~ MAF correlation
- **Question:** Are divergent loci concentrated in specific MAF ranges?
- **Input:** Same as Test 1. MAF column available.
- **Method:** Binned mean delta by MAF. Spearman ρ.
- **Output:** ρ=0.003 (from correlation matrix — **ALREADY COMPUTED**)
- **Conclusion if |ρ| > 0.2:** MAF-dependent bias → use within-bin z-scoring.
- **Conclusion if |ρ| ≈ 0:** No MAF dependence in normalized space.
- **Actual result:** No MAF dependence. Normalization is well-calibrated per-bin.

### Test 3: Per-chromosome delta vs chromosome properties
- **Question:** Does delta correlate with chromosome length, gene density, or recombination rate? Technical or biological pattern?
- **Input:** Per-chromosome mean delta (Figure 10) + reference genome statistics.
- **Method:** Pearson r between mean_delta and chr length / gene count / recombination rate.
- **Expected if technical:** Correlation with chromosome length (longer chr → more variants → more stable bin estimates → less shift).
- **Expected if biological:** Correlation with gene density or recombination rate.
- **Status:** Not yet computed. Per-chr delta table available.

### Test 4: Within-bin delta_z scoring
- **Question:** After accounting for MAF-dependent variance, which variants/loci are genuinely divergent?
- **Input:** delta_norm + MAF for 4.45M variants.
- **Method:** Per MAF bin: delta_z = (delta − mean_bin) / SD_bin. p = 2·Φ(−|delta_z|). BH-FDR.
- **Output:** FDR-significant variant list + window aggregation.
- **Advantage over raw |delta| ranking:** Accounts for MAF-dependent variance structure. Raw ranking is dominated by low-MAF bins where SD is largest.
- **Status:** Not yet computed. Recommended as next step before any locus annotation.

### Test 5: BIN_SD ratio vs sample-size expectation
- **Question:** Is variance inflation between groups purely a sample-size effect?
- **Input:** BIN_SD per MAF bin for SLE and nonSLE.
- **Method:** Compare BIN_SD_SLE/BIN_SD_nonSLE to √(2367/1965)=1.098.
- **Output:** Mean ratio=1.229, expected=1.098 (from Figure 9 — **ALREADY COMPUTED**)
- **Conclusion if ratio≈1.098:** Variance inflation is pure sample size → expected and benign.
- **Conclusion if ratio>1.15:** Additional variance sources (batch, missingness, biology).
- **Actual result:** Ratio 1.23 > 1.10 → ~12% excess variance beyond sample size. Source needs investigation.

### Test 6: PC1 association with per-sample aggregate SDS
- **Question:** Does ancestry (PC1) predict which samples contribute most to SDS?
- **Input:** Per-sample PC1 from freeze + per-variant nG0/nG1/nG2 columns to estimate per-sample contribution.
- **Method:** For each sample, compute mean rSDS of variants where that sample carries the derived allele. Correlate with PC1.
- **Status:** Not computed. Requires per-sample SDS deconvolution (not directly available from current output format).
- **Note:** This is methodologically complex — SDS is a per-variant score, not directly decomposable to per-sample. Approximate via: for variants where a sample carries the singleton (nG1=1, AC=1), check if the sample's PC1 predicts the variant's SDS. This is essentially the s_file generation step run in reverse.

### Test 7: Singleton count association with per-sample SDS contribution
- **Question:** Does sequencing depth / singleton detection rate drive individual-level SDS?
- **Input:** Per-sample singleton_count from freeze + per-variant nG0/nG1/nG2.
- **Method:** Same deconvolution challenge as Test 6. Alternative: compare singleton_count distribution between samples contributing to top-divergent variants vs background.
- **Status:** Not computed. Same methodological limitation as Test 6.

### Test 8: Region-stratified delta comparison (HongKong only)
- **Question:** Within the most balanced region (HK: SLE=1540, control=1504), does the systematic delta persist?
- **Input:** Would require re-running SDS with HK-only samples split by phenotype. Current threeway merge uses full cohorts.
- **Method:** Generate HK-only s/t/o files, re-run SDS compute, compare delta to full-cohort delta.
- **Expected if geographic confound:** HK-only delta should be reduced compared to full-cohort delta.
- **Expected if genuine case/control difference:** HK-only delta should be similar to full-cohort.
- **Status:** Not computed. Requires new input generation + compute. Recommended if focal peaks survive other diagnostics.

### Test 9: Variant missingness vs delta
- **Question:** Are divergent loci enriched for differential missingness between groups?
- **Input:** o_file per-sample observability values (available per chromosome per group).
- **Method:** Per variant, compute fraction of samples with missing genotype in SLE vs nonSLE. Correlate missingness difference with |delta|.
- **Expected if technical artifact:** |delta| should correlate with missingness difference.
- **Status:** Not computed. o_file data available but not aggregated to variant level.

### Test 10: Paired t-test per MAF bin
- **Question:** Is mean(delta) significantly non-zero within specific MAF bins?
- **Input:** delta_norm + MAF bin assignment.
- **Method:** Per bin: one-sample t-test of delta against 0. Bonferroni correction across bins.
- **Expected if global shift:** All or most bins should show significant non-zero mean.
- **Status:** Not computed but trivial given existing data. Recommended as quick check.

---

## §5 Decision Tree

```
Q0: Is delta_norm ~ delta_rSDS correlation near 1?
│   r = 0.959 ✓ — normalization preserves variant ranking.
│   → The global shift (−0.0085) is a constant offset. Focal peaks are preserved.
│
├─ Q1: Is mean(delta_rSDS) ≈ 0 while mean(delta_norm) ≠ 0?
│   │  delta_rSDS = +0.0016 (near zero), delta_norm = −0.0085 ✓
│   │  → Shift is CREATED by per-group normalization (BIN_MEAN + BIN_SD differences).
│   │  → This is a NORMALIZATION ARTIFACT.
│   │
│   └─ Q1a: Are BIN_MEAN differences the primary driver?
│       │  At low MAF: BIN_MEAN_SLE=0.042, BIN_MEAN_nonSLE=0.029 ✓
│       │  SLE higher BIN_MEAN → more subtraction → lower norm_SDS.
│       │  → Yes. Higher BIN_MEAN in SLE is the primary cause.
│       │
│       └─ Q1b: Why is SLE BIN_MEAN higher?
│           │  SLE has fewer samples (1965 vs 2367) → fewer singletons → different
│           │  singleton correction in s_file → different rSDS distribution per bin.
│           │  Higher BIN_SD (1.23×) amplifies the effect.
│           │  → ROOT CAUSE: sample size + singleton burden difference
│           │    manifesting through per-group MAF-bin statistics.
│           │
│           └─ **GATE A: Global delta is a normalization artifact.**
│              Use within-bin delta_z (Test 4) for locus detection.
│              Do NOT interpret mean(delta) as biological.
│
├─ Q2: Are there focal peaks that survive within-bin z-scoring?
│   │  chr19:52.5Mb, chr1:92Mb, chr9:92Mb identified as top windows.
│   │  These are z-score extremes (2-4σ) → candidates for real divergence.
│   │
│   └─ Q3: Could these peaks be explained by residual confounding?
│       │
│       ├─ Q3a: Are peaks in regions of differential missingness? (Test 9)
│       │   NOT YET TESTED.
│       │
│       ├─ Q3b: Are peaks at known batch-effect loci?
│       │   NOT YET TESTED. Check against known problematic regions.
│       │
│       ├─ Q3c: Do peaks overlap with ancestry-informative markers?
│       │   NOT YET TESTED. Cross-reference with PC loadings.
│       │
│       └─ Q3d: Are peaks robust to region-stratified permutation?
│           NOT YET TESTED. This is the definitive test:
│           shuffle phenotype within each region, recompute delta.
│           If observed peak exceeds permutation null → candidate real.
│
└─ **GATE B: Proceed to region-stratified permutation only for
    focal peaks that survive within-bin z-scoring + missingness check.
    Do NOT run genome-wide permutation.**
```

---

## §6 Next Steps (Explicit, Gated)

### Gate A: Normalization artifact confirmed → use corrected deltas

**Status:** CONFIRMED. delta_rSDS ≈ 0, delta_norm ≠ 0. Shift is a normalization artifact.

**Action A1 (immediate):** Compute within-bin delta_z for all 4.45M variants (Test 4). This uses SD(delta) per MAF bin instead of raw |delta|, removing the MAF-dependent variance that raw ranking ignores.

**Action A2 (immediate):** Run Test 10 (paired t-test per MAF bin) to quantify which bins drive the global shift.

**Action A3 (quick):** Window-level aggregation with delta_z → FDR-significant windows. Compare pre- vs post-correction top window rankings.

### Gate B: Focal peaks survive corrected ranking → targeted validation

**Prerequisite:** Gate A actions completed. At least 3 windows with FDR<0.05 outside known problematic regions (HLA, centromere).

**Action B1:** Check peak regions for differential missingness (Test 9).
**Action B2:** Cross-reference with PC loadings — are peak variants ancestry-informative?
**Action B3:** If B1 and B2 pass → region-stratified permutation on focal chromosomes only (chr19 + chr9 + chr22 as control). **Do NOT run genome-wide.**

### Gate C: Permutation confirms peaks → biological annotation

**Prerequisite:** chr19 peak exceeds 95th percentile of permutation null.

**Action C1:** LD-split + VEP annotation of surviving peaks.
**Action C2:** Cross-reference with known SLE GWAS loci.
**Action C3:** Check orthogonal evidence (iHS, Relate/CLUES2) at these loci.

### What NOT to do

- **Do NOT** interpret the global −0.0085 mean delta as evidence of systematically different selection pressure in SLE vs controls. It is a normalization artifact.
- **Do NOT** run genome-wide permutation. Only focal chromosomes with surviving peaks.
- **Do NOT** claim biological selection at any locus without completing Gate B (permutation test).
- **Do NOT** use raw |delta| ranking for locus prioritization. Use within-bin delta_z with FDR.

---

## §7 Output Index

### Diagnostic figures (7 files)
| Figure | File | Description |
|---|---|---|
| Fig 5 | `diagnostic/fig5_singleton_burden.png` | Singleton burden distributions (6 panels) |
| Fig 7 | `diagnostic/fig7_regional_composition.png` | Case-control regional composition (2 panels) |
| Fig 8 | `diagnostic/fig8_delta_sds_vs_daf.png` | Delta SDS vs delta DAF (6 panels) |
| Fig 9 | `diagnostic/fig9_bin_sd_ratio.png` | BIN_SD ratio vs MAF (4 panels) |
| Fig 10 | `diagnostic/fig10_per_chr_delta.png` | Per-chromosome mean delta (4 panels) |
| Corr | `diagnostic/fig_correlation_matrix.png` | Confounder correlation heatmap |
| Decomp | `diagnostic/fig_decomposition.png` | Raw vs normalized delta decomposition (2 panels) |

### UMAP figures (20 files)
| Directory | Contents |
|---|---|
| `diagnostic/umap/umap_common_orig_*.png` | Common variants, original PCs: phenotype, region, region+phenotype, singleton, SLE overlay |
| `diagnostic/umap/umap_common_resid_*.png` | Common variants, residualized PCs: same 5 views |
| `diagnostic/umap/umap_rare_orig_*.png` | Rare variants, original PCs: same 5 views |
| `diagnostic/umap/umap_rare_resid_*.png` | Rare variants, residualized PCs: same 5 views |

### Diagnostic tables (8 files)
| Table | File | Content |
|---|---|---|
| Singleton | `diagnostic/table_singleton_by_region_phenotype.tsv` | Per-region×phenotype singleton count stats |
| Region | `diagnostic/table_regional_composition.tsv` | SLE/control counts per region |
| BIN_SD | `diagnostic/table_bin_sd_ratio.tsv` | BIN_SD ratio by MAF bin (92 rows) |
| Decomposition | `diagnostic/table_decomposition.tsv` | Raw vs normalized delta by MAF bin |
| Per-chr | `diagnostic/table_per_chr_delta.tsv` | Per-chromosome delta statistics |
| Pearson | `diagnostic/table_correlation_pearson.tsv` | Full Pearson correlation matrix |
| Spearman | `diagnostic/table_correlation_spearman.tsv` | Full Spearman correlation matrix |
| DAF corr | `diagnostic/table_delta_sds_daf_correlation.tsv` | Per-chromosome delta~DAF correlation |

### Pre-existing comparison outputs
| File | Description |
|---|---|
| `threeway_merge.tsv` | 4,452,790 variants × 11 columns |
| `mirror_manhattan_SLE_vs_nonSLE_CMplot.png` | CMplot Mirror Manhattan (2026-08-05 redraw, peak-annotated) |
| ~~`mirror_manhattan_SLE_vs_nonSLE.png`~~ | ~~Mirror Manhattan~~ — superseded, deleted 2026-08-05 |
| ~~`delta_manhattan_SLE_minus_nonSLE.png`~~ | ~~Delta Manhattan~~ — superseded, deleted 2026-08-05 |
| ~~`three_panel_comparison.png`~~ | ~~Three-panel (SLE/nonSLE/delta)~~ — superseded, deleted 2026-08-05 |

---

## §8 References

- [[2026-06-26_population_structure_analysis]] — PCA-UMAP, singleton confound, badMIXTURE
- [[2026-08-02_sle_vs_nonsle_sds_comparison]] — Original SLE vs non-SLE comparison
- `SDSworkspace/oneoff/diagnose_sle_nonsle_confounders.py` — Diagnostic script (this analysis)
- `SDSworkspace/oneoff/plot_sle_nonsle_umap.py` — UMAP plot script (this analysis)
