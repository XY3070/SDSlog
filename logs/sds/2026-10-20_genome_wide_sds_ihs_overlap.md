# Genome-wide SDS vs iHS overlap analysis

> Superseded by `2026-10-21_ihs_epo3_reorg_and_overlap_ALL.md`. NCN iHS was removed;
> canonical results now use ALL epo3 under `orthogonal/ihs/`. Numbers below are kept for history only.

Date: 2026-10-20  
Population: NCN (North Chinese, 1255 samples / 2510 haplotypes)  
SDS threshold: |norm_SDS| > 4  
iHS threshold: |norm_iHS| > 4  
Overlap window: 100 kb

## Summary statistics

- Total SDS extreme signals: 3035
- Total iHS extreme signals: 432
- SDS-only (no iHS within 100kb): 2741 (90.3%)
- Shared (both strong within 100kb): 294 (9.7%)
- SDS-dominant (iHS weak but present): 0 (0.0%)

**Young mutation candidates: 2741 (90.3%)**

## DAF distributions

| Category | n | Mean DAF | Median DAF | Q1 | Q3 |
|----------|---|----------|------------|-----|-----|
| SDS-only | 2741 | 0.358 | 0.282 | 0.096 | 0.577 |
| Shared | 294 | 0.230 | 0.102 | 0.084 | 0.314 |

Shared signals have significantly lower DAF (p < 0.001 by Mann-Whitney U test), suggesting they may represent stronger or older selection events.

## Sign concordance (shared signals only)

- Total shared: 294
- Concordant (opposite signs, as expected): 192 (65.3%)
- Discordant (same signs): 102 (34.7%)

Expected concordance: SDS positive (derived young/extended) ↔ iHS negative (Johnson convention: derived extended).

Moderate concordance suggests some shared signals may have complex selection histories or polarization uncertainties.

## Per-chromosome breakdown

Notable chromosomes:
- chr4: 52% SDS-only, 48% shared (highest shared fraction)
- chr6: 83% SDS-only, 17% shared (MHC region, complex selection)
- chr16: 99.3% SDS-only, 0.7% shared
- All other chromosomes: 100% SDS-only

## Interpretation

1. **Genome-wide young mutation pattern**: 90% of SDS extreme signals lack nearby iHS extreme signals, consistent with selection acting within the past ~2000 years (SDS timescale) rather than ~25000 years (iHS timescale).

2. **Chr19 R1/R2 context**: The 33 leads in chr19 R1/R2 are part of this genome-wide pattern. All 33 were classified as young mutation candidates (SDS strong, iHS weak).

3. **Shared signals**: The 9.7% of SDS signals with nearby iHS signals may represent:
   - Older selection events (~25000 years) that also affect recent allele frequency
   - Regions of very strong selection detectable across multiple timescales
   - Coincidental overlap of independent signals

4. **DAF difference**: Shared signals have lower DAF, suggesting they may be under stronger or more recent selection despite having detectable iHS.

## Conclusions

The young mutation signature (SDS strong + iHS weak) is a **genome-wide phenomenon**, not specific to chr19 R1/R2. This supports the interpretation that most SDS extreme signals reflect very recent selection (~2000 years) that has not had time to build extended haplotype structure detectable by iHS (~25000 years).

## Output files

- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/orthogonal/genome_wide_overlap_window100kb.tsv`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/orthogonal/genome_wide_overlap_summary.txt`
