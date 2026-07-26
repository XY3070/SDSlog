# LD-Based Signal Decomposition & Functional Annotation Pipeline

Date: 2026-07-26
Status: COMPLETE (v2 — AlphaMissense + regulatory layers added; EAS eQTL gap documented)
Role: mainline — LD split + multi-layer functional annotation for independent selection signals

## Pipeline Overview

```
significant_regions.tsv + all.normalized.tsv
        │
        ▼
┌─────────────────────────────────────────────────────────┐
│ Phase 1: LD Split (connected components, r² threshold)  │
│   • Tabix-extract region VCF (±50kb flanks)             │
│   • PLINK2 triangle mode pairwise r² (TSV SNPs only)    │
│   • BFS connected components at r² > 0.1                │
│   • Sensitivity check at r² > 0.05                      │
│   • Components with genome-wide sig SNPs always kept    │
│   • Each component = 1 independent selection signal     │
└─────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────┐
│ Phase 2: VEP Annotation (lead SNP + LD proxies r²>0.8)  │
│   Source: VEP 114.1 (WGS conda env)                     │
│   Data: offline GRCh38 cache (Ensembl v114)             │
│     /data/home/grp-wangyf/leiyao/Database/Ensembl/VEP/ │
│   Reference: Broad hg38 FASTA                           │
│   Plugins: CADD v1.6 (whole_genome_SNVs.tsv.gz, 82GB)  │
│   Output: JSON (gene symbol, consequence, SIFT,         │
│           PolyPhen, CADD phred, HGVS, MANE)             │
│   Fallback: Ensembl REST VEP endpoint                   │
└─────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────┐
│ Phase 3: Non-coding Follow-up                           │
│   A. Ensembl Regulatory Build (REST API)                │
│      • Source: rest.ensembl.org                         │
│      • Query: ±500bp around lead SNP                    │
│      • Features: promoter, enhancer, CTCF, open chromatin│
│      • NOT population-specific — universal annotation   │
│                                                         │
│   B. eQTLGen (EUR reference only — see caveat below)    │
│      • Source: eQTLGen consortium 2019                  │
│      • Cohort: ~31,684 EUR-ancestry blood samples       │
│      • Labeled: "eQTLGen_EUR" — NOT definitive for CHN  │
│      • ⚠ PAPER COMPLIANCE GAP (see below)              │
│                                                         │
│   C. CADD scores (deleteriousness, not population-spec) │
│      • CADD phred ≥ 15 flagged as potentially damaging  │
│                                                         │
│   D. dbSNP rsID cross-reference                         │
└─────────────────────────────────────────────────────────┘
```

## Annotation Sources — Provenance & Caveats

### VEP (Variant Effect Predictor) — PRIMARY, TRUSTED

| Property | Value |
|----------|-------|
| Version | 114.1 (WGS conda env) |
| Cache | Ensembl v114, GRCh38, offline |
| Gene models | Ensembl canonical + MANE select |
| Consequence | Sequence Ontology terms |
| SIFT/PolyPhen | Protein structure-based (NOT population-specific) |
| CADD | Combined Annotation Dependent Depletion v1.6 |
| Reference | Broad hg38 |
| **Population specificity** | **None — universal. Appropriate for any cohort.** |

VEP annotates where a variant falls relative to gene models and predicts
functional impact based on protein structure (SIFT/PolyPhen) or integrative
scores (CADD). These scores are NOT population-specific and are valid for
Chinese cohorts.

CADD scores measure variant deleteriousness by integrating diverse annotations
(conservation, epigenetic, functional). CADD phred ≥ 15 is a commonly used
threshold for potentially damaging variants. The CADD plugin data file
(`whole_genome_SNVs.tsv.gz`, 82GB) provides pre-computed scores for all
possible GRCh38 SNVs.

### Ensembl Regulatory Build — TRUSTED, UNIVERSAL

| Property | Value |
|----------|-------|
| Source | Ensembl REST API (`/overlap/region/human/`) |
| Data | Ensembl Regulatory Build (ENCODE, Roadmap, etc.) |
| Features | promoter, enhancer, CTCF binding, open chromatin, TF binding |
| **Population specificity** | **None — universal. Appropriate for any cohort.** |

The Ensembl Regulatory Build annotates regulatory elements based on
multi-cell-type epigenomic data from ENCODE, Roadmap Epigenomics, and
other sources. While the cell types may not perfectly match the tissue
of selection, regulatory element annotations are generally conserved
across populations.

### eQTLGen — ⚠ EUR REFERENCE ONLY, NOT DEFINITIVE FOR CHN

| Property | Value |
|----------|-------|
| Source | eQTLGen consortium (2019) |
| Samples | ~31,684 blood samples |
| Ancestry | **European (EUR)** |
| File | `2019-12-11-cis-eQTLsFDR0.05-...txt.gz` (323 MB, 10.5M rows) |
| **Population specificity** | **EUR — NOT appropriate as primary evidence for Chinese cohorts** |
| Usage in catalog | Labeled `eQTLGen_EUR` — strictly as reference flag |

**Paper-compliance gap:** Chinese/Asian-specific eQTL data is NOT currently
available in this environment. eQTL effects differ across populations due to
LD differences and population-specific regulatory variants. A European eQTL
hit does NOT confirm regulatory function in Chinese populations.

**Resolution options (for paper):**
1. Query GTEx v8 Asian-ancestry subset (limited N but available)
2. Use East Asian-specific eQTL resources if accessible (e.g., Chinese blood
   eQTL studies, GTEx portal filtered to EAS)
3. Use TWAS/PrediXcan models trained on Asian populations
4. Flag as limitation: "regulatory annotation is based on universal (Ensembl
   Regulatory Build) and European (eQTLGen) resources; Chinese-specific
   eQTL validation remains future work"

## LD Split Methodology

### Why Connected Components (r²)?

Three methods were considered:
1. Gabriel et al. (D' confidence intervals) — rejected: D' inflated by rare
   alleles, leaves SNPs unassigned, designed for GWAS tag SNPs not selection
   signal decomposition.
2. Connected components (r² threshold) — **chosen**: r² directly measures
   allele dosage correlation — "do these alleles travel together on haplotypes?"
   Each component = one independent selection haplotype.
3. Community detection (Louvain/Leiden) — rejected: adds resolution parameter
   without biological justification; selection sweeps produce simple haplotype
   blocks, not hierarchical communities.

### Threshold Selection

- **r² > 0.1** (primary): captures core haplotypes while resolving independent
  signals. At this threshold, chr19 region 1 yields 6 components including the
  2-SNP core haplotype (MIR7-3, SDS=10.44).
- **r² > 0.05** (sensitivity): more inclusive. At this threshold, chr19
  region 1 collapses to 2 components — the upstream block and the UHRF1 block.

The r² > 0.1 threshold is the primary analysis because:
1. It correctly separates the known independent signals (MIR7-3 vs UHRF1)
2. It captures the small-but-significant core haplotype (2 SNPs at r²=0.77)
3. At r² > 0.05, distinct signals merge into fewer, less interpretable blocks

### Minimum Component Size

Components containing genome-wide significant SNPs (p < Bonferroni) are
**always kept** regardless of size. This preserves the chr19:4768268/4748270
2-SNP core haplotype that would otherwise be dropped.

Other components require ≥ 3 SNPs and max |SDS| > 3 to be reported.

## Results Summary (15 independent signals, 10 non-MHC peaks)

### Coding Variants (missense)

| Signal | Gene | rsID | Consequence | SDS | CADD |
|--------|------|------|-------------|-----|------|
| chr12 ALDH2 core | ALDH2 | rs671 | Glu504Lys missense | +9.82 | TBD |
| chr4 ADH1B | ADH1B | rs1229984 | Arg48His missense | +6.99 | TBD |

Note: CADD phred scores pending re-run with CADD plugin. ALDH2 rs671
(Glu504Lys) and ADH1B rs1229984 (Arg48His) are the classic East Asian
alcohol metabolism selection targets, independently confirmed by our SDS scan.

### Non-coding Signals (top by |SDS|)

| Signal | Gene | Consequence | SDS | Regulatory | eQTLGen_EUR |
|--------|------|-------------|-----|------------|-------------|
| chr19 MIR7-3 | MIR7-3 | upstream | +10.44 | no | yes |
| chr19 UHRF1 | UHRF1 | intronic | +9.34 | no | yes |
| chr19 DNM2 | DNM2 | intronic | +9.27 | no | yes |
| chr12 NAA25 | NAA25 | intronic | +9.24 | no | yes |
| chr19 PLIN3 | PLIN3 | upstream | +8.57 | yes | yes |
| chr12 TRPV4 | TRPV4 | intronic | +6.22 | no | yes |
| chr12 OAS3 | OAS3 | 3' UTR | +6.13 | yes | yes |
| chr12 ATP2A2 | ATP2A2 | intronic | +6.12 | no | yes |
| chr22 ACO2 | ACO2 | intronic | +6.07 | yes | yes |
| chr11 intergenic | — | intergenic | +5.79 | no | yes |
| chr19 DPP9 | DPP9 | intronic | +5.47 | no | yes |
| chr19 ARRDC5 | ARRDC5 | upstream | +4.00 | yes | yes |

### chr12 ALDH2 Sweep Architecture

The chr12 mega-region resolves into 5 connected components (r² > 0.1),
revealing the spatial architecture of the ALDH2 sweep:

```
109.8Mb    110.3Mb      110.6-111.8Mb      112.0-112.7Mb    112.9-113.0Mb
  |           |              |                   |                |
TRPV4       ATP2A2        ALDH2 core          NAA25             OAS3
SDS=+6.22   SDS=+6.12     SDS=+9.82          SDS=+9.24        SDS=+6.13
intronic    intronic      missense (rs671)    intronic         3'UTR
```

The NAA25 signal (SDS=+9.24) is only 220kb from ALDH2 and likely represents
hitchhiking on the same sweep, but the connected-components analysis separates
them because they are on different r²>0.1 LD backgrounds. The ATP2A2 signal
is just 2 SNPs (kept due to genome-wide significance) and may be a weak
independent signal or a LD-break artifact.

### chr19 Region 1: Six Independent Signals

```
4.71Mb   4.77Mb    4.87Mb     4.89-4.90Mb   4.95Mb
  |        |         |           |             |
DPP9    MIR7-3    PLIN3       ARRDC5        UHRF1
SDS=+5.5  SDS=+10.4 SDS=+8.6  SDS=+4.0     SDS=+9.3
intronic upstream  upstream   upstream      intronic
```

The two strongest signals (MIR7-3 and UHRF1) are separated by 180kb with
r²=1e-5 — completely independent sweeps.

## Scripts

| Script | Location | Role |
|--------|----------|------|
| `ld_split_and_annotate.py` | `SDSpipeline/scripts/postprocess/` | Main pipeline |
| `plot_locuszoom.py` | `SDSpipeline/scripts/postprocess/` | Regional LocusZoom (step 0) |
| `batch_locuszoom_all_peaks.py` | `SDSworkspace/oneoff/` | Batch LocusZoom + heatmap |
| `analyze_chr19_ld_structure.py` | `SDSworkspace/oneoff/` | chr19 deep-dive |

## Paper-Compliance Gaps

| Gap | Severity | Resolution |
|-----|----------|------------|
| **Chinese-specific eQTL data** | HIGH | eQTLGen (EUR) used as reference flag only. Need Chinese blood eQTL or GTEx EAS subset. |
| **CADD scores pending** | LOW | Plugin configured, data file present. Scores will be included in next re-run. |
| **dbSNP rsID resolution** | LOW | Working but slow (tabix on full dbSNP). Some rsIDs not resolving — may need format fix. |
| **VEP intergenic annotation** | LOW | Intergenic variants get minimal annotation. Regulatory Build fills some gaps but not all. |
| **Tissue-specific regulatory annotation** | MEDIUM | Ensembl Regulatory Build is multi-tissue. Tissue-matched annotation (e.g., liver for ALDH2/ADH1B) would strengthen interpretation. |

## Next Steps

1. Re-run full pipeline with CADD plugin active → update catalog with CADD scores
2. Investigate GTEx v8 Asian-ancestry eQTL data availability
3. Tissue-specific regulatory annotation for liver (ADH1B/ALDH2) and brain/immune (MHC)
4. Orthogonal validation (iHS/EHH, Relate/CLUES2) for top non-coding signals

---

## v2 Update (2026-07-26, late): Enhanced Annotation Layers

### Added Annotation Sources

| Layer | Source | Status |
|-------|--------|--------|
| **SIFT** | VEP built-in (protein conservation) | ✓ Working |
| **PolyPhen** | VEP built-in (protein structure) | ✓ Working |
| **AlphaMissense** | VEP plugin (DeepMind, 642MB hg38 data) | ✓ Working |
| **CADD v1.7** | VEP plugin (82GB, needs bgzip+tabix) | ⏳ Indexing in background |
| **phastCons** | VEP Conservation plugin (21GB BigWig) | ⏳ Pending format verification |
| **Regulatory layers** | Ensembl Regulatory Build REST API | ✓ Working |
| **Motif disruption** | FunMotifs plugin (needs JASPAR DB) | ❌ Not available |
| **Hi-C/ABC** | ABC model data | ❌ Not available locally |
| **Chinese/EAS eQTL** | Population-specific eQTL | ❌ PAPER-COMPLIANCE GAP |

### Regulatory Layer Classification

Ensembl Regulatory Build features classified by `description` field:

| Description | Epigenomic Signal | Layer |
|-------------|-------------------|-------|
| `open_chromatin_region` | DNase-seq / ATAC-seq | Chromatin accessibility |
| `promoter` | H3K4me3, H3K9ac | Histone modification |
| `enhancer` | H3K4me1, H3K27ac | Histone modification |
| `CTCF_binding_site` | CTCF ChIP-seq | Insulator/TAD boundary |
| `TF_binding_site` | TF ChIP-seq | Transcription factor binding |

### Final Results: 15 Independent Signals

#### Coding Signals

| Signal | Gene | HGVS | SIFT | PolyPhen | AlphaMissense | SDS |
|--------|------|------|------|----------|---------------|-----|
| chr12 ALDH2 core | ALDH2 | Glu504Lys | 0 (deleterious) | 0.709 (possibly_damaging) | **0.886 (likely_pathogenic)** | +9.82 |
| chr4 ADH1B | ADH1B | His48Arg | 1 (tolerated) | 0 (benign) | 0.238 (likely_benign) | +6.99 |

**ADH1B note:** SIFT/PolyPhen/AlphaMissense all predict benign — the Arg48His
substitution is structurally tolerated. Selection likely acts on the enzymatic
phenotype (alcohol metabolism efficiency) rather than protein damage. This is
consistent with the known East Asian selection at ADH1B.

**ALDH2 note:** All three predictors flag Glu504Lys as damaging/pathogenic.
This variant causes ALDH2 deficiency (alcohol flush reaction). The strong SDS
signal (+9.82) confirms this as the strongest protein-damaging selection target
in our Chinese cohort.

#### Non-coding Signals with Regulatory Overlap

| Signal | Gene | Regulatory Features | SDS |
|--------|------|---------------------|-----|
| chr19 PLIN3 | PLIN3 | **promoter + enhancer** flanking lead SNP | +8.57 |
| chr19 ARRDC5 sig5 | ARRDC5 | **enhancer** overlap | +4.00 |
| chr12 OAS3 | OAS3 | **enhancer** overlap (lead SNP within) | +6.13 |
| chr22 ACO2 | ACO2 | **enhancer** proximal | +6.07 |

**chr19 PLIN3 (SDS=+8.57):** Lead SNP chr19:4867689:G:T sits in a 46bp gap
between an enhancer (ENSR19_CH33, 4,866,242-4,867,642) and a promoter
(ENSR19_BZ49, 4,867,643-4,868,184). This is a classic regulatory junction
region — selection may affect PLIN3 expression by altering enhancer-promoter
communication or TF binding at the boundary.

#### Top Non-coding Signals (no regulatory annotation)

| Signal | Gene | Consequence | SDS | DAF |
|--------|------|-------------|-----|-----|
| chr19 MIR7-3 | MIR7-3 | upstream (intergenic) | +10.44 | 0.067 |
| chr19 UHRF1 | UHRF1 | intronic | +9.34 | 0.061 |
| chr19 DNM2 | DNM2 | intronic | +9.27 | 0.061 |
| chr12 NAA25 | NAA25 | intronic | +9.24 | 0.245 |

These signals have no Ensembl Regulatory Build annotation in the immediate
vicinity (±500bp). Further investigation needed:
- MIR7-3: core haplotype is only 2 SNPs — may require expanded regulatory
  search (±5kb) or long-range interaction data (Hi-C)
- UHRF1: large intronic block (242 SNPs) — likely regulatory element within
  the intron not captured by Ensembl Regulatory Build; needs ENCODE cCRE
  cross-reference
- NAA25: strong hitchhiking signal from ALDH2 sweep — not an independent
  selection target

### EAS eQTL Status

**eQTLGen (EUR) retained as reference flag only**, labeled `eQTLGen(EUR_ref_only)`.
All 13 non-coding signals have eQTLGen hits but these are NOT definitive for
Chinese interpretation.

**Paper-compliance gap remains.** External EAS eQTL resources needed:
1. GTEx v8 portal → filter to Asian ancestry donors
2. eQTL Catalogue (ebi.ac.uk/eqtl) → population-stratified queries
3. Published Chinese blood eQTL studies (e.g., Chen et al. 2023)

### Pending Infrastructure

| Item | Status | Action |
|------|--------|--------|
| CADD bgzip+tabix | Indexing (82GB, bgzip running) | Will enable `--plugin CADD` for phred scores |
| phastCons BigWig | Needs VEP plugin format verification | May work with current file; needs testing |
| Motif databases | Not available | JASPAR/HOCOMOCO download needed |
| Hi-C/ABC | Not available | External resource |

### Script Version

`SDSpipeline/scripts/postprocess/ld_split_and_annotate.py` v2
- VEP 114.1 with AlphaMissense plugin
- Ensembl Regulatory Build with layer classification
- eQTLGen explicitly labeled as EUR reference only
- Connected-component LD split at r²>0.1
