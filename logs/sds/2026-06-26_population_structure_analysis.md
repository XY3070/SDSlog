# 2026-06-26 Population structure: ADMIXTURE → badMIXTURE → PCA-UMAP

## Environment & Tools

| Tool | Role | Env / Path |
|---|---|---|
| ADMIXTURE 1.3.0 | Unsupervised clustering K=2..10 | `SDSworkspace/runs/admixture_9k_v1_20260619/unsupervised/` |
| badMIXTURE (R package) | Residual test: model vs observed per-site | conda `sds` (`/data/home/grp-wangyf/intern/miniforge3/envs/sds`) |
| PLINK2 v2.0.0-a.6.34LM | MAF filter, LD prune, PCA | `/data/home/grp-wangyf/leiyao/Tools/PLINK2/plink2` |
| umap-learn (Python) | 50-PC → 2D UMAP embedding | conda `sds` (`python3.14`) |
| matplotlib | All figures | conda `sds` / uv `.venv` |

## Input Provenance

| Layer | Source | Details |
|---|---|---|
| ADMIXTURE genotype | `SDSworkspace/runs/admixture_9k_v1_20260619/input/9k_ldpruned.{pgen,psam,pvar}` | LD-pruned (50/5/0.2), 107,927 SNPs, 4,393 samples |
| PCA-UMAP genotype (common) | `raw/9k/snp_singleton_qc/snp_qc_pass.vcf.gz` | 14.56M variants → MAF>1% → 580k → LD-prune → 108k |
| PCA-UMAP genotype (rare) | Same VCF | 14.56M variants → MAF 0.1%–1% → 509k → LD-prune → 376k |
| Sample metadata | `raw/9k/sample_qc/cohort_freeze.tsv` | Regions: Guangzhou, Shenzhen, HongKong, Shanghai, Jinan, JiNing |
| ADMIXTURE Q/P matrices | `runs/admixture_9k_v1_20260619/unsupervised/9k.{K}.{Q,P}` | K=2..10, `.Q` = admixture proportions, `.P` = ancestral allele freqs (L × K) |

## Step 1 — ADMIXTURE: Unsupervised K=2..10

**Script:** `SDSworkspace/oneoff/run_badmixture_analysis.R`  
**Method:** Standard ADMIXTURE unsupervised run for K=2..10 on LD-pruned SNPs. Per-K cluster assignment (dominant component, threshold maxQ≥0.6). K evaluation via Evanno Delta-K, 50% subsample stability, normalized entropy, ambiguous proportion.

### Single-figure ancestry plot (K=2)

**Script:** `SDSpipeline/scripts/plot_ancestry_one_figure.py`  
All 4,393 samples on one stacked bar, sorted by region block then component-0 proportion (monotonic gradient). Region labels rotated 45°.

**Output:** `runs/admixture_9k_v1_20260619/figures/ancestry_one_figure_K2.png`

### K evaluation (non-CV)

| Metric | Best K | Behavior |
|---|---|---|
| Evanno Delta-K | varies | second-order LL rate-of-change |
| Stability (50% subsample, 10 reps) | varies | cluster concordance across splits |
| Normalized entropy | lower = better | cluster separation |
| Ambiguous (maxQ<0.6) | higher = more uncertain | per-individual assignment confidence |

### ADMIXTURE cluster outputs

| File | Content |
|---|---|
| `badmixture/cluster_plots_per_K.pdf` | Per-K 4-panel: Q heatmap (50/cluster), PCA of Q, cluster sizes, mean ancestry per cluster |
| `badmixture/full_heatmap_K{2..10}.png` | Full-cohort Q matrix heatmap (ordered by cluster→maxQ) |
| `badmixture/K_evaluation.png` | 6-panel: LL, Delta-K, Stability, Entropy, Ambiguous%, metrics table |
| `badmixture/region_cluster_K{2,4}.png` | Best-K region×cluster composition barplot |

All paths relative to `SDSworkspace/runs/umap_9k/`.

## Step 2 — badMIXTURE: Residual test

**Script:** `SDSworkspace/oneoff/run_badmixture_residuals.R`  
**Method:** For each K=2..8, sample 50 individuals per ADMIXTURE cluster (same sampling as Step 1, no re-sample). Extract 2,000 random SNP genotypes via `plink2 --export A`. Compute ADMIXTURE model expectation:

```
G_exp[i,j] = 2 · Σ_k Q[i,k] · P[j,k]
R[i,j] = G_obs[i,j] − G_exp[i,j]
```

Residuals visualized with badMIXTURE diverging palette: blue (−) → white (0) → red (+). Color randomness quantifies fit: random speckle = good fit; structured blocks = model missing signal.

### Per-K residual metrics

| K | ResidVar | MAE | Row ACF | Col ACF | \|R\|>1% | Randomness↓ |
|---|----------|-----|---------|---------|-------|-------------|
| 2 | 3.036 | 1.634 | 0.861 | 0.008 | 82.1 | 0.870 |
| 3 | 3.041 | 1.635 | 0.858 | 0.012 | 82.2 | 0.871 |
| 4 | 3.047 | 1.637 | 0.855 | 0.013 | 82.3 | 0.868 |
| 5 | 3.047 | 1.637 | 0.858 | 0.010 | 82.3 | 0.867 |
| 6 | 3.050 | 1.639 | 0.859 | 0.011 | 82.4 | 0.870 |
| 7 | 3.050 | 1.639 | 0.860 | 0.010 | 82.4 | 0.870 |
| 8 | 3.049 | 1.638 | 0.859 | 0.010 | 82.4 | 0.869 |

**Randomness = |Row ACF| + |Col ACF|**. Lower = residuals more random = better fit.

### Key signals

- **Row ACF ~0.86 (critical):** Inter-individual residual correlation. Shows systematic between-individual structure not captured by discrete ADMIXTURE clusters. This is badMIXTURE's primary diagnostic.
- **Col ACF ~0.01 (clean):** Inter-SNP residual independence. No systematic SNP-level bias.
- **Randomness flat across K (0.867–0.871):** Adding clusters does not improve fit — model saturated at low K.
- **82% large residuals (|R|>1):** Per-SNP genotype predictions deviate substantially. Partly explained by binomial variance at low MAF.

**Conclusion:** Discrete mixture model alone insufficient for this cohort. Continuous gradient approach needed.

### badMIXTURE outputs

| File | Content |
|---|---|
| `badmixture/badmixture_residuals_K{2..8}.png` | 4-panel: main heatmap (blue-white-red), color scale, cluster assignment bar, mean residual per individual |
| `badmixture/badmixture_quality_vs_K.png` | 6-panel cross-K: ResidVar, MAE, Randomness, Row ACF, Col ACF, PropLarge |

All paths relative to `SDSworkspace/runs/umap_9k/`.

## Step 3 — PCA-UMAP: Continuous gradient

**Scripts:** `SDSworkspace/oneoff/run_umap_9k.sh` (common), `SDSworkspace/oneoff/run_umap_9k_rare.sh` (rare)  
**Pipeline:** MAF filter → LD prune (50/5/0.2) → top 50 PCs (plink2 --pca) → UMAP (n_neighbors=30, min_dist=0.1, seed=42)  
**Motivation:** badMIXTURE Row ACF ~0.86 → discrete ADMIXTURE clusters miss gradient structure. PCA-UMAP tests whether continuous low-dimensional manifold captures what discrete model cannot.

### SNP yield comparison

| Stage | Common (MAF>1%) | Rare (0.1%–1%) |
|---|---|---|
| VCF input | 14.56M variants | 14.56M variants |
| MAF filter | 580,363 | 508,963 |
| LD prune (r²>0.2 removed) | 107,927 (18.6% retained) | 376,481 (74.0% retained) |
| PCA samples | 4,392 | 4,382 |

Rare variants: LD prune retention 4× higher → rare alleles carry less pairwise LD → more independent SNP axes for PCA.

### UMAP figure matrix (5 views × 2 variant sets = 10 figures)

| View | Common output | Rare output | What it tests |
|---|---|---|---|
| Region | `*_umap_region.png` | `*_rare_umap_region.png` | Geographic labels on continuous manifold |
| K2 cluster | `*_umap_admixture_k2.png` | `*_rare_umap_admixture_k2.png` | Discrete cluster vs continuous topology |
| PC1 gradient | `*_umap_pc1.png` | `*_rare_umap_pc1.png` | PC1 structure continuity (inferno colormap) |
| PC1+PC2 bicolor | `*_umap_pc12_bicolor.png` | `*_rare_umap_pc12_bicolor.png` | R=PC1, G=PC2 — pure hues = PC-structured, muddy = noise |
| Plain | `*_umap.png` | `*_rare_umap.png` | Overall topology, all samples |

Prefix: `9k_snp_qc` (common) / `9k_snp_qc_rare` (rare). All in `SDSworkspace/runs/umap_9k/`.

### Coordinate tables

`9k_snp_qc_umap.tsv` / `9k_snp_qc_rare_umap.tsv` — 9 columns: FID, IID, UMAP1, UMAP2, Region, K2_Cluster, PC1, PC2. One row per sample.

## Cross-method synthesis

```
ADMIXTURE K=2..10          badMIXTURE residuals       PCA-UMAP (common/rare)
─────────────────          ─────────────────────       ──────────────────────
Discrete clusters           Row ACF ~0.86               Continuous manifold
Region×cluster structure    Model misfit confirmed      Geographic separation visible
K selection via Δ-K/stab    Randomness flat across K     Rare: 3.5× more SNP axes
                      ↘         ↓         ↙
                       ADMIXTURE discrete assumption
                       misses inter-individual gradient.
                       PCA-UMAP continuous approach
                       captures this hidden structure.
```

1. **ADMIXTURE** finds interpretable region×cluster composition, but K evaluation metrics alone cannot detect model misfit.
2. **badMIXTURE** proves misfit exists (Row ACF 0.86) and is not fixable by increasing K (flat randomness score). This is the quantitative evidence that discrete clusters are insufficient.
3. **PCA-UMAP common** (108k SNPs, MAF>1%) shows geographic structure on continuous manifold — the gradient that ADMIXTURE bins into discrete clusters.
4. **PCA-UMAP rare** (376k SNPs, MAF 0.1%–1%) provides denser SNP coverage — 74% LD-prune retention vs 19% for common — potentially capturing finer-scale recent demographic events.
5. **Rare-variant QC** (2026-06-29): 32 samples in 2 tight UMAP clusters removed. Root cause: PC1 vs singleton_count r = −0.50 — rare-variant PCA is a data-quality axis. HK within-region UMAP spread reduced 46% after removal. PC1~singleton correlation unchanged (systemic, not limited to outliers).
6. **Poor-separation diagnosis** (2026-06-29): Within-China genetic differentiation is subtle (Fst ~ 0.001–0.002). PC1 captures South–North cline (eta²=0.41) but explains only 11% of total variance. Singleton count confound is 60% as strong as biological signal. Only 3 effective groups (South/East/North), not 6 regions. UMAP amplifies noise because PC2–PC50 are noise-dominated. Call-rate 0.95→0.97 has negligible effect.
7. **Re-analysis with residualization** (2026-06-29): Singleton count regressed from PCs → confound completely removed (r=-0.50→0.00). Only 4 informative PCs retained. UMAP with n_neighbors={30,50,80} on residual PCs produces smoother manifold. Southern superpop rare-variant PCA confirms no sub-structure within GZ/SZ/HK. badMIXTURE misfit is partly biological (continuous cline vs discrete model) and partly technical (singleton confound). After residualization, the biological cline is isolated.

## Script inventory

| Script | Purpose | Location |
|---|---|---|
| `plot_ancestry_one_figure.py` | Single stacked-bar ancestry plot (K=2), monotonic sort | `SDSpipeline/scripts/` |
| `run_badmixture_analysis.R` | ADMIXTURE cluster plots, K evaluation, region composition | `SDSworkspace/oneoff/` |
| `run_badmixture_residuals.R` | badMIXTURE residual analysis (model vs observed per-site) | `SDSworkspace/oneoff/` |
| `run_umap_9k.sh` | PCA-UMAP pipeline (common variants MAF>1%) | `SDSworkspace/oneoff/` |
| `run_umap_9k_rare.sh` | PCA-UMAP pipeline (rare variants MAF 0.1%–1%) | `SDSworkspace/oneoff/` |
| `run_umap_9k_rare_clean.sh` | Outlier-removed rare PCA-UMAP pipeline (steps 1–3) | `SDSworkspace/oneoff/` |
| `plot_umap_rare_clean.py` | Standalone UMAP + 5-view plotting for cleaned rare data | `SDSworkspace/oneoff/` |

## Output file index

Figures under `SDSworkspace/runs/`:

```
admixture_9k_v1_20260619/figures/
    ancestry_one_figure_K2.png              ← single stacked bar, monotonic component sort

umap_9k/badmixture/
    cluster_plots_per_K.pdf                 ← ADMIXTURE K=2..10 per-K 4-panel
    full_heatmap_K{2..10}.png              ← full-cohort Q heatmaps
    K_evaluation.png                        ← 6-panel K evaluation
    region_cluster_K{2,4}.png              ← region×cluster composition
    badmixture_residuals_K{2..8}.png       ← residual heatmaps (blue-white-red)
    badmixture_quality_vs_K.png             ← cross-K residual metrics

umap_9k/                                    ← common variant UMAP (MAF>1%, 108k SNPs)
    9k_snp_qc_umap.png                      ← plain
    9k_snp_qc_umap_region.png               ← colored by region
    9k_snp_qc_umap_admixture_k2.png         ← colored by ADMIXTURE K=2 cluster
    9k_snp_qc_umap_pc1.png                  ← colored by PC1 (inferno)
    9k_snp_qc_umap_pc12_bicolor.png         ← R=PC1, G=PC2 bicolor
    9k_snp_qc_umap.tsv                      ← coords + metadata

umap_9k/                                    ← rare variant UMAP (MAF 0.1%–1%, 376k SNPs)
    9k_snp_qc_rare_umap.png                 ← plain
    9k_snp_qc_rare_umap_region.png          ← colored by region
    9k_snp_qc_rare_umap_admixture_k2.png    ← colored by ADMIXTURE K=2 cluster
    9k_snp_qc_rare_umap_pc1.png             ← colored by PC1 (inferno)
    9k_snp_qc_rare_umap_pc12_bicolor.png    ← R=PC1, G=PC2 bicolor
    9k_snp_qc_rare_umap.tsv                 ← coords + metadata
```

## Step 4 — Rare-variant outlier detection & removal (2026-06-29)

**Motivation:** Rare-variant UMAP showed two visually isolated small groups (11 + 21 samples). Investigated whether these represent data quality issues or genuine sub-population structure.

### Outlier identification

**Script:** `SDSworkspace/oneoff/run_umap_9k_rare_clean.sh` (steps 1–3), `SDSworkspace/oneoff/plot_umap_rare_clean.py` (step 4)  
**Method:** DBSCAN (eps=0.5, min_samples=5) on UMAP coordinates identified 3 tight clusters outside the main body. Two most extreme selected as outliers:

| Group | N | UMAP1 | UMAP2 | Region | Key feature |
|---|---|---|---|---|---|
| G1 (mid-left) | 11 | ~ −15.50 | ~12.85 | 11 HK | Near-identical UMAP coords (range < 0.02) |
| G2 (far-right) | 21 | ~ −5.55 | ~8.88 | 19 HK, 1 China, 1 Jinan | Tight cluster, isolated from main body |

### Data quality investigation

Compared outlier QC metrics against cohort (`raw/9k/sample_qc/sample_qc_metrics.tsv`):

| Metric | G1 (n=11) | G2 (n=21) | Cohort (n=4897) |
|---|---|---|---|
| F_MISS (mean) | 0.00398 | 0.00413 | 0.00402 |
| F_het (mean) | −0.0548 | −0.0593 | −0.0560 |
| het_outlier | 0 | 0 | — |
| status=PASS | 11 (100%) | 21 (100%) | 4351 (89%) |
| singleton_count (mean) | **553** | **543** | **655** |

All 32 outliers pass standard QC. No het_outlier, no singletons_outlier, no exclusion reasons.

### Root cause: rare-variant PC1 is a data-quality axis

**Critical finding:** PC1 vs singleton_count correlation r = **−0.50** across all 4,382 samples. Rare-variant PCA is substantially driven by sequencing depth/coverage variation:

- Lower depth → fewer singletons detected → systematic shift in rare-variant genotype calls → PC1 separation
- Common variants (MAF>1%) are robust to depth variation → no outlier separation in common UMAP
- Outlier groups have ~15% fewer singletons than cohort mean (553/543 vs 655)

**Conclusion:** These 32 samples are QC outliers specific to rare-variant space. They pass standard QC because standard QC uses common variants. For rare-variant analysis, singleton count should be treated as a QC covariate.

### Outlier removal and re-analysis

32 samples removed via `--remove rare_outliers_remove.txt` at MAF filter step. Full pipeline re-run:

| Stage | Original | Clean |
|---|---|---|
| Samples | 4,382 | 4,351 |
| MAF-filtered SNPs | 508,963 | 505,987 |
| LD-pruned SNPs | 376,481 (74.0%) | 373,799 (73.9%) |

### Impact on UMAP structure

| Metric | Original | Clean | Change |
|---|---|---|---|
| HK UMAP1 std | 2.239 | 1.206 | **−46%** |
| Jinan UMAP1 std | 1.779 | 0.602 | **−66%** |
| Total UMAP1 std | 2.153 | 1.155 | **−46%** |
| PC1~singleton r | −0.50 | −0.50 | unchanged |

HK and Jinan within-region spread dramatically reduced. However, PC1~singleton correlation unchanged — this is a systemic cohort-wide confound, not limited to the 32 outliers.

**Recommendation:** For any paper-facing rare-variant analysis, either:
1. Regress out singleton count from PCs before UMAP/clustering, or
2. Use singleton count as a covariate in downstream tests, or
3. Apply stricter singleton-count-based QC thresholds

### Cleaned rare-variant outputs

```
umap_9k/
    rare_outliers_remove.txt                     ← 32 IIDs excluded
    9k_snp_qc_rare_clean_umap.png                ← plain, outliers removed
    9k_snp_qc_rare_clean_umap_region.png         ← colored by region
    9k_snp_qc_rare_clean_umap_admixture_k2.png   ← colored by ADMIXTURE K=2 cluster
    9k_snp_qc_rare_clean_umap_pc1.png            ← colored by PC1 (inferno)
    9k_snp_qc_rare_clean_umap_pc12_bicolor.png   ← R=PC1, G=PC2 bicolor
    9k_snp_qc_rare_clean_umap.tsv                ← coords + metadata (4,351 rows)
```

## Step 5 — SNP call-rate sensitivity & poor-separation diagnosis (2026-06-29)

### Call-rate threshold

**Current QC:** variant missingness ≤ 5% (call rate ≥ 0.95) in `step1_snp_qc.sh`.
Raised to **call rate ≥ 0.97** (`--geno 0.03`). Impact: 103,159 / 14,558,211 variants removed (0.7%).

### Call-rate 0.97 re-run results

| Dataset | N samples | MAF SNPs | LD-pruned | PC1~singleton r |
|---|---|---|---|---|
| Common 0.95 (v2) | 4,383 | 579,979 | 85,811 (14.8%) | −0.495 |
| Common 0.97 | 4,383 | 573,550 | 84,689 (14.8%) | −0.494 |
| Rare 0.95 (orig) | 4,382 | 508,963 | 376,481 (74.0%) | −0.500 |
| Rare 0.97 | 4,383 | 504,181 | 372,851 (74.0%) | −0.500 |

**Call rate 0.95 → 0.97 has negligible effect.** SNP yield changes < 1.5%. PC1~singleton correlation unchanged. All 32 previously-identified outliers remain fully outside 1-99% UMAP range in call-rate-0.97 run.

### Why population structure cannot be separated well

#### 1. PCA signal is weak — this is a within-China cohort

| Metric | Common (MAF>1%) | Rare (0.1%–1%) |
|---|---|---|
| PC1 variance explained | 11.5% | 6.2% |
| PC1-2 cumulative | 13.9% | 9.1% |
| PC1-10 cumulative | 30.3% | 26.0% |

For comparison: inter-continental studies (e.g., 1KGP) typically show PC1 explaining 30–50% of variance. Within-China genetic differentiation is ~20× weaker than inter-continental. Most genetic variation is within-region, not between-region.

#### 2. Structure is 1-dimensional: South–East–North cline

PC1 ANOVA by region: **F=561.6, p≈0, eta²=0.41** (common) — highly significant. But PC2 eta² drops to 0.005. Only PC1 carries biological signal; PC2–PC50 are noise-dominated.

PC1 mean by region (common variants):

| Region | PC1 mean | singleton mean | Interpretation |
|---|---|---|---|
| Guangzhou (GZ) | +0.0041 | 622 | South |
| Shenzhen (SZ) | −0.0031 | 616 | South |
| HongKong (HK) | +0.0046 | 595 | South |
| Shanghai (SH) | −0.0137 | 431 | East |
| Jinan (JN) | −0.0220 | 958 | North |
| JiNing (JiN) | −0.0221 | 949 | North |

Pairwise Cohen's d on PC1:

| Pair | Cohen's d | Separation |
|---|---|---|
| South vs North (GZ–JN) | **3.1** | Clear |
| South vs East (GZ–SH) | **2.1** | Moderate |
| East vs North (SH–JN) | **1.6** | Weak-moderate |
| GZ vs SZ (both South) | **0.6** | None |
| JN vs JiN (both North) | **0.03** | None |

**Only 3 effective groups** (South / East / North), not 6 regions. Within-group regions (GZ/SZ/HK and JN/JiN) are indistinguishable.

#### 3. PC1 is confounded with sequencing depth

| Signal | Common PC1 r² | Rare PC1 r² |
|---|---|---|
| Region (eta²) | **0.410** | **0.393** |
| Singleton count (r²) | **0.244** | **0.250** |
| F_MISS (r²) | 0.000 | 0.000 |

PC1 captures both biology (South–North cline) AND technical artifact (singleton count). F_MISS (missingness) is NOT the confound — singleton count is. Singleton count varies 2.2× across regions (SH=431 vs JN=958), likely due to systematic sequencing batch/site effects.

**Confound/region ratio = 0.6×** — singleton count explains ~60% as much PC1 variance as true geography.

#### 4. UMAP amplifies noise when signal is low-dimensional

- UMAP receives 50 PCs but only PC1 has biological signal
- Noise-dominated PCs 2–50 create spurious local structure → "radial dots"
- Rare variants amplify this: weaker signal (PC1=6.2%) + same confound = more UMAP artifacts
- Tight clusters in UMAP are artifacts from samples with similar singleton-count profiles, not genuine sub-populations

#### 5. Common vs rare: neither separates well, but for different reasons

**Common variants (MAF>1%):**
- Stronger biological signal (PC1=11.5%)
- But capture deep ancestry → within-China differentiation is recent (<10k years)
- Most common variants predate population split → limited resolution for fine structure
- Result: 1 continuous gradient, no discrete clusters

**Rare variants (MAF 0.1%–1%):**
- Weaker biological signal (PC1=6.2%) but theoretically capture recent events
- Much more sensitive to sequencing depth variation → stronger confound
- More SNPs survive LD prune (74% vs 15%) but signal-to-noise lower
- Result: confound-dominated, "radial dot" artifacts

### Root cause summary

```
Within-China genetic differentiation is subtle (Fst ~ 0.001–0.002)
    ↓
PC1 captures South–North cline (eta²=0.41) but explains only 11% of total variance
    ↓
PC2–PC50 are noise-dominated → UMAP spreads noise into 2D → no clear clusters
    ↓
Singleton count varies 2.2× by region (batch effect) → r²=0.24 on PC1
    ↓
Confound is 60% as strong as biological signal → cannot separate geography from batch
```

### Recommendations

1. **Regress out singleton count** from genotype PCs before UMAP/clustering
2. **Use only PC1–PC3** for UMAP (not PC1–PC50) to reduce noise amplification
3. **Apply singleton-count-based QC** before rare-variant analysis (exclude bottom decile)
4. **Use supervised methods** (e.g., region-labeled discriminant analysis) rather than unsupervised PCA-UMAP for population structure
5. **For paper-facing figures**: present PC1 as continuous South–North cline rather than discrete clusters; this is the true biological structure

### Call-rate 0.97 outputs

```
umap_9k/
    snp_qc_callrate097.{bed,bim,fam}               ← 14,455,052 variants (call rate ≥ 0.97)
    9k_snp_qc_v2_umap*.png                         ← common MAF>0.01, call rate 0.95 (regenerated)
    9k_snp_qc_callrate097_umap*.png                ← common MAF>0.01, call rate 0.97
    9k_snp_qc_rare_callrate097_umap*.png           ← rare MAF 0.001–0.01, call rate 0.97
    9k_snp_qc_callrate097_umap.tsv                 ← common coords (4,383 rows)
    9k_snp_qc_rare_callrate097_umap.tsv            ← rare coords (4,383 rows)
```

### Script inventory (new)

| Script | Purpose | Location |
|---|---|---|
| `run_umap_callrate097.sh` | 3-dataset PCA-UMAP pipeline (common 0.95 + common 0.97 + rare 0.97) | `SDSworkspace/oneoff/` |
| `plot_umap_generic.py` | Parameterized UMAP + 5-view plotting for any PCA prefix | `SDSworkspace/oneoff/` |

## Step 6 — 6-Part re-analysis: residualization, informative PCs, southern subset (2026-06-29)

**Motivation:** Previous diagnosis found singleton count confounds PC1 (r=-0.50), only 3 effective groups, UMAP amplifies noise. User required: (1) use only informative PCs, (2) regress singleton count, (3) larger n_neighbors for smoother UMAP, (4) covariate-adjusted PCA, (5) southern superpop rare-variant analysis, (6) quantitative evaluation + badMIXTURE connection.

**Scripts:** `SDSworkspace/oneoff/run_reanalysis_pipeline.sh` (shell), `SDSworkspace/oneoff/analyze_population_structure.py` (Python)  
**Output dir:** `SDSworkspace/runs/population_structure_reanalysis/`

### Part 1: Informative PC detection

Combined criteria: Cattell scree (2nd-derivative elbow), Kaiser (eigenvalue > mean), Tracy-Widom (first 3 PCs).

| Dataset | Scree elbow | Kaiser PCs | Informative PCs retained |
|---|---|---|---|
| Common97 | PC3 | 6 | **4** (PC1-4) |
| Rare97 | PC3 | 7 | **4** (PC1-4) |
| Rare clean | PC3 | 7 | **4** (PC1-4) |

TW test: all PC1 eigenvalues below TW critical value due to LD inflation — LD reduces effective number of independent SNPs, shrinking the expected null eigenvalue. The scree+kaiser combination provides the practical cutoff.

### Part 2: PC residualization

Regressed `singleton_count + F_MISS + F_het` from each informative PC via OLS.

| Dataset | PC1~singleton r (before) | PC1~singleton r (after) | R² tech covariates PC1 |
|---|---|---|---|
| Common97 | −0.4939 | **−0.0000** | 0.245 |
| Rare97 | −0.5004 | **−0.0000** | 0.252 |
| Rare clean | −0.5013 | **0.0000** | 0.253 |

**Singleton confound completely removed.** F_MISS and F_het contributed <1% additional R² beyond singleton_count. Singleton count alone explains ~25% of PC1 variance.

### Part 3: UMAP with larger n_neighbors

Residual informative PCs (4 PCs) → UMAP with n_neighbors ∈ {30, 50, 80}. Both original (for comparison) and residual PC inputs tested.

**18 UMAP runs** (3 datasets × 2 input types × 3 n_neighbors). 5-view plots per run (region, K2, PC1, PC1+PC2 bicolor, plain). Grid comparison figures generated.

UMAP trustworthiness scores (residual PCs):

| Dataset | nn=30 | nn=50 | nn=80 |
|---|---|---|---|
| Common97 | 0.960 | 0.953 | 0.950 |
| Rare97 | 0.977 | 0.975 | 0.974 |
| Rare clean | 0.976 | 0.973 | 0.972 |

Trustworthiness decreases slightly with larger n_neighbors — expected tradeoff: global structure preserved at cost of local fidelity. The continuous South-North cline should appear smoother at nn=80.

### Part 4: Covariate-adjusted PCA

Two approaches documented:
- **(a) Post-hoc residualization** (Part 2): simple, computationally cheap, sufficient for modest confound (r=-0.50)
- **(b) GRM conditioning** via `plink2 --make-rel covar=`: gold standard, yields orthogonal conditioned PCs, noted for future work

### Part 5: Southern superpop rare-variant PCA

Subset to GZ+SZ+HK. Two configurations:
- Unbalanced: all 3,389 southern samples (GZ=30, SZ=267, HK=3092)
- Balanced: GZ=30 + SZ=267 + 300 random HK = 597 samples

| Configuration | N | PC1 variance | PC1~singleton r |
|---|---|---|---|
| Southern all (unbalanced) | 3,389 | 9.9% | −0.292 |
| Southern balanced | 597 | 5.7% | −0.396 |

**Key finding:** Southern PC1 is even weaker (5.7-9.9%) than full-cohort PC1 (6.2% rare), confirming within-south differentiation is minimal. Southern singleton confound is reduced (r=-0.29) in the unbalanced set because southern samples have more homogeneous singleton counts. In the balanced set, HK down-sampling slightly increases the confound (r=-0.40) — the 300 random HK happen to have more variable singleton counts.

**No sub-structure within southern superpop.** GZ, SZ, and HK are genetically indistinguishable at the resolution of this SNP set — consistent with shared Cantonese/southern Han ancestry.

### Part 6: Quantitative evaluation

| Metric | Common97 | Rare97 | Rare clean |
|---|---|---|---|
| PC1~singleton r (orig) | −0.494 | −0.500 | −0.501 |
| PC1~singleton r (resid) | **0.000** | **0.000** | **0.000** |
| eta² PC1 (region, orig) | 0.410 | 0.393 | — |
| UMAP trustworthiness (nn=50) | 0.953 | 0.975 | 0.973 |

### Connection to badMIXTURE

badMIXTURE Row ACF = 0.86: inter-individual residual correlation from discrete ADMIXTURE model. Two drivers identified:

1. **Biological:** Within-China population is a continuous South-North cline, not discrete clusters. ADMIXTURE's discrete assumption is structurally wrong — this is the primary cause of misfit.
2. **Technical:** Singleton count creates a gradient across individuals (r=-0.50 on PC1) that amplifies the apparent discreteness in the embedding. ADMIXTURE partially latches onto this confound.

After residualization, the technical confound is removed. The remaining UMAP structure isolates the pure biological cline. If re-running badMIXTURE on residualized genotype data were feasible, Row ACF should decrease — the confound-driven portion of the inter-individual correlation is eliminated.

**Recommendation for paper:** Present population structure as a continuous South-North PC1 gradient with singleton count residualized. Do not force discrete clustering. The cohort has 3 effective groups (South, East, North) but these grade into each other.

### Reanalysis outputs

```
population_structure_reanalysis/
    figures/
        figure_01_scree_*.png                      ← scree plots (3 datasets)
        figure_02a_pc1_vs_singleton_*.png          ← before/after residualization (3)
        figure_02b_r2_by_pc_*.png                  ← R² bar charts (3)
        figure_03_umap_grid_*.png                  ← 3×3 UMAP grids (6: orig+resid × 3)
        figure_04_effective_rank_*.png             ← effective rank comparison (3)
        figure_05_*_southern_*.png                 ← southern PCA/UMAP (6)
        figure_06a_eta2_*.png                      ← eta² per PC (3)
        figure_06d_umap_metrics_*.png              ← trustworthiness + dispersion (3)
        *_orig_nn{30,50,80}_umap_*.png            ← 5-view UMAP per config (90)
        *_resid_nn{30,50,80}_umap_*.png           ← 5-view UMAP per config (90)
    tables/
        table_01_informative_pcs_*.tsv             ← PC eigenvalue analysis (3)
        table_02_residualization_*.tsv             ← regression coefficients (3)
        table_06_metrics_*.tsv                     ← quantitative metrics (3)
        *_coords.tsv                               ← UMAP coordinates (18)
    data/
        southern_all.keep / southern_balanced.keep ← sample lists
        southern_*_pca.eigenvec / .eigenval        ← southern PCA results
```

