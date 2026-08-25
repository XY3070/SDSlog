# chr19 R1/R2 lead-level LD、重组与 EHH adjudication

Date: 2026-08-25  
Topic: sds / orthogonal evidence  
Status: current-data adjudication complete; genealogy pending

## Question

判断 chr19 R1/R2 中 greedy clump lead 是否代表同一条 selected haplotype，还是多个相对独立的 selection signal；记录当前数据能支持到什么程度，以及 Relate 完成前的下一步。

## Inputs

- Lead annotation: `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/sds_output/ALL/signals_clump/loci_p5e5_annotated.tsv`
- Phased VCF: `/data/home/grp-wangyf/xuyuan/SDSworkspace/input/raw/vcf/ALL/shapeit5/UKBQC_ALL_chr19.phased.vcf.gz`
- Clump sweep: `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/sds_output/ALL/signals_clump/all_clump_signals.tsv`
- Analysis script: `/data/home/grp-wangyf/xuyuan/SDSworkspace/oneoff/analyze_chr19_r1_r2_now.py`
- EHH script: `/data/home/grp-wangyf/xuyuan/SDSpipeline/scripts/orthogonal/run_ehh_at_signals.py`

Regions:

- R1: locus72–90, 19 leads
- R2: locus91–104, 14 leads
- ALL phased data: 4,383 diploid samples / 8,766 haplotypes
- EHH: both regions, lead ±100 kb

## Results

### Lead-level LD

- 33 leads produce 528 pairwise comparisons.
- Every pair has 8,766 usable phased haplotypes.
- Maximum lead r² is 0.088 in R1 and 0.086 in R2.
- No lead pair reaches the greedy-clump threshold `r² >= 0.1`.
- locus77 maximum same-R1 lead r² is 0.0071.
- locus79 maximum same-R1 lead r² is 0.0742, with locus81.

Interpretation: lead-level evidence supports separation of the selected lead markers. This is not equivalent to proving independent causal mutations: low-frequency proxies, alternate markers, and multiple origins remain possible.

### Four-gamete test

- 525/528 lead pairs show all four phased haplotypes.
- This is direct evidence of recombination between most lead pairs in the observed sample.
- It argues against treating the full R1 or R2 interval as one unrecombined haplotype.
- It does not by itself establish independent selection origin or genealogy.

### Clump stability

- Lead representation is relatively stable in the existing sweep index.
- Member extent is not stable, especially locus79: member count ranges from 1 to 233 across `r²`/window configurations.
- locus77 member count ranges from 8 to 39 and remains a polarization/high-frequency QC concern.

Interpretation: greedy clump has identified useful lead candidates, but member count must not be interpreted as biological region size. locus79 is best labeled `stable lead, unstable LD tail` pending proxy-level LD.

### SDS direction

- R1 contains mostly positive SDS but also negative loci83, 86, 82, and especially locus77 (`norm_SDS=-7.5`).
- R2 is predominantly positive, with locus99 as the main opposite-direction lead.

Interpretation: R1 has weaker directional coherence than R2. locus77 requires AA/DA, DAF, raw rSDS, and derived/minority haplotype QC before biological interpretation.

### EHH

- 33/33 lead EHH runs completed after fixing `chr19`/`19` contig handling, duplicate POS handling, selscan POS query, output parsing, and allele-specific duplicate-position selection.
- All final overview curves use ±100 kb.
- R1 strongest derived EHH at 50 kb: locus85=0.086, locus81=0.062, locus79=0.052.
- R2 strongest derived EHH at 50 kb: locus91=0.221, locus96=0.150, locus100=0.143, locus104=0.134, locus95=0.131.

Interpretation: R2 shows stronger and more coherent extended-haplotype evidence than R1. All-lead overlay is an overview only; lead ranking requires EHH50/EHH100, half-life, iHH, and derived/ancestral comparison.

Important QC: locus78 previously produced EHH `-1`; this was selscan's undefined sentinel caused by two ALT records at one POS and selecting monomorphic `T:C` instead of lead `T:G`. It was corrected and rerun. EHH values must never be interpreted as negative biological measurements.

## Adjudication

Current conclusion:

1. R1 is highly consistent with multiple relatively independent or semi-independent selection signals: lead-to-lead LD is below threshold, recombination is observed for nearly all pairs, SDS direction is mixed, and no single shared EHH pattern spans all leads.
2. R2 has multiple lead-level-separated signals, but its stronger/coherent EHH pattern leaves open whether several leads are markers on one or a few related selected haplotypes. Relate genealogy is required before calling independent origins.
3. Greedy-clump member instability does not overturn the lead-level result; it specifically warns that clump boundaries and member counts are not biological boundaries.
4. No causal gene or independent selection origin is assigned from these analyses alone.

## Next steps

1. Complete full-chr19 Relate with current CHB pyrho map; do not build separate regional trees.
2. Run RelateSelection/branch-based summaries on full chr19, then subset R1/R2 ± flank.
3. For each lead: selected-allele branch sharing, local TMRCA/age, allele trajectory, and independent-origin assessment.
4. Use CLUES2 only for prioritized leads after genealogy and polarization QC.
5. Before final R1/R2 claim, compute member-level cross-clump proxy LD; current result is lead-level LD only.
6. Add EAS/non-EAS AF comparison after harmonized CHB/CHS/JPT/KHV or equivalent population frequency inputs become available.

## Canonical outputs

- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/sds_output/ALL/chr19_r1_r2_now/lead_master.tsv`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/sds_output/ALL/chr19_r1_r2_now/lead_pairwise_ld.tsv`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/sds_output/ALL/chr19_r1_r2_now/four_gamete.tsv`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/sds_output/ALL/chr19_r1_r2_now/carrier_overlap.tsv`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/sds_output/ALL/chr19_r1_r2_now/clump_sensitivity.tsv`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/sds_output/ALL/chr19_r1_r2_now/ehh_summary.tsv`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/sds_output/ALL/chr19_r1_r2_now/R1_lead_r2_square.png`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/sds_output/ALL/chr19_r1_r2_now/R2_lead_r2_square.png`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/sds_output/ALL/chr19_r1_r2_now/R1_derived_EHH_decay.png`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/sds_output/ALL/chr19_r1_r2_now/R2_derived_EHH_decay.png`
