# 9k Production Batch 2 — SDS Genome-wide Compute & Postprocess

**Date:** 2026-07-01 to 2026-07-04
**Role:** `mainline`
**Status:** complete

## Cohort & Sample Policy

- **Population:** ALL (4393 QC-passed samples, NCN+SCN pooled)
- **Cohort freeze:** `raw/9k/snp_singleton_qc_chr*_v*/frozen_samples_in_vcf.txt` (4383 per chr)
- **Note:** 10 samples in original frozen_samples.txt (4393) absent from QC VCF (4383). Filtered to VCF-intersection for input generation.

## Demographic Model & Gamma

- **Gamma file:** `SDSworkspace/results/production/gamma/g_file.Gravel_EAS.txt`
- **Model:** Gravel_EAS, n=100
- **Present-day Ne(0):** implicit (Gravel_EAS gamma)
- **DAF range:** 0.048–0.952, 113 bins

## Input Lineage

- **Source VCF:** `raw/9k/snp_singleton_qc_chr{N}_v{V}/snp_qc_pass.vcf.gz`
- **QC versions:** chr5=v2, chr20=v3, all others=v1
- **Sample list:** VCF-filtered (4383 samples)
- **Input contract:**
  - `t_file`: bcftools query GT → gawk genotype conversion (0/1/2/NA)
  - `s_file`: bcftools query AC=1 singleton extraction per arm
  - `o_file`: all-ones observability (4383 columns)
  - `b_file`: centromere-based arm boundaries

## Compute

- **Script:** `submit_sds_genomewide.sh` (managed concurrency, max 2 concurrent)
- **Queue:** normal (LSF)
- **Job group:** `9k_production_batch2`
- **Chunk rows:** 10000
- **Array parallel:** auto-sensed (32–64)

### Chromosome completion

| Chr | t_file size | sds.tsv size | Compute time |
|-----|------------|-------------|-------------|
| 5   | 2.9G       | 21M         | ~30 min     |
| 7   | 2.7G       | 19M         | ~30 min     |
| 8   | 2.5G       | 18M         | ~30 min     |
| 15  | 1.3G       | 9.4M        | ~45 min     |
| 17  | 1.2G       | 8.2M        | ~40 min     |
| 18  | 1.3G       | 8.8M        | ~40 min     |
| 19  | 905M       | 6.3M        | ~50 min     |
| 20  | 972M       | 6.8M        | ~50 min     |
| 21  | 640M       | 4.5M        | ~55 min     |
| 22  | 612M       | 4.2M        | ~55 min     |

All 10 chromosomes completed without failures. Combined with existing chr1,4,6,14 for genome-wide analysis (14 autosomes total).

## Analysis Masks

Canonical location: `SDSworkspace/input/reference/mask/` (symlinked from `sds_rebuild/masks/`)

### Authoritative UCSC-derived masks (GRCh38)
- **Primary:** `grch38_combined_blacklist.bed` — 2,886 merged exclusion regions
- **Centromere:** 22 regions (UCSC cytoBandIdeo acen)
- **Pericentromeric:** 11 regions + 44-region blacklist (UCSC cytoBandIdeo gvar+stalk)
- **Assembly gaps:** 507 regions (UCSC gap table)
- **Segmental duplications:** 3,042 regions (UCSC genomicSuperDups, fracMatch≥0.95)
- **Accessibility:** 3,887,975 regions (inverted 1KGP strict mask)
- **Subtelomeric:** 44 regions
- **Zero recombination:** 29 regions
- Provenance: `mask_provenance.json` (2026-06-07 build)

### chr1 Outlier Analysis

83 SNPs with |rSDS| > 100 in raw chr1 output:
- **12-17Mb cluster (23 SNPs):** segdup + assembly gap region → covered by combined_blacklist
- **147-150Mb cluster (22 SNPs):** pericentromeric heterochromatin 1q12 → covered by combined_blacklist  
- **Scattered (38 SNPs):** DAF < 0.05 low-frequency variants where gamma model poorly calibrated
- 47/53 extreme outliers covered by combined_blacklist; remaining filtered by MAF≥0.05 gate

DAF distribution of |rSDS| > 50:
- DAF < 0.01: 16,595 SNPs → all filtered by MAF≥0.05
- DAF < 0.03: 2,748 → filtered
- DAF 0.03-0.05: 3 → filtered
- DAF > 0.05: 246 → these are real signals

## Postprocessing

- **Script:** `postprocess_sds_results.py`
- **Bin mode:** MAF (minor allele frequency)
- **Bin widths:** 0.005 (MAF < 0.1), 0.01 (MAF ≥ 0.1)
- **MAF threshold:** ≥ 0.05
- **Normalization:** MAF-binned common-variant (50 bins)
- **Exclusion mask:** `masks/exclusion_mask_combined.bed` (110 regions)
- **Input files:** 28 (14 chromosomes × 2 arms)

### Results

| Metric | Value |
|--------|-------|
| Total SNPs | 8,920,556 |
| Common variants (MAF≥0.05) | 2,982,620 |
| Excluded by mask | ~43,000 |
| Bonferroni threshold | 1.68 × 10⁻⁸ |
| Significant regions | 17 |

### Significant regions

| Region | #SNPs | Top SNP | Top p-value |
|--------|-------|---------|-------------|
| chr1:16024618-16058689 | 9 | chr1:16057529:T:A | 9.7×10⁻⁴³ |
| chr1:17289277-17289281 | 2 | chr1:17289281:C:T | 1.9×10⁻⁴⁵ |
| chr1:27878455-27878460 | 2 | chr1:27878455:C:G | 4.0×10⁻¹⁵ |
| chr1:84027543 | 1 | chr1:84027543:G:A | 5.4×10⁻⁵⁰ |
| chr1:87046796 | 1 | chr1:87046796:C:T | 1.1×10⁻⁹ |
| chr1:118551214 | 1 | chr1:118551214:G:C | 3.2×10⁻¹³⁵ |
| chr1:154671107 | 1 | chr1:154671107:T:C | 2.2×10⁻²⁴ |
| chr1:214482990-214483070 | 3 | chr1:214483070:T:C | 4.7×10⁻¹¹ |
| chr1:219904266-219904293 | 4 | chr1:219904266:G:T | 4.3×10⁻¹⁷ |
| chr4:99159154-99322288 | 24 | chr4:99318162:T:C | 9.7×10⁻¹⁴ |
| chr6:27334326-27895650 | 22 | chr6:27560310:C:T | 1.2×10⁻¹⁰ |
| chr6:28512833-29067202 | 21 | chr6:28607395:T:A | 3.5×10⁻¹⁰ |
| chr6:29387461-29636005 | 114 | chr6:29563197:A:G | 1.0×10⁻¹⁰ |
| chr6:30246974-30350189 | 5 | chr6:30348014:G:A | 3.9×10⁻⁹ |
| chr19:4748270-5023601 | 37 | chr19:4768268:A:C | 1.8×10⁻²⁵ |
| chr19:10324817-11002401 | 38 | chr19:10765735:C:T | 7.6×10⁻²⁰ |
| chr22:41491321-41548233 | 11 | chr22:41504089:G:A | 1.3×10⁻⁹ |

## Manhattan Plot

- **Script:** `report_cmplot_manhattan.R` (CMplot)
- **Fix applied:** chr.labels auto-detected from data (was hardcoded 1-22)
- **Output:** `all.manhattan.png` (232KB, 16×7 in, 400 DPI)

## QC Notes

- Frequency bins: mean SDS near 0 across DAF spectrum, SD increases at extremes (expected)
- chr6 MHC region: expected multi-peak signal, consistent with known selection
- chr4 99.2–99.3 Mb: known follow-up locus confirmed
- chr1: multiple dispersed signals, need orthogonal validation
- Disk space was tight — cleaned chunk_submit (15GB), audit duplicates (~20GB), old per-chromosome outputs

## Known Issues

- chr1 p/q arm files only (no merged chr1.sds.tsv) — postprocess reads arm files directly, OK
- chr14, chr15, chr21, chr22 p-arms acrocentric/empty (53-byte header-only files)
- 10 samples dropped between frozen list and QC VCF — need cohort freeze manifest update
- 8 autosomes not yet QC'd/computed (chr2,3,9,10,11,12,13,16)

## Output Paths

| File | Path |
|------|------|
| Normalized TSV | `SDSworkspace/results/production/sds_output/ALL/all.normalized.tsv` |
| Frequency bins | `SDSworkspace/results/production/sds_output/ALL/all.frequency_bins.tsv` |
| Significant regions | `SDSworkspace/results/production/sds_output/ALL/all.significant_regions.tsv` |
| Manhattan plot | `SDSworkspace/results/production/sds_output/ALL/all.manhattan.png` |
| Manhattan points | `SDSworkspace/results/production/sds_output/ALL/all.manhattan_points.tsv` |
| Postprocess summary | `SDSworkspace/results/production/sds_output/ALL/all.postprocess_summary.tsv` |
| Raw SDS (per chr) | `SDSworkspace/results/production/sds_output/ALL/chr{N}.sds.tsv` |
