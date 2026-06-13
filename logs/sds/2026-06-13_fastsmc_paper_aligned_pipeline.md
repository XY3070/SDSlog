# FastSMC Selection Scan: Paper-Aligned Pipeline

**Date**: 2026-06-11 ~ 2026-06-13
**Status**: Complete (4 configurations tested)

## Motivation

Implement FastSMC-based selection scan aligned with the Science Advances paper (doi:10.1126/sciadv.adi8419):
- "Since the method was developed and tuned using microarray data, we extracted a superset of variants included in the Illumina HumanOmniExpressExome BeadChip array, the Illumina Infinium Asian Screening Array, and the Affymetrix Japonica array."
- 0.05 cM sliding windows
- Gamma null model with iterative refinement
- One-sided p-values, genome-wide threshold 5×10⁻⁸

## Pipeline Architecture

### Scripts
- `FastSMC/scripts/run_fastsmc_validation.py` — VCF staging + FastSMC execution
- `FastSMC/scripts/summarize_fastsmc_genetic_windows.py` — IBD → DRC50 per genetic window
- `FastSMC/scripts/gamma_selection_scan.py` — Gamma null + iterative refinement + p-values
- `FastSMC/scripts/build_genetic_windows.py` — 0.05 cM window builder from SDSworkspace maps
- `FastSMC/scripts/extract_microarray_variants.py` — GRCh37→GRCh38 microarray position matcher
- `FastSMC/scripts/report_fastsmc_manhattan.R` — Manhattan plot generator (CMplot)
- `FastSMC/scripts/run_microarray_pipeline.sh` — End-to-end parallel pipeline

### Genetic Maps
- **Source**: `SDSworkspace/input/reference/maps/no_chr_in_chrom_field/plink.chr{N}.GRCh38.map`
- **Format**: PLINK (chrom, marker_id, cM, bp)
- **Total**: 3,545.83 cM across 22 autosomes
- **Migration**: Previously used FastSMC/maps (RELATE format), now centralized in SDSworkspace

### Microarray Variants
- **Source manifests**: `SDSworkspace/input/reference/microarray/`
  - `HumanOmniExpressExome.csv` (938,058 autosomal SNPs)
  - `InfiniumAsianScreeningArray.csv` (624,148 autosomal SNPs)
- **GRCh37→GRCh38 matching**: Nearest-variant within ±5kb (no chain file available)
- **Match rate**: 88.5% (1,220,702 unique GRCh38 positions)
- **BED file**: `SDSworkspace/input/reference/microarray/microarray_matched_grch38_noprefix.bed`

### Mask
- **File**: `sds_rebuild/masks/grch38_fastsmc_paper_mask.bed`
- **Components**: Centromeres (22) + Zero-recombination regions (26, ≥50kb)
- **Coverage**: 112 Mb (3.9% of genome)

## Results: 4 Configurations

### Config 1: WGS + 0.05 cM
- **Variants**: All WGS SNPs (no filter)
- **Windows**: 1,308,560 (0.05 cM)
- **Non-zero windows**: 4.5%
- **Gamma shape**: 0.57 (< 1, zero-inflated)
- **Significant (p<5e-8)**: 416
- **Top -log10(p)**: 14,081 (chr8:112.9Mb)
- **Problem**: Extreme p-values due to zero-inflation

### Config 2: WGS + 0.5 cM
- **Variants**: All WGS SNPs
- **Windows**: 152,966 (0.5 cM)
- **Non-zero windows**: ~30%
- **Gamma shape**: 1.37 (> 1, healthy)
- **Significant (p<5e-8)**: 110
- **Top -log10(p)**: 6,430 (chr15:22.2Mb)
- **Improvement**: Shape > 1, but p-values still high

### Config 3: Microarray + 0.05 cM
- **Variants**: 1,220,702 microarray-matched (~18.7k/chr)
- **Windows**: 1,308,560 (0.05 cM)
- **Non-zero windows**: 4.5%
- **Gamma shape**: 0.33 (< 1, still zero-inflated)
- **Significant (p<5e-8)**: 905
- **Top -log10(p)**: 1,128 (chr10:49.8Mb ABCC2)
- **Problem**: Zero-inflation persists (window size vs sample size)

### Config 4: Microarray + 0.5 cM ⭐ Recommended
- **Variants**: 1,220,702 microarray-matched
- **Windows**: 152,966 (0.5 cM)
- **Non-zero windows**: 4.5%
- **Gamma shape**: 0.45 (< 1)
- **Significant (p<5e-8)**: 177
- **Top -log10(p)**: 658 (chr1:120.1Mb)
- **Best balance**: Microarray variants + reasonable window size

## Key Signals (Config 4: Microarray + 0.5 cM)

| Rank | Position | -log10(p) | Gene/Region |
|------|----------|-----------|-------------|
| 1 | chr1:120.1Mb | 658 | Unknown |
| 2 | chr2:86.5Mb | 350 | Near centromere ⚠️ |
| 3 | chr10:48.3Mb | 292 | ABCC2/CPN1 region |
| 4 | chr8:11.6Mb | 283 | Unknown |
| 5 | chr10:49.0Mb | 252 | ABCC2 region |
| 6 | chr10:49.6Mb | 249 | ABCC2/CPN1 |
| 7 | chr15:32.7Mb | 238 | MAP1A/STRC |
| 8 | chr2:88.1Mb | 204 | Near centromere ⚠️ |
| 9 | chr1:147.0Mb | 187 | Unknown |
| 10 | chr10:50.0Mb | 160 | ABCC2 region |

## Known Issues

1. **Zero-inflation**: With 1,255 samples, most 0.05 cM windows have 0 IBD segments (only 4.5% non-zero). The paper used 487,409 samples (388× more), so their windows had much better coverage.

2. **Gamma shape < 1**: Even with 0.5 cM windows + microarray variants, shape = 0.45 (< 1). With WGS + 0.5 cM, shape = 1.37 (healthy). The difference is that microarray has fewer variants per window.

3. **Centromere-adjacent signals**: chr2:86.5Mb and chr2:88.1Mb are near the chr2 centromere (92.2-94.1Mb) and may be artifacts.

4. **chr2:3Mb signal**: Persistent across all configurations, not in a zero-recombination region. May be a real selective sweep or structural variant artifact.

## Output Files

| File | Description |
|------|-------------|
| `FastSMC/results/summary/NCN/full/genomewide_0.5cM_microarray_gamma.tsv` | Full results (Config 4) |
| `FastSMC/results/summary/NCN/full/genomewide_0.5cM_microarray_gamma.top.tsv` | Per-chromosome top peaks |
| `FastSMC/plots/NCN/full/NCN.genomewide_microarray_0.5cM.png` | Manhattan plot |
| `FastSMC/plots/NCN/full/NCN.genomewide_microarray_0.5cM.pdf` | Manhattan plot (PDF) |
| `FastSMC/resources/genetic_windows_0.05cM.bed` | 59,645 windows |
| `FastSMC/resources/genetic_windows_0.5cM.bed` | 7,003 windows |
| `SDSworkspace/input/reference/microarray/microarray_matched_grch38_noprefix.bed` | 1.22M positions |
| `sds_rebuild/masks/grch38_fastsmc_paper_mask.bed` | Centromere + zero-recomb mask |
| `sds_rebuild/masks/grch38_zero_recombination.bed` | Zero-recomb regions from maps |
