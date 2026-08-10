# p<5e-5 Locus Table with RePoS (NyuWa/TOPMed/1KG) + vPortal (Fst/Tajima's D) Annotation

Date: 2026-08-10
Status: COMPLETE — master TSV assembled (37 loci)
Role: mainline — teacher-requested expanded signal table (Excel catalog supplement)
Sample policy: ALL (4383); Coordinates hg38

## 1. Task (user/teacher)

From `SDS_signals_clump.xlsx` catalog: list all loci with **p < 5e-5**; annotate
upstream/downstream genes for intergenic variants; add NyuWa/TOPMed/1KG results
(RePoS); add evolutionary values from vPortal (Fst, Tajima's D); EAS populations
only (CHN best). Output as TSV (user merges to Excel manually).

## 2. Locus list (37, adopted config r2=0.1_win=0.5cM_ldmerge, lead p < 5e-5)

chr4×1 (ADH1B), chr11×1 (intergenic OR4B2P/OR4X2), chr12×3 (CCDC63, ALDH2,
MAPKAPK5), chr19×31 (4885kb region 15, 10690kb region 16), chr22×1 (ACO2).
16 are catalog signals; 21 are sub-Bonferroni clumps (5e-5 > p > 1.1e-8).

## 3. Data sources

| Source | What | Method |
|---|---|---|
| SDS (local) | norm_SDS, P, norm_iHS (polarized), EHH_class | signals_clump catalog + all_clump_signals + polarized norm files |
| VEP (local, WGS env) | consequence, gene | 37-lead VCF → VEP cache GRCh38.112 |
| rsID | Ensembl REST colocated_variants (YAML parse) | 37/37 found |
| GTF (GRCh38.112) | upstream/downstream genes (nearest) | local file |
| **RePoS** (bigdata.ibp.ac.cn/RePoS) | **NyuWa SDS/iHS, TOPMed SDS (AFR/EAS/EUR), 1KGP iHS (per pop)** | variant.php?vid=chr-pos per locus; EAS only (CHB/CHS/JPT reported, CDX/KHV available) |
| **vPortal** (mulinlab.org/vportal) | **EAS FST1/FST2 (Wright / Cockerham&Weir), Tajima.D, normiHS, normnSL, iSAFE, CLR, PI, DDAF, HET** | portal/queryEvolution?q=chr:pos&g=hg38&hasref=false per locus |

Note: RePoS variant not found for 1 locus (chr19:10378293 rs56295652 TYK2 — absent
from DB, kept with "." for RePoS columns; vPortal values present).

## 4. Highlights (EAS)

- **ADH1B (rs1229984)**: EAS_FST1=0.549, FST2=0.791 (top Fst genome-wide candidate);
  NyuWa SDS 11.84 (p=2.4e-32), TOPMed EAS 7.47 (p=7.9e-14), 1KG iHS CHB -1.08/CHS -1.43
- ALDH2 (rs671): NyuWa 11.86, TOPMed EAS 8.66 (p=4.9e-18), FST1=0.139
- chr12:110858516 (CCDC63): NyuWa 4.17, TOPMed EAS 4.74 (p=2.1e-6) — independent chr12 block
- chr22 ACO2: NyuWa 3.84 (p=1.3e-4), FST1=0.166
- chr19 10325679 (RAVER1): NyuWa 2.25, 1KG CHB iHS -2.09 (p=0.037)
- chr19 10555015: EAS_normiHS (vPortal) +2.54 — strongest vPortal iHS among chr19 loci
- Most chr19 4885kb/10690kb blocks: weak NyuWa/TOPMed/1KG signals (consistent with
  SDS-specific/recent selection)

## 5. Caveats

- **vPortal AA differs from SDS at some loci** (e.g. ADH1B: vPortal posSel AA=T vs
  SDS AA=C — different ancestral annotation conventions); values reported as-is
  with source's own polarization.
- RePoS/1KG iHS hg19-derived (Johnson 2018) — signs relative to their own annotation.
- 1KG iHS "." = allele absent/NA in that population.
- norm_iHS for sub-threshold loci = polarized ALL iHS (local).

## 6. Output

`SDSworkspace/results/production/sds_output/ALL/signals_clump/SDS_signals_clump_p5e5.tsv`
(37 rows × 27 cols: SNP, Signal, CHR, POS_hg38, rsID, Consequence, Genes,
norm_SDS, P-value, norm_iHS, Upstream_gene, Downstream_gene, NyuWa_SDS,
NyuWa_iHS, TOPMed_EAS_SDS, 1KG_iHS_CHB/CHS/JPT, EAS_FST1, EAS_FST2,
EAS_Tajima.D, EAS_normiHS, EAS_iSAFE, EAS_CLR, EAS_PI, EAS_DDAF, EHH_class)

User merges into SDS_signals_clump.xlsx manually.
