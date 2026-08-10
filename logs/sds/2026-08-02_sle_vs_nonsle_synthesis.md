# 2026-08-02 SLE vs non-SLE SDS: Synthesis & Interpretation

**Role:** `audit-only` — synthesis of all diagnostic analyses. No new computation.  
**Parent entries:**
- [2026-08-02_sle_vs_nonsle_sds_comparison.md](2026-08-02_sle_vs_nonsle_sds_comparison.md) — original comparison
- [2026-08-02_sle_vs_nonsle_confounder_diagnostic.md](2026-08-02_sle_vs_nonsle_confounder_diagnostic.md) — confounder diagnostic

---

## §1 Five Established Facts

### Fact 1: Singleton burden is strongly region-dependent

Singleton count varies **2.2×** across regions (Shanghai 431 → Jinan 958). Region alone explains **28.6%** of singleton variance (Model M1: `singleton ~ region`, R²=0.286). This variation is **not** explained by sequencing depth (DP varies only 1.15×, Ti/Tv stable at 1.41 across all regions).

| Region | N | Mean singleton | Mean DP | Ti/Tv |
|---|---|---|---|---|
| Shanghai | 95 | 431 | 54.9 | 1.406 |
| HongKong | 3,052 | 597 | 57.1 | 1.410 |
| Shenzhen | 267 | 616 | 52.9 | 1.406 |
| JiNing | 391 | 949 | 49.5 | 1.410 |
| Jinan | 171 | 958 | 52.3 | 1.422 |

**Interpretation:** The 2.2× regional singleton variation is a batch/cohort effect (library preparation, coverage uniformity, or sample ascertainment differences between sequencing sites), not a depth or quality artifact.

### Fact 2: Ignoring interaction, SLE shows lower singleton burden

Model M2 (`singleton ~ region + phenotype`): β_pheno = **−44.1** (p = 3.4×10⁻¹⁴). After controlling for region, SLE cases have ~44 fewer singletons than controls. However, phenotype adds only **ΔR² = +0.009** beyond region — the effect is statistically significant but explains <1% of singleton variance.

### Fact 3: Region × phenotype interaction is highly significant

ANOVA comparing `~region+phenotype` vs `~region*phenotype`:

| Model | R² | RSS | df |
|---|---|---|---|
| Without interaction | 0.2781 | 122,043,754 | 3800 |
| With interaction | **0.3346** | 112,505,035 | 3797 |

**F(3, 3797) = 107.31, p ≈ 0.** ΔR² = +5.6% — interaction explains more variance than the phenotype main effect alone.

### Fact 4: Shenzhen and HongKong show opposite SLE effects

Per-region SLE−control singleton difference from the interaction model:

| Region | Δ (SLE−control) | Direction | p |
|---|---|---|---|
| **Shenzhen** | **+375.0** | SLE >> control | <0.0001 |
| HongKong | −48.8 | SLE < control | <0.0001 |
| Shanghai | −155.1 | SLE < control | <0.0001 |
| JiNing | +3.5 | No difference | 0.57 |

Shenzhen's interaction coefficient = +423.7 — it overwhelms the HongKong main effect (−48.7) and is the primary reason the no-interaction model gives a misleading single β_pheno of −44.

Shenzhen SLE samples (n=187) have mean singleton=728 at DP=54.2; Shenzhen controls (n=80) have mean singleton=353 at DP=49.8. The control group's anomalously low singleton count (353 — lowest of any group in the entire cohort, even below Shanghai's 431) drives the extreme +375 delta.

### Fact 5: No consistent SLE-associated singleton burden exists across all regions

The direction of SLE−control singleton difference varies by region (Shenzhen +375, HongKong −49, JiNing +3.5). There is no single "SLE effect" on singleton burden. The apparent overall SLE deficit (β=−44) is a statistical artifact of averaging over regions with opposite-direction effects, weighted by sample size (HK dominates at n=3,044).

**Caveat:** Shanghai has only 7 SLE samples — the −155 delta is directionally consistent with HK but unstable. Shenzhen SLE/control ratio is 187/80 — control n is small relative to SLE.

---

## §2 What This Means for the SLE vs non-SLE SDS Comparison

### 2.1 The global delta (−0.0085) is a normalization artifact

From the confounder diagnostic: raw rSDS delta = **+0.0016** (near zero, slightly positive), but normalized delta = **−0.0085** (negative). The shift direction **flips** during per-group MAF-bin normalization. SLE has higher BIN_MEAN (e.g., 0.042 vs 0.029 at MAF 0.05–0.07) and higher BIN_SD (1.23×) — both push SLE norm_SDS downward relative to non-SLE even when raw rSDS is identical.

**Conclusion:** The genome-wide negative delta does NOT reflect systematically weaker selection in SLE. It is a normalization artifact caused by per-group bin statistics differing between SLE (N=1965) and non-SLE (N=2367).

### 2.2 Singleton burden does not provide a simple confound explanation

Initial hypothesis: SLE cases have fewer singletons → lower SDS signal → negative delta.

Reality is more complex:
- Region explains 29% of singleton variance; phenotype explains <1%
- Shenzhen shows SLE singletons 375 **higher** than controls — opposite of the HK pattern
- If singleton burden directly drove SDS, we would expect Shenzhen SLE to have **higher** SDS than SZ controls. But the genome-wide delta is negative everywhere.

The singleton → SDS pathway is not straightforward. Singleton burden affects s_file composition, which affects per-group BIN_MEAN/BIN_SD, which affects normalization. But the effect is mediated through the **entire group's** singleton distribution, not individual samples.

### 2.3 Geographic confounding is the dominant structural problem

SLE cases are **78% HongKong** (1540/1965). Controls are spread across 6 regions with different singleton profiles. Geographic confounding operates at two levels:

1. **Global shift**: SLE group composition (mostly HK, singleton ~597) vs non-SLE composition (mixed, singleton ~682) → different per-group bin statistics → normalization artifact.

2. **Focal peaks**: A variant showing high delta could reflect (a) genuine case/control frequency difference, (b) regional frequency difference between HK and other regions, or (c) both. Since SLE is concentrated in HK, (a) and (b) are confounded at almost every locus.

### 2.4 Focal peaks (chr19:52.5Mb, chr9:92Mb) remain uninterpreted

The normalization artifact explains the global mean shift but does NOT explain focal extreme windows. The chr19:52.5Mb window (delta=−1.07, n=11 variants) and chr9:92Mb (delta=+0.25) are z-score extremes (|delta| > 4σ) that survive the global shift. However:

- **Not yet tested**: whether these peaks are enriched for differential missingness between groups (Test 9 from diagnostic)
- **Not yet tested**: whether peaks overlap with ancestry-informative markers (PC loadings)
- **Not yet tested**: whether peaks survive within-bin delta_z scoring with FDR control (Test 4)
- **Not yet tested**: whether peaks would survive region-stratified phenotype permutation

These peaks should NOT be interpreted as biological selection signals without passing the above checks.

---

## §3 Causal Diagram

```
Sequencing site (GZ/SZ/HK/SH/JN/JiN)
    │
    ├──→ singleton burden (2.2× variation, R²=0.29)
    │       │
    │       └──→ per-group BIN_MEAN/BIN_SD
    │               │
    │               └──→ normalization artifact → global delta ≠ 0
    │
    ├──→ sample ancestry (PC1, South-North cline)
    │       │
    │       └──→ allele frequencies at ancestry-informative loci
    │               │
    │               └──→ possible focal delta peaks (NOT YET TESTED)
    │
    └──→ phenotype composition (SLE% varies 0%–70% by site)
            │
            ├──→ SLE group = 78% HK (singleton ~597)
            ├──→ non-SLE group = mixed regions (singleton ~682)
            │
            └──→ geographic confounding of any "SLE effect"
                    │
                    └──→ region × phenotype interaction (F=107, p≈0)
                            No consistent SLE singleton burden exists
```

---

## §4 What Can and Cannot Be Claimed

### CAN be claimed (evidence-supported)

1. **The genome-wide delta_SDS (−0.0085) is a normalization artifact.** Raw rSDS difference between SLE and non-SLE is negligible (+0.0016). The negative shift emerges from per-group MAF-bin statistics differing between groups of different sizes and singleton compositions.

2. **Singleton burden varies 2.2× by region** and this variation is not explained by sequencing depth (DP 1.15×) or variant quality (Ti/Tv 1.01×). It is a batch/cohort effect.

3. **There is no consistent SLE-associated singleton burden.** The SLE−control singleton difference varies by region: Shenzhen +375 (SLE higher), HongKong −49 (SLE lower), JiNing +3.5 (no difference). The region × phenotype interaction is highly significant (F=107, p≈0).

4. **SLE cases are not a genetically distinct subpopulation.** PCA-UMAP shows SLE cases embedded within the broader population structure. The apparent SLE-control separation on PC1 (d=0.428) is driven by geographic concentration (78% HK), not a distinct genetic architecture.

5. **DP and sequencing platform do not explain the singleton-phenotype relationship.** Adding DP to the regression model does not change β_pheno; inferred platform (Illumina vs BGI within HK) is not a significant predictor of singleton count.

### CANNOT be claimed (insufficient evidence)

1. **"SLE patients experience systematically different recent selection pressure."** The global delta is an artifact. Focal peaks have not been validated against confounders.

2. **"chr19:52.5Mb is a SLE-specific selection signal."** This peak has not been tested for differential missingness, ancestry informativeness, or survival under region-stratified permutation.

3. **"SLE cases have lower singleton burden due to disease biology."** The region × phenotype interaction shows this is not a consistent effect. Shenzhen contradicts it directly.

4. **"Correcting for singleton count will resolve the SDS differences."** Singleton burden is regionally structured and interacts with phenotype. A simple linear correction would be misspecified.

---

## §5 Decision Tree (Updated)

```
Q1: Is the genome-wide delta biologically meaningful?
└─ NO. delta_rSDS ≈ 0, delta_norm = −0.0085. Normalization artifact.
   → Do NOT interpret mean(delta). Use within-bin delta_z for locus detection.

Q2: Is there a consistent SLE singleton burden effect?
└─ NO. Region × phenotype interaction F=107, p≈0.
   Shenzhen +375, HongKong −49 — opposite directions.
   → Do NOT use a single β_pheno adjustment.

Q3: Do any focal peaks survive basic confounder checks?
├─ chr19:52.5Mb (delta=−1.07): NOT YET CHECKED
│   Need: missingness test, PC-loading overlap, within-bin delta_z
├─ chr9:92Mb (delta=+0.25): NOT YET CHECKED
└─ chr1:92Mb (delta=−0.86): NOT YET CHECKED
   → Gate B: only proceed to annotation if peaks pass checks above.

Q4: What WOULD establish a real case/control difference?
└─ Region-stratified permutation test on focal chromosomes only.
   Shuffle phenotype within each region, recompute delta, build null.
   If observed peak exceeds 95th percentile → candidate real.
   This is the ONLY test that properly controls for the region×phenotype confound.
```

---

## §6 Recommendations

### Immediate (no new compute)

1. **Within-bin delta_z scoring** (Test 4 from diagnostic): use SD(delta) per MAF bin instead of raw |delta|. This handles MAF-dependent variance and provides FDR-controlled significant variant lists. No new compute needed — works on existing threeway_merge data.

2. **Delta_z-corrected Manhattan plot**: regenerate delta Manhattan using delta_z instead of raw delta. This will show whether chr19:52.5Mb survives MAF-aware variance scaling.

### Short-term (targeted compute)

3. **Missingness check at focal peaks** (Test 9): for chr19:52.5Mb, chr9:92Mb, chr1:92Mb, compare per-variant missingness between SLE and non-SLE groups. If peaks overlap regions of differential missingness → downgrade to artifact.

4. **PC-loading overlap**: check whether top-divergent variants overlap with known ancestry-informative markers (high PC-loading SNPs). If yes → geographic confound, not selection.

### If peaks survive checks 1-4

5. **Region-stratified permutation on chr19 + chr9 only** (NOT genome-wide). This is the definitive test. Shuffle SLE/control labels within each region, recompute delta for the focal chromosomes, build null distribution.

### Do NOT do

- Do NOT run genome-wide permutation (waste of compute)
- Do NOT interpret global delta as biological
- Do NOT apply a single β_pheno correction for singleton burden
- Do NOT claim biological selection at any locus without completing steps 1-5
- Do NOT use the existing top_divergent_windows.tsv for biological interpretation (raw |delta| ranking, no confounder checks)

---

## §7 Output Index

### Comparison outputs (pre-existing)
| File | Description |
|---|---|
| `threeway_merge.tsv` | 4,452,790 variants, 11 columns |
| `mirror_manhattan_SLE_vs_nonSLE_CMplot.png` | CMplot Mirror Manhattan (2026-08-05 redraw; SLE top / nonSLE bottom, peak annotations: shared black / SLE-only red / nonSLE-only blue) |
| ~~`mirror_manhattan_SLE_vs_nonSLE.png`~~ | ~~Mirror Manhattan~~ — superseded, deleted 2026-08-05 |
| ~~`delta_manhattan_SLE_minus_nonSLE.png`~~ | ~~Delta Manhattan~~ — superseded, deleted 2026-08-05 |

### Diagnostic figures (7 files)
| File | Description |
|---|---|
| `diagnostic/fig5_singleton_burden.png` | Singleton burden by phenotype × region |
| `diagnostic/fig7_regional_composition.png` | SLE/control regional composition |
| `diagnostic/fig8_delta_sds_vs_daf.png` | Delta SDS vs delta DAF |
| `diagnostic/fig9_bin_sd_ratio.png` | BIN_SD ratio vs MAF |
| `diagnostic/fig10_per_chr_delta.png` | Per-chromosome delta |
| `diagnostic/fig_correlation_matrix.png` | Confounder correlations |
| `diagnostic/fig_decomposition.png` | Raw vs normalized delta decomposition |

### UMAP figures (20 files)
| Directory | Contents |
|---|---|
| `diagnostic/umap/umap_common_orig_*.png` | Common variants, original PCs: 5 views |
| `diagnostic/umap/umap_common_resid_*.png` | Common variants, residualized PCs: 5 views |
| `diagnostic/umap/umap_rare_orig_*.png` | Rare variants, original PCs: 5 views |
| `diagnostic/umap/umap_rare_resid_*.png` | Rare variants, residualized PCs: 5 views |

### Regression outputs (new)
| File | Description |
|---|---|
| `diagnostic/fig_forest_plot.png` | Forest plot: per-region SLE−control singleton Δ |
| `diagnostic/fig_singleton_regression_diagnostics.png` | 4-panel regression diagnostic |
| `diagnostic/table_singleton_regression_models.tsv` | M1-M4 coefficients, SE, p, R² |
| `diagnostic/table_singleton_region_delta.tsv` | Per-region Δ_i, SE, CI |

### Key tables
| File | Description |
|---|---|
| `diagnostic/table_superpop_region_stats_full.tsv` | Per-region singleton, DP, Ti/Tv, F_MISS, F_het |
| `diagnostic/table_decomposition.tsv` | Raw vs normalized delta by MAF bin |
| `diagnostic/table_bin_sd_ratio.tsv` | BIN_SD ratio vs MAF |

### Scripts
| File | Description |
|---|---|
| `SDSworkspace/oneoff/diagnose_sle_nonsle_confounders.py` | Diagnostic figures + tables |
| `SDSworkspace/oneoff/plot_sle_nonsle_umap.py` | UMAP plots |
| `SDSworkspace/oneoff/model_singleton_regression.py` | Regression models + forest plot |
| `SDSworkspace/oneoff/compute_titv_dp_stream.py` | Ti/Tv+DP streaming computation |
| `SDSpipeline/scripts/preprocessing/CalculateTiTvAndDPForSample.cpp` | C++ Ti/Tv+DP (compiled) |

---

## §8 References

- [[2026-06-26_population_structure_analysis]] — PCA-UMAP, singleton confound, PC1 structure, badMIXTURE
- [[2026-08-02_sle_vs_nonsle_sds_comparison]] — Original SLE vs non-SLE SDS comparison
- [[2026-08-02_sle_vs_nonsle_confounder_diagnostic]] — 8-confounder diagnostic + decision tree
- `SDSworkspace/manifests/cohort_freeze_sds.tsv` — Cohort freeze with singleton_count, F_MISS, F_het, Region, Phenotype
- `SDSworkspace/results/production/sds_output/SLE/` — SLE SDS outputs
- `SDSworkspace/results/production/sds_output/non-SLE/` — non-SLE SDS outputs
- `raw/9k/all.chr22.QC.vcf.gz` — Raw joint-called VCF (source of DP, Ti/Tv)
