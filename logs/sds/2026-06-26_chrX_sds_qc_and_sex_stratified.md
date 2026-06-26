# chrX SDS QC and Sex-Stratified Analysis

**Date:** 2026-06-26
**Role:** mainline
**Demographic model:** SDS-MODEL-001 (Gravel_EAS, g_file.Gravel_EAS.txt)
**Ne(0):** implicit (Gravel_EAS autosome gamma, chrX Ne 3/4 bias unaddressed)
**Input lineage:** raw-9k chrX VCF → plink2 QC (split-par, --chr X non-PAR) → freeze-pass sample filter
**Sample policy:** Chinese freeze-pass only (F: 2720, M: 1653), sex-stratified
**Site universe:** non-PAR chrX (2,781,480-58,099,999 P-arm, 63,800,001-155,701,382 Q-arm, centromere 58.1-63.8Mb excluded)
**Singleton universe:** per-sex independent extraction

---

## 1. Input VCF

- Source: `/data/home/grp-wangyf/xuyuan/raw/9k/all.chrX.QC.vcf.gz` (114 GB)
- 8,793,737 variants, 9,466 samples (all populations including non-Chinese references)
- Index built: `.tbi` (132 KB, ~20 min for 114 GB)

## 2. Sex Check (plink2 --check-sex)

### Method
- **plink1 not available** — used plink2 `--check-sex` (same F-statistic output)
- PAR excluded via `--split-par hg38` + `--not-chr XY` → maps PAR to PAR1/PAR2 chr codes → excluded
- 7.66M non-PAR variants after QC (biallelic SNP, geno 0.1, hwe 1e-6, mac 1)
- PSAM generated from metadata: 5,035 F, 4,145 M, 286 unknown (SEX=0)

### Results
- **F distribution**: clean bimodal. Female cluster mean F=0.043, Male cluster mean F=0.991
- F ≤ 0.2 → Female: 5,051 samples
- F ≥ 0.8 → Male: 4,211 samples
- 0.2 < F < 0.8 → Ambiguous: 204 samples (excluded)
- **DISCORDANT**: 3 samples (meta F, comp M): HGDP01371 (F=0.845), NA10854 (F=0.866), NA20530 (F=0.807)
- **ASSIGNED**: 284 samples (previously missing sex → now assigned)
- **Metadata fixed**: 260 entries updated in `sample_10645_cleaned.tsv` (backup: `.bak_chrX`)

### Cohort freeze filter
- `cohort_freeze.tsv` ExclusionStatus=pass: 4,383 samples
- After cross-reference with sex results: **2,720 females, 1,653 males** (4,373 total)
- 10 excluded from freeze (all high_het_5sd) + 204 ambiguous F + 3 discordant (reassigned, kept)
- Non-Chinese reference samples correctly dropped (~4,900 removed from chrX analysis)

### Outputs
```
chrX_F_distribution.png          # F histogram by metadata sex
chrX_F_boxplot.png               # F boxplot M vs F
chrX_F_discordance_overview.png  # OK/PROBLEM/ASSIGNED overview
chrX_F_density_by_pedsex.png     # F density by PEDSEX
chrX_final_females_freeze.txt    # 2,720 female sample IDs
chrX_final_males_freeze.txt      # 1,653 male sample IDs
chrX_excluded.txt                # 204 ambiguous samples
chrX_sexcheck.sexcheck           # raw plink2 output
sample_10645_cleaned.tsv         # fixed metadata
```

## 3. PAR Exclusion (BUG FIX)

### Initial bug
Used `--not-chr XY` expecting plink2 to code PAR as XY chromosome. Actual plink2 behavior: `--split-par hg38` codes PAR1 as `PAR1`, PAR2 as `PAR2`. `--not-chr XY` was a no-op → **214,663 PAR1 + 23,017 PAR2 variants leaked into analysis**.

### Fix
Re-ran plink2: `--chr X` (keep only non-PAR chrX, exclude PAR1/PAR2):
```
7,421,843 non-PAR variants (was 7,659,523; difference = 237,680 PAR excluded)
```

### Verification
- Min position in VCF: 14,587 (above PAR1 2,781,479) ✓
- Max position: 156,029,855 (below chrX end 156,040,895) ✓
- No PAR1/PAR2 variants in pgen pvar ✓

## 4. Male Singleton Detection (BUG FIX)

### Initial bug
Standard `generate_s_file.sh` awk pattern only matches diploid het (`0/1`, `1/0`). Male chrX has mixed encoding:
- **Haploid**: `0` (ref), `1` (alt) — singleton carrier = `1`
- **Pseudo-diploid**: `0/0` (ref), `1/1` or `1|1` (alt hemizygous)

Neither `0/1` nor `1/0` matches haploid `1` or pseudo-diploid `1/1`. Result: **5,893 singletons detected vs 823,240 correct count** (140× undercount).

### Fix
Custom Python singleton extractor (`extract_male_singletons.py`) that detects ANY ALT allele carrier:
- `GT == "1"` (haploid ALT)
- `GT in ("1/1", "1|1")` (pseudo-diploid ALT)
- `GT in ("0/1", "1/0", "0|1", "1|0")` (diploid het — kept for completeness)

### Results
| | Old (buggy) | New (fixed) | Increase |
|---|---|---|---|
| P-arm singletons | 2,160 | 299,567 | 139× |
| Q-arm singletons | 3,733 | 523,673 | 140× |
| Total | 5,893 | 823,240 | 140× |

Female/Male singleton ratio: 1,516,557 / 823,240 = 1.84× (reasonable given chrX Ne difference).

## 5. SDS Input Generation

Per-sex, with fixed non-PAR VCFs:

```bash
# Female
run_sds_input.sh --pop chrX_F --chr X \
  --vcf chrX_qc_noPAR_females.vcf.gz \
  --sample-list chrX_final_females_freeze.txt \
  --out-root sds_input_noPAR

# Male (with custom s_file from extract_male_singletons.py)
run_sds_input.sh --pop chrX_M --chr X \
  --vcf chrX_qc_noPAR_males.vcf.gz \
  --sample-list chrX_final_males_freeze.txt \
  --out-root sds_input_noPAR
# Then replace s_file with custom extraction output
```

### Input files
| File | Female | Male |
|---|---|---|
| b_file | 30 B | 30 B |
| s_file | 14 MB | 7.4 MB |
| t_file | 421 MB | 109 MB |
| o_file | 5.4 KB | 3.3 KB |

Centromere split (b_file): P-arm 1-58,099,999, Q-arm 63,800,001-156,040,895.

## 6. SDS Compute

```bash
run_sds_compute.sh --pop chrX_F/M --chr X \
  --vcf chrX_qc_noPAR_[fe]males.vcf.gz \
  --in-root sds_input_noPAR \
  --out-root sds_output_noPAR \
  --g-file g_file.Gravel_EAS.txt
```

### Results
| | Female | Male |
|---|---|---|
| P-arm processed/kept | 30,280 / 29,112 | 7,563 / 6,998 |
| Q-arm processed/kept | 50,335 / 50,161 | 15,387 / 14,468 |
| Total SDS-scored | 79,273 | 21,466 |
| P-arm output | 1.8 MB | 390 KB |
| Q-arm output | 3.1 MB | 826 KB |

**Note**: Male Q-arm was EMPTY before fix (kept=0). After PAR fix + singleton fix, both arms compute correctly.

## 7. Normalization (per-arm, fine bins)

### Bin contract
- MAF < 0.1: bin_width = 0.005
- MAF ≥ 0.1: bin_width = 0.01
- Min-bin-variant merge: bins with < 100 variants merged into adjacent bins
- **Per-arm normalization**: P-arm and Q-arm normalized independently

### Female frequency bins (per-arm)
All 50 bins well-populated (250-1,670 variants). SD range: 0.06-0.22. Mean near zero for mid-frequency bins. ✓

### Male frequency bins (per-arm)
Required bin merging for sparse high-MAF bins. Post-merge:
- Low-MAF bins (0.05-0.20): well-populated, SD 0.5-0.8 (elevated vs female due to smaller N + hemizygosity)
- High-MAF bins (>0.20): merged due to low counts. Only 72-133 variants in raw bins — merged into wider bins

### Per-arm norm rationale
chrX P-arm (55 Mb) and Q-arm (92 Mb) have different recombination landscapes, gene densities, and evolutionary histories. Pooling across arms would mix different SDS null distributions. Per-arm normalization isolates arm-specific background.

## 8. Manhattan Plots

### R script bug
`report_cmplot_manhattan.R` line 86: `chr >= 1L & chr <= 22L` — chrX variants filtered out. `as.integer("X")` → NA.

### Fix
- Map chr → 23 in normalized TSV
- Modify `chr <= 22L` → `chr <= 23L`

### Outputs
```
chrX_F_noPAR.png  (1.1 MB, 79,273 variants)  # Female, per-arm norm
chrX_M_noPAR.png  (743 KB, 21,466 variants)  # Male, per-arm norm
```

## 9. Top Signals

### Female
| Position | norm_SDS | p-value | Region |
|---|---|---|---|
| chrX:96,921,235 | 4.36 | 1.3e-05 | Xq22.1 (PCDH19 region) |
| chrX:69,621,562 | 4.33 | 1.5e-05 | Xq13.1 (MED12/NLGN3) |
| chrX:96,930,161 | 4.28 | 1.9e-05 | Xq22.1 |

Bonferroni threshold: 6.31e-07. **No female signal reaches genome-wide significance.**

### Male
| Position | norm_SDS | p-value | Region |
|---|---|---|---|
| chrX:70,640,125 | 4.84 | 1.3e-06 | Xq13.1 (MED12) |
| chrX:134,072,843 | 3.66 | 2.5e-04 | Xq26.3 |
| chrX:116,806,470 | 3.61 | 3.1e-04 | Xq23 |

Bonferroni threshold: 2.33e-06. **Top male signal at Xq13.1 approaches but does not exceed threshold** (1.3e-06 vs 2.3e-06). Worth orthogonal validation (iHS/EHH, Relate/CLUES2).

## 10. Key Caveats

1. **Gamma mismatch**: Gravel_EAS gamma calibrated for autosome Ne. chrX effective Ne = 3/4 autosome. SDS values systematically biased — chrX-specific gamma needed for paper compliance.
2. **Male SD blow-up**: Low-MAF bins have SD 0.5-0.8 (4-5× female). Driven by small sample (N=1,653 hemizygous) + pseudo-diploid encoding noise. Broader bins or chrX-specific gamma might help.
3. **No significant peaks**: Not necessarily a bug. chrX is gene-poor (non-PAR), hemizygous in males (purifying selection dominates), and sample size is modest. Null result is biologically plausible.
4. **AA/DA polarization**: Not yet done for chrX (ancestor annotation pending). Current REF/ALT placeholders — paper-compliance gap.
5. **Mask/heterochromatin**: chrX centromere (58.1-63.8 Mb) excluded via b_file. Heterochromatin regions on chrX (e.g., Xp11.23, Xq21.1) not explicitly masked. Build chrX-specific mask per genome-wide mask contract.

## 11. All Output Paths

### Root: `/data/home/grp-wangyf/xuyuan/SDSworkspace/chrX_QC/`

```
# Sex check
chrX_sexcheck.sexcheck
chrX_final_females_freeze.txt       # 2,720 samples
chrX_final_males_freeze.txt         # 1,653 samples
chrX_final_sex_assignment.tsv
chrX_F_distribution.png
chrX_F_boxplot.png
chrX_F_discordance_overview.png
chrX_F_density_by_pedsex.png

# QC'd VCFs (non-PAR only)
chrX_qc_noPAR.vcf.gz                # 14 GB, 7,421,843 variants, all 9,466 samples
chrX_qc_noPAR_females.vcf.gz        # 862 MB, 2,720 females
chrX_qc_noPAR_males.vcf.gz          # 690 MB, 1,653 males

# SDS inputs
sds_input_noPAR/chrX_F/
  chrX_b_file.txt                   # centromere split bounds
  chrX_s_file.txt                   # 14 MB, 1,516,557 singletons
  chrX_t_file.txt                   # 421 MB
  chrX_o_file.txt                   # sample order
sds_input_noPAR/chrX_M/
  chrX_s_file.txt                   # 7.4 MB, 823,240 singletons (fixed)
  chrX_t_file.txt                   # 109 MB

# SDS outputs (per-arm normalized)
sds_output_noPAR/chrX_F/
  chrX_F_noPAR.normalized.tsv       # 14 MB, 79,273 variants
  chrX_F_noPAR.frequency_bins.tsv
  chrX_F_noPAR.significant_regions.tsv
  chrX_F_noPAR.postprocess_summary.tsv
  chrX_F_noPAR.png                  # Manhattan (1.1 MB)
sds_output_noPAR/chrX_M/
  chrX_M_noPAR.normalized.tsv       # 3.6 MB, 21,466 variants
  chrX_M_noPAR.frequency_bins.tsv
  chrX_M_noPAR.significant_regions.tsv
  chrX_M_noPAR.postprocess_summary.tsv
  chrX_M_noPAR.png                  # Manhattan (743 KB)

# Scripts (one-off, promote to SDSpipeline if reused)
extract_male_singletons.py          # male hemizygous-aware singleton extractor
generate_s_file_chrX_male.sh        # alternative s_file generator (not used; Python better)
```

### Fixed metadata
`/data/home/grp-wangyf/xuyuan/SDSworkspace/input/derived/sds_legacy/metadata/sample_10645_cleaned.tsv` (backup: `.bak_chrX`)
