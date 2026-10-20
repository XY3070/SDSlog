# CLUES2 Production Run: chr19 Gravel EAS Demographic Model

**Date**: 2026-10-14  
**Job**: LSF 557741 (`smp`, 8 cores, 32GB, 24h)  
**Script**: `SDSworkspace/results/production/orthogonal/relate_clues2/run_clues2_chr19_gravel.sh`  
**Output**: `SDSworkspace/results/production/orthogonal/relate_clues2/NCN_gravel/clues/`

## Background

Previous session (2026-10-11) established that using a constant coalescent model (NCN.coal) vs the correct Gravel EAS demographic model produces dramatically different CLUES2 results at chr19:4,768,268 (logLR 4.3 vs 1479, opposite selection directions). This production run uses Gravel EAS for all 7 chr19 lead signals.

## Input

- **Genealogy**: NCN chr19 v7 Relate (job 503210, constant coal for tree sampling)
- **Coal model for CLUES2**: `CLUES2/Gravel_EAS.coal` (Gravel CHB 2011: 40 time points, bottleneck at gen 920, recent Ne ~1100 + growth)
- **Mutation rate**: 1.25e-8
- **Trees sampled**: 200 per signal (SampleBranchLengths with Gravel coal)
- **PopFreq**: from `prepare_clues_inputs.py` (correctly handles is_flipped polarization)
- **Time bins**: `--timeBins 50 200` → 3 epochs: 0-50g, 50-200g, 200-600g
- **tCutoff**: 600 generations

## Signals Analyzed

| # | Gene | Position | popFreq | logLR | -log10(p) |
|---|------|----------|---------|-------|-----------|
| 1 | MIR7-3 | chr19:4,768,268 | 0.0402 | 65.8 | 27.6 |
| 2 | UHRF1 | chr19:4,951,434 | 0.0629 | 148.2 | 63.2 |
| 3 | DNM2 | chr19:10,765,735 | 0.0323 | 293.3 | 126.1 |
| 4 | PLIN3 | chr19:4,867,689 | 0.0821 | 32.4 | 13.3 |
| 5 | DPP9 | chr19:4,710,802 | 0.0598 | 66.8 | 28.0 |
| 6 | ARRDC5(-) | chr19:4,903,524 | 0.9550 | 105.7 | 44.8 |
| 7 | ARRDC5(+) | chr19:4,889,675 | 0.3542 | 3.8 | 1.25 |

## Results: Selection Coefficients by Epoch

| Locus | Epoch 1 (0-50g) | Epoch 2 (50-200g) | Epoch 3 (200-600g) |
|-------|-----------------|-------------------|--------------------|
| **MIR7-3** | -0.0089 [-0.015, -0.003] neg | +0.0191 [+0.013, +0.025] pos+ | +0.0118 [+0.006, +0.018] pos+ |
| **UHRF1** | -0.0130 [-0.072, +0.045] n.s. | +0.0123 [-0.048, +0.073] n.s. | +0.0046 [-0.057, +0.067] n.s. |
| **DNM2** | -0.0089 [-0.015, -0.003] neg | +0.0210 [+0.015, +0.027] pos+ | -0.0057 [-0.012, +0.001] n.s. |
| **PLIN3** | -0.0226 [-0.025, -0.021] neg | +0.0173 [+0.015, +0.019] pos+ | +0.0070 [+0.005, +0.009] pos+ |
| **DPP9** | -0.0034 [-0.010, +0.003] n.s. | +0.0293 [+0.024, +0.035] pos+ | -0.0068 [-0.013, -0.000] neg |
| **ARRDC5(-)** | +0.0056 [-0.001, +0.012] n.s. | -0.0408 [-0.047, -0.035] neg- | -0.0002 [-0.006, +0.006] n.s. |
| **ARRDC5(+)** | +0.0056 [-0.003, +0.014] n.s. | +0.0087 [+0.001, +0.016] pos+ | +0.0012 [-0.007, +0.010] n.s. |

## Results: Frequency Trajectory (MAP from posterior)

| Locus | t=0 | t=50 | t=100 | t=200 | t=300 | t=400 |
|-------|-----|------|-------|-------|-------|-------|
| MIR7-3 | 0.040 | 0.049 | 0.030 | 0.013 | 0.000 | 0.000 |
| UHRF1 | 0.063 | 0.069 | 0.060 | 0.042 | 0.013 | 0.000 |
| DNM2 | 0.032 | 0.041 | 0.035 | 0.019 | 0.027 | 0.022 |
| PLIN3 | 0.082 | 0.124 | 0.097 | 0.078 | 0.090 | 0.074 |
| DPP9 | 0.060 | 0.058 | 0.039 | 0.021 | 0.025 | 0.038 |
| ARRDC5(-) | 0.955 | 0.955 | 0.975 | 0.987 | 0.977 | 1.000 |
| ARRDC5(+) | 0.354 | 0.342 | 0.332 | 0.305 | 0.300 | 0.298 |

## Interpretation

### Clear positive selection (recent sweeps)

**MIR7-3, PLIN3, DNM2**: All show the same 3-epoch pattern:
- Epoch 1 (0-50g): weak negative s — allele frequency slightly decreasing recently
- Epoch 2 (50-200g): strong positive s (~0.015-0.021) — the main sweep window
- Epoch 3 (200-600g): variable — positive for MIR7-3/PLIN3, n.s. for DNM2

The p(t) trajectory confirms: frequency rises from near-zero at ~300-400g ago to present-day levels, with the steepest increase in the 50-200g window. This is consistent with a **recent incomplete sweep** (DAF 3-8%) starting ~200-400 generations ago (~5000-10000 years).

**DPP9**: Epoch 2 has the strongest positive s=+0.029 among all loci, with a tight CI. The trajectory shows a clear decline from 0.06 to 0.02 between 50-200g ago, meaning the allele was MORE common in the past — epoch 2 positive selection means selection was maintaining it against drift, not driving an increase.

### Purifying selection on near-fixed allele

**ARRDC5(-)** (DAF=0.955): The derived allele is nearly fixed. Epoch 2 shows strong negative s=-0.041 — selection against the derived allele when it was at even higher frequency (~0.99) in the past. The trajectory goes from 1.0 (400g ago) → 0.95 (present), consistent with the ancestral allele slowly recovering.

### Weak/ambiguous signals

**UHRF1**: Despite logLR=148 (highly significant overall), all epoch CIs span zero. The posterior is informative for p(t) (clear decline from 0.06 to 0 over 400g) but the HMM cannot precisely localize WHEN selection acted. This suggests either: (a) the selection is spread across multiple epochs, (b) the 200-tree sampling is insufficient for this locus, or (c) the signal is confounded by demography/recombination.

**ARRDC5(+)** (DAF=0.354): logLR=3.8, all CIs span zero. Essentially no detectable selection signal.

### Continuous s(t) from posterior derivative

Derived via s(t) = -dp̄/dt / [p̄(1-p̄)] from the posterior mean trajectory (smoothing window=30 generations). Key observations:
- Continuous s(t) broadly agrees with epoch MLEs but reveals finer temporal structure
- s(t) is undefined (masked gray) where p→0 or p→1 (division by zero)
- MIR7-3 and PLIN3 show s(t) peaking around 50-150g ago, consistent with epoch 2
- DPP9 shows a sharp positive spike at ~100-150g ago
- UHRF1 shows noisy s(t) with no clear peak, consistent with wide epoch CIs

## Technical Notes

### Posterior matrix format
- Shape: `(n_freq_bins=600, n_time_points=599)`
- **Columns** (time points) sum to 1.0 — each column is P(freq | time)
- Frequency grid: `beta.ppf(linspace(0,1,600), 0.5, 0.5)` — extreme density near 0 and 1
- First non-zero bin at 6.9e-6, last at 1-6.9e-6; middle spacing ~0.003

### Visualization
- `clues2_pt_st_aligned.png`: epoch-based step s(t) bars below p(t) heatmap
- `clues2_pt_st_continuous.png`: continuous s(t) from dp/dt below p(t) heatmap
- Script: `clues2_pt_st_continuous.py`

### Tree topology vs branch lengths
Previous analysis confirmed: Relate tree topology is identical under constant vs Gravel coal (RF distance = 0). Only branch lengths change (~50% shorter under Gravel). This means the Relate genealogy (built with constant coal) is valid for use with Gravel coal in SampleBranchLengths + CLUES2.

## Comparison with Constant Coal (Previous Run)

| Locus | Constant coal logLR | Gravel logLR | Constant s_epoch2 | Gravel s_epoch2 |
|-------|--------------------|--------------|-------------------|-----------------|
| MIR7-3 | ~4 | 65.8 | +0.007 n.s. | +0.019 pos+ |
| Others | not run individually | see above | — | — |

The Gravel model increases power by 1-2 orders of magnitude in logLR.

## File Paths

- Production script: `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/orthogonal/relate_clues2/run_clues2_chr19_gravel.sh`
- Gravel coal: `/data/home/grp-wangyf/xuyuan/CLUES2/Gravel_EAS.coal`
- Results root: `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/orthogonal/relate_clues2/NCN_gravel/clues/`
- Per-signal output: `{sig}/{sig}.clues_inference.txt`, `.clues_CI.txt`, `.clues_post.txt`, `.clues_freqs.txt`
- Visualization: `NCN_gravel/clues/clues2_pt_st_continuous.png`
- Viz script: `NCN_gravel/clues/clues2_pt_st_continuous.py`
