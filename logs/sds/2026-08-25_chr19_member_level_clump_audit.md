# chr19 R1/R2 member-level clump audit

Date: 2026-08-25  
Topic: sds / LD clump adjudication  
Status: complete; genealogy pending

## Purpose

Lead-level LD showed all R1/R2 lead pairs below `r²=0.1`, but greedy clump member ranges were unstable. This audit keeps the existing production clumps and tests whether any member proxy pair across two clumps has high genotype LD. It audits clump boundaries; it does not redefine biological signals by itself.

## Inputs and method

- Existing lead annotation: `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/sds_output/ALL/signals_clump/loci_p5e5_annotated.tsv`
- Existing members: `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/sds_output/ALL/signals_clump/clump_members.tsv`
- Phased VCF: `/data/home/grp-wangyf/xuyuan/SDSworkspace/input/raw/vcf/ALL/shapeit5/UKBQC_ALL_chr19.phased.vcf.gz`
- Script: `/data/home/grp-wangyf/xuyuan/SDSworkspace/oneoff/analyze_chr19_member_ld.py`
- Exact lead POS/REF/ALT matching; singleton leads absent from member table were added back as their own member.
- Genotype dosage r²; 4,383 samples; 1,118 unique member variants; missing genotypes=0.

## Results

- 262 within-region clump pairs: R1=171, R2=91.
- R1: 19/171 pairs have `max member r² >=0.1`; 12/171 have `>=0.2`.
- R2: 33/91 pairs have `max member r² >=0.1`; 24/91 have `>=0.2`.
- R1 proxy-LD components at max-member `r²>=0.1`:
  - isolated: locus90; locus83; locus72; locus82; locus76;
  - locus85+locus80;
  - locus74/75/78/84/86/87/89;
  - locus73/77/79/81/88.
- R2: all 14 leads form one connected proxy-LD component at max-member `r²>=0.1`.

Representative high proxy-LD pairs:

- R1 locus73–79: lead r²=0.019, max member r²=0.922;
- R1 locus86–84: lead r²=0.060, max member r²=0.922;
- R1 locus79–81: lead r²=0.078, max member r²=0.909;
- R1 locus75–89: lead r²=0.083, max member r²=0.828;
- R2 locus97–96: lead r²=0.026, max member r²=0.967;
- R2 locus92–96: lead r²=0.089, max member r²=0.920.

## Interpretation

1. Lead-level clumping over-splits some local LD networks. A low lead-lead r² does not guarantee that all members of the two clumps are independent.
2. R1 still does not behave like one uniform macro-sweep: member proxy-LD separates it into several local networks plus isolated leads, and SDS direction/EHH are heterogeneous.
3. R1 can currently be described as multiple local candidate networks and isolated candidates, not yet as 19 fully independent selection events.
4. R2 is especially unresolved: all 14 leads are connected by at least one high-LD member proxy pair. This could reflect a long/complex LD structure, greedy proxy reassignment, or several related selected haplotypes. Relate genealogy is required.
5. Max-member LD is sensitive to one proxy pair and transitive chaining. Components are diagnostic, not final merge calls. Use member-pair fraction/P95/P99 and local genealogy before collapsing components.

## Next steps

1. Use `member_cross_clump_ld.tsv` to rank pairs by max r², P95, P99, and fraction of member pairs with `r²>=0.1`.
2. Define robust proxy-LD edges using both a max threshold and a minimum member-pair support threshold; do not merge by single-edge connected components alone.
3. Re-select stable representative leads for each robust local network, retaining all original leads as annotations.
4. Use full-chr19 Relate local trees to determine whether each proxy network is one genealogy or multiple selection origins.
5. Keep locus77 polarization QC and locus79 unstable-tail flags active.

## Conservative member-aware clumping (completed)

The cross-clump table was converted into an exploratory member-aware clump
catalog. An edge is `robust` only when it has at least 20 cross-member pairs,
at least 5 pairs with `r²>=0.1`, and member-pair P95 `>=0.1`. Robust edges are
then merged by complete-link clustering: every cross-clump pair between two
provisional groups must pass the robust rule. This prevents one proxy pair or
transitive chaining from collapsing the whole macro-region.

Results:

- R1: 11 robust edges; 5 multi-lead local clumps (2, 2, 2, 2, 2 leads), with
  the remaining 9 leads singleton candidates.
- R2: 19 robust edges; 3 multi-lead local clumps (2, 4, 3 leads), with the
  remaining 5 leads singleton candidates.
- The prior max-only graph connected all R2 leads, but the conservative
  complete-link result does not. Therefore max-member connected components are
  diagnostic only, not a final clump definition.

Outputs:

- `member_aware_edges.tsv`: each cross-clump pair classified as `robust`,
  `proxy_only`, or `none`.
- `member_aware_clumps.tsv`: conservative exploratory clump assignment.
- `analysis_status.tsv`: `cross_clump_proxy_LD=complete`; final biological
  independence remains pending Relate genealogy.

## Outputs

- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/sds_output/ALL/chr19_r1_r2_now/member_cross_clump_ld.tsv`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/sds_output/ALL/chr19_r1_r2_now/member_ld_components.tsv`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/sds_output/ALL/chr19_r1_r2_now/member_ld_qc.tsv`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/sds_output/ALL/chr19_r1_r2_now/R1_member_max_r2_square.png`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/sds_output/ALL/chr19_r1_r2_now/R2_member_max_r2_square.png`
