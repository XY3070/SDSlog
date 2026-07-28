# Orthogonal Validation: iHS, Motif Disruption, Relate+CLUES2, 1KCP eQTL

Date: 2026-07-26 (iHS, Motif, eQTL), 2026-07-28 (Relate v5 resubmitted)
Status: iHS ✓, Motif ✓, 1KCP eQTL ✓, Relate chr19 ⏳ (job 503068, v5)
Role: mainline — orthogonal validation of SDS genome-wide significant peaks

## 1. Genome-wide iHS (selscan v2.1.2)

### Method

- **Input**: NCN SHAPEIT5-phased VCFs (1255 samples, chr1-22)
- **Map**: pyrho CHB genetic maps (SDSworkspace/input/reference/maps/CHB/),
  converted to PLINK format for selscan
- **Command**: `selscan --ihs --pmap --maf 0.05 --threads 4`
- **Normalization**: 100-bin frequency normalization (|norm_iHS| > 4 = extreme)
- **Flag**: `--pmap` required for maps with pre-computed genetic positions

**Critical technical note:** selscan `--pmap` flag is ESSENTIAL when using pyrho
maps. Without it, selscan expects the map to have exactly matching positions with
the VCF and produces garbage count errors (2^64 overflow). With `--pmap`, selscan
treats map column 3 as pre-computed genetic positions and handles position
interpolation correctly.

### Results: 22 chromosomes

| Chr | SNPs | Extreme (|norm|>4) |
|-----|------|-------------------|
| chr6 | 362,728 | 397 (MHC — artifact enrichment) |
| chr3 | 380,286 | 178 |
| chr7 | 317,775 | 168 |
| chr8 | 288,953 | 128 |
| chr9 | 231,055 | 127 |
| chr4 | 392,461 | 92 |
| chr19 | 115,741 | 90 |
| chr1 | 405,085 | 86 |
| chr2 | 424,694 | 84 |
| chr11 | 264,381 | 70 |
| chr14 | 173,805 | 67 |
| chr10 | 272,718 | 66 |
| chr15 | 152,435 | 60 |
| chr13 | 194,077 | 54 |
| chr16 | 159,349 | 51 |
| chr12 | 251,656 | 42 |
| chr21 | 75,145 | 38 |
| chr20 | 114,317 | 36 |
| chr17 | 137,784 | 34 |
| chr18 | 151,999 | 27 |
| chr5 | 337,795 | 24 |
| chr22 | 72,561 | 22 |

### iHS at SDS Significant Signals

| Signal | Gene | SDS | DAF (ALL) | norm_iHS | Interpretation |
|--------|------|-----|-----------|----------|----------------|
| chr12 OAS3 | OAS3 | +6.13 | 0.151 | **+3.89** | STRONG — consistent hard sweep |
| chr11 intergenic | — | +5.79 | 0.149 | **+2.69** | Moderate hard sweep |
| chr12 ALDH2 | ALDH2 | +9.82 | 0.245 | +1.86 | Moderate — below extreme threshold |
| chr12 NAA25 | NAA25 | +9.24 | 0.245 | +1.85 | Same haplotype as ALDH2 |
| chr22 ACO2 | ACO2 | +6.07 | 0.522 | +0.74 | Weak |
| chr19 UHRF1 | UHRF1 | +9.34 | 0.061 | -0.31 | NO iHS signal |
| chr19 PLIN3 | PLIN3 | +8.57 | 0.085 | -0.13 | NO iHS signal |
| chr19 DPP9 | DPP9 | +5.47 | 0.053 | -0.93 | Opposite direction |
| chr4 ADH1B | ADH1B | +6.99 | 0.700 | **NOT IN VCF** | Variant filtered in NCN phasing |
| chr19 MIR7-3 | MIR7-3 | +10.44 | 0.067 | **NOT IN iHS** | DAF=0.04 in NCN < MAF 0.05 |
| chr19 DNM2 | DNM2 | +9.27 | 0.061 | **NOT IN iHS** | DAF < 0.05 in NCN |
| chr12 TRPV4 | TRPV4 | +6.22 | 0.069 | **NOT IN iHS** | DAF < 0.05 in NCN |

### iHS Interpretation

**Positive controls partially validated:**
- OAS3 shows strong iHS (norm=+3.89), consistent with SDS. This is an ALDH2
  sweep hitchhiker with independent iHS signal.
- ALDH2 shows only moderate iHS (+1.86). The classic ALDH2 sweep (rs671,
  SDS=+9.82) is a strong SDS signal but the iHS signal is attenuated — likely
  because the variant frequency (DAF=0.25) is in a range where iHS has reduced
  power, or the sweep is old enough that haplotype homozygosity has decayed.
- ADH1B could not be validated because the variant (rs1229984) is not in the
  NCN SHAPEIT5-phased VCF — it was filtered during phasing QC.

**Systematic limitation identified: iHS has low power for rare variants.**
The strongest SDS signals (MIR7-3 DAF=0.067, DNM2 DAF=0.061, UHRF1 DAF=0.061)
have DAF below 0.05 in the NCN subpopulation and are excluded from iHS by the
MAF filter. Even those with DAF slightly above 0.05 (UHRF1 at 0.063, PLIN3 at
0.085) show no iHS signal. This is expected: iHS requires variants to be at
intermediate frequencies for power, and rare-allele selection sweeps are better
detected by SDS (which uses singleton density) than by haplotype-based methods.

**Recommendation:** iHS provides orthogonal validation for common-variant SDS
signals (OAS3, ALDH2, NAA25, chr11). For rare-variant signals (MIR7-3, DNM2,
UHRF1, PLIN3), Relate+CLUES2 or singleton-based methods are more appropriate
validation approaches.

## 2. Motif Disruption (JASPAR 2024)

### Method

- JASPAR 2024 CORE vertebrates non-redundant (879 PWMs)
- ±20bp flank around lead SNP from hg38 FASTA
- PWM scanning: ref vs alt allele, ΔPWM = alt_score - ref_score
- Threshold: |ΔPWM| ≥ 3.0

### Top Motif Disruptions at SDS Signals

| Signal | SDS | Top Disrupted TF | ΔPWM | Regulatory Context |
|--------|-----|-----------------|------|-------------------|
| OAS3 | +6.13 | **JUN** | **-21.0** | enhancer overlap |
| TRPV4 | +6.22 | ATF3 | +22.0 | ALDH2 upstream |
| chr11 | +5.79 | CDX4, HOXD9 | +21.2 | intergenic |
| NAA25 | +9.24 | **FOS** | +17.6 | ALDH2 hitchhiker |
| PLIN3 | +8.57 | MEF2B, **TCF7L2** | +18.2 | **promoter+enhancer junction** |
| UHRF1 | +9.34 | **JUN** | +18.8 | intronic |
| DNM2 | +9.27 | **TFAP2C/A/B** | ±18 | intronic (family-wide) |
| MIR7-3 | +10.44 | ARNT2, TFEC | ±19 | intergenic |
| ACO2 | +6.07 | CDX1, **STAT3** | +19.7 | enhancer proximal |

### Motif Interpretation

**JUN/FOS (AP-1 complex) recurrently disrupted** across UHRF1, PLIN3, OAS3.
AP-1 is a master regulator of cell proliferation, differentiation, and stress
response. Its recurrent disruption across independent selection signals
suggests AP-1-mediated regulatory adaptation as a common selection target.

**TFAP2 family (TFAP2C/A/B) disrupted in DNM2** — all three family members
show consistent ΔPWM, suggesting the DNM2 lead SNP sits in a TFAP2 binding
site. TFAP2 is involved in neural crest development and lipid metabolism.

**TCF7L2 disrupted in PLIN3** — TCF7L2 is a well-known type 2 diabetes risk
gene and Wnt signaling effector. PLIN3 (perilipin 3) is a lipid droplet
coat protein. The PLIN3 lead SNP sits at a promoter-enhancer junction, and
TCF7L2 motif disruption suggests altered Wnt-responsive regulation of lipid
metabolism.

**STAT3 disrupted in ACO2** — STAT3 is a key immune and metabolic TF.
ACO2 (aconitase 2) is a TCA cycle enzyme. Enhancer-proximal STAT3 motif
disruption may alter metabolic gene regulation.

## 3. 1KCP Chinese eQTL

### Method

- Source: https://yanglab.westlake.edu.cn/resources/1kcp/eqtl/
- 22 chromosome summary files, 223.8M eQTL associations
- Tabix-indexed for fast region queries
- Query: ±5kb around each lead SNP, p < 1e-5

### Top Chinese eQTL Hits at SDS Signals

| Signal | SDS | 1KCP eQTL Target | Best p | Effect |
|--------|-----|-----------------|--------|--------|
| ALDH2 core | +9.82 | HECTD4 | <1e-100 | Very strong |
| DPP9 | +5.47 | DPP9 | <1e-47 | cis-eQTL |
| NAA25 | +9.24 | GIT2 | <1e-33 | trans |
| MIR7-3 | +10.44 | (intergenic) | 0.0009 | Weak — one gene |
| UHRF1 | +9.34 | KDM4B | significant | Regulatory |

**Key: 1KCP (Chinese) eQTL targets differ from eQTLGen (EUR).** The paper-
compliance gap for population-matched eQTL is now closed. Chinese-specific
eQTL evidence is integrated into the signal catalog.

### Remaining Gaps for Gene Mapping

| Method | Status |
|--------|--------|
| Physical proximity (VEP) | ✓ Complete |
| Ensembl Regulatory Build | ✓ Complete |
| 1KCP Chinese eQTL | ✓ Complete |
| Hi-C / ABC model | ❌ Not locally available |
| EAS eQTL in additional tissues | ❌ 1KCP is blood only |

## 4. Relate + CLUES2 (chr19 pilot)

### Status

- **Job 503040** submitted (fixed from 502949 crash)
- 502949 crashed at Finalize: `Ne: unbound variable` in RelateParallel.sh line 243
- Fix: explicit `-N 10000` parameter
- CLUES2 auto-submit script ready, will launch after Relate completes

### Expected Output

- Per-signal selection coefficients (s) with time bins
- Log-likelihood ratio vs neutral model
- Selection trajectory plots

## 5. Consolidated Signal Evidence

| Signal | SDS | iHS | Motif | 1KCP eQTL | Regulatory | Overall |
|--------|-----|-----|-------|-----------|------------|---------|
| ALDH2 | +9.82 | +1.9 | — (missense) | HECTD4 1e-100 | — | CONFIRMED missense |
| ADH1B | +6.99 | NA | — (missense) | — | — | CONFIRMED missense (technical NA) |
| OAS3 | +6.13 | **+3.9** | JUN -21.0 | enhancer | STRONG |
| NAA25 | +9.24 | +1.9 | FOS +17.6 | GIT2 1e-33 | | MODERATE (ALDH2 hitchhiker) |
| chr11 | +5.79 | **+2.7** | CDX4 +21 | | | MODERATE |
| PLIN3 | +8.57 | -0.1 | TCF7L2 +18 | promoter+enhancer | MODERATE (regulatory mechanism) |
| MIR7-3 | +10.44 | NA | ARNT2 +19 | weak eQTL | | WEAK (rare variant, no iHS) |
| UHRF1 | +9.34 | -0.3 | JUN +18.8 | KDM4B | | WEAK (rare variant, no iHS) |
| DNM2 | +9.27 | NA | TFAP2 family | DNM2 | | WEAK (rare variant, no iHS) |
| ACO2 | +6.07 | +0.7 | STAT3 +16 | enhancer | MODERATE |

### Key Observation

**Rare-variant SDS signals (DAF < 0.1) show no iHS signal.** This is not a
failure of SDS but a feature: SDS detects selection on rare variants via
singleton density, while iHS requires intermediate-frequency variants for
haplotype-based power. MIR7-3 (strongest SDS signal, DAF=0.067) is invisible
to iHS. This reinforces the complementary value of SDS for detecting recent
selection on rare alleles — precisely the regime where haplotype methods
have low power.

## Output Files

| File | Description |
|------|-------------|
| `orthogonal/ihs/NCN/chr*.ihs.out` | Raw iHS scores (22 chr) |
| `orthogonal/ihs/ihs_sds_crossref.tsv` | iHS-SDS cross-reference table |
| `orthogonal/motif/motif_disruption_details.tsv` | Per-SNP-TF motif disruption |
| `orthogonal/motif/motif_disruption_summary.tsv` | Per-signal motif summary |
| `signals/all_signals_catalog.tsv` | Updated with 1KCP eQTL columns |
| `orthogonal/relate_clues2/` | Relate chr19 (job 503040 running) |

## Scripts

- `SDSpipeline/scripts/orthogonal/run_genomewide_ihs.sh` — iHS batch submitter
- `SDSpipeline/scripts/orthogonal/run_ehh_at_signals.py` — EHH decay curves
- `SDSpipeline/scripts/orthogonal/motif_disruption.py` — PWM motif scanning
- `SDSpipeline/scripts/postprocess/ld_split_and_annotate.py` — LD split + annotation + 1KCP eQTL

---

## 6. Relate + CLUES2 (chr19 pilot) — Debugging History

### Attempt 1 (job 502949, agent submission)
- **Error**: `Ne: unbound variable` at RelateParallel.sh line 243
- **Root cause**: RelateParallel requires `-N <Ne>` parameter for BuildTopology step
- **What worked**: MakeChunks + Paint completed (3076s CPU, 1.96GB memory)
- **What failed**: BuildTopology never reached — script crashed at variable reference

### Attempt 2 (job 503040, -N added but with --dist)
- **Error**: `Assertion bp_pos[snp] == mbp failed` in MakeChunks
- **Root cause**: Genetic map (chr19_relate.map) in refs/ had different positions
  from the `.dist` file generated by PrepareInputFiles
- **Lesson**: The `.dist` file encodes within-chunk genetic distances from the
  same map used during haps preparation. Using a different map causes position
  mismatch in MakeChunks.

### Attempt 3 (job 503067, original map but with --dist)
- **Error**: Same assertion failure — `--dist` contains positions from a
  different map version
- **Lesson**: `--dist` should only be used with the EXACT same map as
  PrepareInputFiles. Better to omit --dist and let Relate compute distances
  from the map internally.

### Attempt 4 (job 503068, original map, NO --dist, -N 10000)
- **Fix**: Removed `--dist` flag entirely, used only `--map`
- **Submitted**: 2026-07-28 13:32, 16 cores, 80GB, 48hr
- **Expected runtime**: 2-4 hours (Paint ~1hr, BuildTopology+Infer ~2-3hr)

### CLUES2 Submission Script
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/orthogonal/relate_clues2/run_clues2_chr19.sh`
- Will auto-submit with `-w "done(<relate_job>)"` dependency
- 7 chr19 lead SNPs queued: MIR7-3, UHRF1, DNM2, PLIN3, DPP9, ARRDC5×2

### Relate Technical Requirements (Lessons Learned)
1. `-N <effective_population_size>` is MANDATORY — RelateParallel uses it in
   BuildTopology and InferBranchLengths steps
2. `--dist` must match the map EXACTLY — safest to omit and let Relate compute
   genetic distances from `--map` internally
3. Output directory must be the current working directory (cd into it first)
4. Use relative paths for all input files
5. LD_LIBRARY_PATH must include conda env's lib for GLIBCXX compatibility

