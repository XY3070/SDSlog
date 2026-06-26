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

## Script inventory

| Script | Purpose | Location |
|---|---|---|
| `plot_ancestry_one_figure.py` | Single stacked-bar ancestry plot (K=2), monotonic sort | `SDSpipeline/scripts/` |
| `run_badmixture_analysis.R` | ADMIXTURE cluster plots, K evaluation, region composition | `SDSworkspace/oneoff/` |
| `run_badmixture_residuals.R` | badMIXTURE residual analysis (model vs observed per-site) | `SDSworkspace/oneoff/` |
| `run_umap_9k.sh` | PCA-UMAP pipeline (common variants MAF>1%) | `SDSworkspace/oneoff/` |
| `run_umap_9k_rare.sh` | PCA-UMAP pipeline (rare variants MAF 0.1%–1%) | `SDSworkspace/oneoff/` |

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
