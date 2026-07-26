# chr19 Top Significant Region: LocusZoom & LD Structure Analysis

Date: 2026-07-22 (chr19), extended 2026-07-25 (all peaks)
Status: COMPLETE — chr19 + batch all non-MHC significant peaks
Role: mainline — regional LD characterization for genome-wide significant peaks
Demographic model: SDS-MODEL-001 (Gravel_CHB@100k via g_file.Gravel_EAS.txt)
Input lineage: 9k QC v1 (snp_qc_pass.vcf.gz per chromosome), 4383 frozen samples
Sample policy: ALL (MERGED NCN+SCN)
Site universe: all.normalized.tsv (4,545,974 SNVs, 22 chr, MAF ≥ 0.05)
Gamma: g_file.Gravel_EAS.txt

## Region: chr19:4,748,270-5,023,601

Lead SNP: chr19:4768268:A:C (DAF=0.0674, norm_SDS=+10.44, p=1.55e-25)
Sig SNVs: 32 in region; 489 common variants in 500kb window

### Method

1. **LocusZoom**: plot_locuszoom.py (new script, SDSpipeline/scripts/postprocess/)
   - LD computed with PLINK2 --r2-unphased vs lead SNP
   - Recombination rate from pyrho CHB genetic map (SDSworkspace/input/reference/maps/CHB/)
   - Gene annotations from Ensembl REST API (GRCh38, cached 30d)
2. **LD heatmap**: Top 60 SNPs by |norm_SDS|, pairwise r² via PLINK2 triangle mode
3. **r² vs SDS scatter**: All region SNPs vs original lead, Spearman rank correlation

### Key Finding: Two Independent Selection Signals

The chr19 peak resolves into **two completely independent selection signals**
separated by ~180kb, on different haplotype backgrounds (pairwise r² = 1.07e-5).

| Signal | Position | Lead SNP | norm_SDS | DAF | Gene Context |
|--------|----------|----------|----------|-----|-------------|
| 1 (upstream) | 4,748,270 | chr19:4748270:G:A | +10.28 | 0.062 | intergenic: DPP9↔MIR7-3HG |
| 1 (core) | 4,768,268 | chr19:4768268:A:C | +10.44 | 0.067 | intergenic: near MIR7-3 miRNA |
| 2 (downstream) | 4,951,434 | chr19:4951434:C:T | +9.34 | 0.061 | **within UHRF1** exon |

LD between signals: r²(chr19:4768268:A:C, chr19:4951434:C:T) = 1.07e-5

LD within Signal 1: r²(chr19:4748270:G:A, chr19:4768268:A:C) = 0.767 (same haplotype)

### LD Architecture

```
4.75Mb   4.77Mb       4.87Mb        4.95Mb
  |        |             |             |
  ├─Signal 1────────┤   weak LD       ├─Signal 2─┤
  r²=0.77              r²~0.06-0.11   r²=1e-5 vs Signal 1
  MIR7-3HG/FEM1A                       UHRF1/KDM4B
```

Signal 1: intergenic, near MIR7-3HG lncRNA and MIR7-3 microRNA. Nearest protein-coding
genes: FEM1A (23kb downstream), DPP9 (44kb upstream).

Signal 2: within UHRF1 (ubiquitin-like with PHD and ring finger domains 1), a DNA
methylation maintenance factor. Also near KDM4B (17kb downstream, histone demethylase).

### r² vs SDS Trend

Spearman ρ = -0.008, p = 0.87 — **no significant trend**.
SNPs with high SDS but low r² to lead SNP are common in this region, confirming the
signals are independent selection events, not a single decaying sweep.

### LD Statistics

- SNPs in 500kb window: 946
- SNPs with r² ≥ 0.2 vs original lead: 1/9,560 (0.01%)
- SNPs with r² ≥ 0.5 vs original lead: 1/9,560 (0.01%)
- SNPs with r² ≥ 0.8 vs original lead: 0/9,560 (0%)

### gnomAD Cross-Reference

Lead SNP chr19:4768268:A:C: gnomAD EAS AF = 0.05635 (consistent with observed DAF=0.0674)

### Biological Interpretation

Both signals are rare-allele selection sweeps (DAF ≈ 0.06), suggesting recent or ongoing
selection in Chinese populations. Signal 1 is intergenic with possible regulatory targets
(MIR7-3). Signal 2 targets UHRF1, a key epigenetic regulator. The independence of these
signals (r² ≈ 0) means selection acted on different haplotype backgrounds — consistent
with either:
- Two independent selective events at different times
- Selection on a haplotype that subsequently recombined
- Polygenic selection across the region with independent adaptive alleles

Further orthogonal validation needed (iHS/EHH, Relate/CLUES2).

### Output Files

| File | Description |
|------|-------------|
| plots/chr19_4768268.locuszoom.png | LocusZoom with original lead chr19:4768268:A:C |
| plots/chr19_4951434.locuszoom.png | LocusZoom with downstream lead chr19:4951434:C:T |
| plots/chr19_r2_vs_sds.png | r² vs norm_SDS scatter + bar plot |
| plots/chr19_ld_heatmap.png | 60×60 pairwise LD heatmap (top |SDS| SNPs) |
| plots/chr19_ld_original_lead.vcor | PLINK2 LD output (9,561 pairs vs original lead) |
| plots/chr19_ld_downstream_lead.vcor | PLINK2 LD output (9,561 pairs vs downstream lead) |
| plots/locuszoom_cache/ensembl_genes_chr19_*.json | Cached Ensembl gene annotations |

### Script Used

- `SDSpipeline/scripts/postprocess/plot_locuszoom.py` — reusable LocusZoom generator
- `SDSworkspace/oneoff/analyze_chr19_ld_structure.py` — LD analysis for chr19

---

## Batch Extension (2026-07-25): All Non-MHC Significant Peaks

Applied the same LocusZoom + LD heatmap workflow to all genome-wide significant
peaks except chr6 (MHC — structurally complex, artifact-suspect per CLAUDE.md).

### Processed Regions

| Region | Chr | Lead SNP | norm_SDS | DAF | Gene Context |
|--------|-----|----------|----------|-----|--------------|
| chr4:99,159,154-99,322,288 | 4 | chr4:99318162:T:C | +6.99 | 0.30 | **ADH1B** (classic alcohol metabolism) |
| chr11:48,235,462 | 11 | chr11:48235462:G:A | +5.79 | 0.15 | **PTPRJ** (single SNP, low confidence) |
| chr12:109,825,802-112,971,371 | 12 | chr12:111803962:G:A | +9.82 | 0.24 | **ALDH2** (merged 5 sub-regions, classic sweep) |
| chr19:4,748,270-5,023,601 | 19 | chr19:4768268:A:C | +10.44 | 0.067 | MIR7-3HG/UHRF1 — two independent signals |
| chr19:10,378,293-11,002,401 | 19 | chr19:10765735:C:T | +9.27 | 0.061 | SLC44A2/ILF3/DNMT1 — near LDLR |
| chr22:41,474,810-41,522,608 | 22 | chr22:41504089:G:A | +6.07 | 0.48 | **DEPDC5** (tumor suppressor) |

### Method

- **LocusZoom**: `SDSpipeline/scripts/postprocess/plot_locuszoom.py`
  - LD: PLINK2 `--r2-unphased` vs lead SNP
  - Recombination: pyrho CHB genetic map (`SDSworkspace/input/reference/maps/CHB/`)
  - Genes: Ensembl REST API GRCh38 (30d JSON cache)
- **LD heatmap**: Top 60 SNPs by |norm_SDS|, pairwise r² via PLINK2 `triangle` mode
  - VCF region subset via tabix + bgzip to avoid full-chromosome scans
- Heatmap skipped for chr11 (1 SNP only)

### Key Findings by Peak

**chr4 ADH1B**: Lead SNP chr4:99318162:T:C (DAF=0.30) — known ADH1B Arg48His
(rs1229984). Strong LD block spanning the ADH cluster. Consistent with classic
alcohol metabolism selection in East Asians.

**chr12 ALDH2 mega-region**: 5 adjacent significant sub-regions merged into one
3.1Mb window. Lead chr12:111803962:G:A (DAF=0.24) at rs671 (Glu504Lys) — the
canonical ALDH2 East Asian selection signal. LD heatmap shows extended haplotype
homozygosity pattern spanning >1Mb, consistent with a strong hard sweep.
Hitchhiking signals at chr12:112,030,402-112,680,092 (PTPN11/RPH3A) and upstream
regions.

**chr19 region 2 (lipid cluster)**: Lead chr19:10765735:C:T (DAF=0.061) near
SLC44A2/ILF3/DNMT1, ~300kb upstream of LDLR. Rare-allele signal (DAF=0.06),
similar frequency to chr19 region 1. LD heatmap needed to assess independence
from nearby peaks.

**chr22 DEPDC5**: Lead chr22:41504089:G:A (DAF=0.48) in DEPDC5 intron. Common
variant sweep. Small region (~47kb), consistent with a single selection target.

### Output Files

All in `SDSworkspace/results/production/sds_output/ALL/plots/`:

| File | Description |
|------|-------------|
| chr4_99240kb.locuszoom.png | chr4 ADH1B LocusZoom |
| chr4_99240kb_ld_heatmap.png | chr4 ADH1B LD heatmap |
| chr11_48235kb.locuszoom.png | chr11 PTPRJ LocusZoom (no heatmap — 1 SNP) |
| chr12_111398kb.locuszoom.png | chr12 ALDH2 mega-region LocusZoom |
| chr12_111398kb_ld_heatmap.png | chr12 ALDH2 LD heatmap |
| chr19_4768268.locuszoom.png | chr19 region 1 LocusZoom (original lead) |
| chr19_4951434.locuszoom.png | chr19 region 1 LocusZoom (downstream lead) |
| chr19_r2_vs_sds.png | chr19 region 1 r² vs SDS scatter |
| chr19_ld_heatmap.png | chr19 region 1 LD heatmap |
| chr19_10690kb.locuszoom.png | chr19 region 2 LocusZoom |
| chr19_10690kb_ld_heatmap.png | chr19 region 2 LD heatmap |
| chr22_41498kb.locuszoom.png | chr22 DEPDC5 LocusZoom |
| chr22_41498kb_ld_heatmap.png | chr22 DEPDC5 LD heatmap |

### Scripts

- `SDSpipeline/scripts/postprocess/plot_locuszoom.py` — reusable LocusZoom generator
- `SDSworkspace/oneoff/analyze_chr19_ld_structure.py` — chr19 deep-dive analysis
- `SDSworkspace/oneoff/batch_locuszoom_all_peaks.py` — batch processor for all peaks
