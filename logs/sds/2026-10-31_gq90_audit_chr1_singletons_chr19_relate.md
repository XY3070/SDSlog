# GQ>=90 audit: chr1 singletons and chr19 Relate region

Date: 2026-10-31
Topic: sds
Type: test
Question: Does the current SDS site set retain genotype GQ; what fraction passes GQ>=90; if singletons were additionally filtered at GQ>=90 how many remain; and is genotyping a confounder of the chr19 Relate results?

## Input paths

- `/data/home/grp-wangyf/xuyuan/raw/9k/snp_singleton_qc/snp_qc_pass.vcf.gz` (chr1; 4,383 samples; FORMAT GT:DP:GQ; 14,558,211 records)
- `/data/home/grp-wangyf/xuyuan/raw/9k/snp_singleton_qc_chr*_v*/snp_qc_pass.vcf.gz` (all other chromosomes; GT-only)
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/sds_input/ALL/chr*_s_file.txt`
- `/data/home/grp-wangyf/xuyuan/raw/9k/all.chr19.QC.vcf.gz` (9,466 samples; FORMAT GT:DP:GQ; tabix index; contig chr19)
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/input/raw/vcf/ALL/shapeit5/UKBQC_ALL_chr19.phased.vcf.gz` (4,383 samples; 3,803,696 sites)
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/orthogonal/relate_clues2/ALL/results/UKBQC_ALL_chr19_relate_sds.SDS`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/sds_output/ALL/signals_clump/loci_p5e5_final.tsv`

## Output paths

- `/data/home/grp-wangyf/xuyuan/SDSworkspace/tmp/altgq_chr19/chr19_altgq_perSite.tsv` (per-site ALT-GQ classification; 1,756,654 rows)
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/tmp/altgq_chr19/summary.txt`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/tmp/altgq_chr19/lead_vs_bg_lowGQ.tsv`
- chr1 per-site scan outputs were deleted after counting (numbers below)

## Parameters

- GQ bins in Graphtyper: {0,3,9,12,15,20,25,30,35,40,50,60,75,99}; `GQ>=90` is equivalent to `GQ=99`.
- chr1 singleton gate: `AC=1 && F_MISSING<=0.005`; GQ applied to the carrier genotype.
- chr19 region: `chr19:1-58,617,616`; biallelic SNPs; 4,383 ALL samples; `AC>0` after subsetting.
- ALT-GQ labels: chained `bcftools filter -m + -e 'GT="alt" & FMT/GQ<X' -s ALT_LTX` with X=90/75/50 (min ALT-call GQ bins). Single `&` is the verified same-sample operator; `&&` gives false positives.
- AC-matched background: 20 sites per lead, AC within +/-20%, >100 kb from any lead.
- Model role: audit-only. No demographic model used in this audit.

## Scripts / commands / job IDs

- bcftools 1.23.1 with `LD_PRELOAD=/data/home/grp-wangyf/intern/miniforge3/lib/libopenblas.so`.
- chr19 scan: 12 windows of 4,884,801 bp, `xargs -P 8`, `nice -n 10`; wall time 11m31s.
- Pipeline per window:
  `bcftools view -r chr19:S-E -S samples.txt -m2 -M2 -v snps -Ou all.chr19.QC.vcf.gz | bcftools +fill-tags -Ou -- -t AC | bcftools view -i 'AC>0' -Ou | bcftools filter -m + -e 'GT="alt" & FMT/GQ<90' -s ALT_LT90 -Ou | ... -e '<75' ... | ... -e '<50' ... | bcftools query -f '%POS\t%INFO/AC\t%INFO/GQ_AVG\t%INFO/DP_AVG\t%FILTER\n'`
- Validation: 100-kb slice agreed exactly with an independent awk min-ALT-GQ recomputation; 50-kb spot check also exact.

## Key plots

- None archived.

## Key result files

- `GQ-AUDIT-CHR19-PERSITE`: `/data/home/grp-wangyf/xuyuan/SDSworkspace/tmp/altgq_chr19/chr19_altgq_perSite.tsv`
- `GQ-AUDIT-CHR19-SUMMARY`: `/data/home/grp-wangyf/xuyuan/SDSworkspace/tmp/altgq_chr19/summary.txt`
- `GQ-AUDIT-CHR19-LEADVS BG`: `/data/home/grp-wangyf/xuyuan/SDSworkspace/tmp/altgq_chr19/lead_vs_bg_lowGQ.tsv`

## Results

### chr1 (only current QC VCF with FORMAT/GQ)

- Genotype GQ, 6 x 100-kb windows (169.0M calls): all calls GQ>=90 = 83.5%; ALT calls GQ>=90 = 92.3%; ALT >=75 = 95.8%; ALT >=50 = 99.1%.
- Current singletons (`s_file`): 3,501,799 positions / 3,525,093 AC=1 records.
  - After carrier GQ>=90: 3,440,145 positions (98.24%); 3,462,520 records (98.22%).
  - Loss: 61,654 positions (1.76%).
- 22-autosome singleton total: 44,081,927 positions. If chr1 is representative, ~1.8% loss genome-wide (~0.78M).
- 21/22 current QC VCFs are GT-only; a genome-wide GQ filter requires re-extraction from raw joint VCFs.

### chr19 Relate region

- Relate input: whole chr19, 3,803,696 phased sites, 4,383 samples (8,766 haplotypes). .SDS has 1,534,701 unique positions.
- Scanned: 1,756,654 biallelic SNP records (1,729,018 positions) with AC>0 in the 4,383 subset.
- Categories: all ALT GQ>=90 = 1,379,533 (78.53%); min ALT GQ 75-89 = 47,036 (2.68%); 50-74 = 90,304 (5.14%); <50 = 239,781 (13.65%); any ALT<90 = 377,121 (21.47%).
- Strong AC dependence: ALT<90 rate = 5.24% at AC=1, 15.92% at AC 2-5, 34.41% at 6-20, 64.76% at 21-100, 98.18% at AC>100. The min-based flag is a carrier-count artifact for common variants.
- 100-kb windows (n=516 with >=50 sites): mean 21.32%, median 20.71%, p95 30.01%, max 77.62%. Top windows: 24.8 Mb (77.6%), 58.6 Mb (65.6%), 24.9 Mb (59.1%), 24.4 Mb (57.1%), 27.3 Mb (53.3%), 54.8 Mb (50.6%), 47.9 Mb (46.3%). Low-GQ burden clusters in pericentromeric/subtelomeric difficult regions.
- Signal regions: R1 chr19:4,748,270-5,023,601 = 24.17% flagged; R2 chr19:10,324,817-11,002,401 = 22.10%; genome background = 21.47%.
- Relate-used sites: all 1,534,701 matched; flagged rate 20.15% (slightly below background); category mix 79.85/2.58/4.95/12.61%.
- 35 chr19 leads: all have min ALT GQ<50; they are common (AC 439-8,276). Low-GQ carrier fraction (frac_low = ALT calls with GQ<90 / ALT calls):
  - Leads: median 4.94% (IQR 1.95-9.55), mean 8.0%; 17/35 >5%, 9/35 >10%, 2/35 >25%.
  - AC-matched background (700 sites): median 4.26% (IQR 1.66-12.29), mean 11.6%; 318/700 >5%, 194/700 >10%, 100/700 >25%.
  - Mann-Whitney p=0.33 (AUC 0.52); per-lead matched comparison: leads slightly lower (Wilcoxon p=0.0045). No lead enrichment.
  - Key leads: 4,768,268 frac_low 2.62%, mean ALT GQ 98.0; 5,023,601 10.89%/94.7; 10,515,885 4.94%/97.2; 10,555,015 7.58%/96.6.
- INFO/GQ_AVG is not a substitute for ALT-carrier GQ: 210,880/377,121 (55.92%) of min-ALT<90 sites still have GQ_AVG>=90; GQ_AVG is the 9,466-sample all-call mean (not recomputed on 4,383) and is ~0.6-1.2 units above the 4,383-sample all-call mean.

## Conclusion

- Applying GQ>=90 to singleton carriers has a small effect: chr1 retains 98.24% of singleton positions; the 22-autosome singleton set is expected to lose ~1.8% if chr1 is representative.
- Genotyping quality does not show a systematic signal-region effect on chr19: R1/R2 and the Relate-used sites are at or below the chromosome-wide low-GQ burden, and 35 lead low-GQ carrier fractions match AC-matched background. Low-GQ sites cluster in pericentromeric/subtelomeric regions.
- The current site set retains usable genotype GQ only on chr1; GQ_AVG cannot replace ALT-carrier GQ. A definitive genotyping sensitivity test still requires a controlled Relate/CLUES2 rerun with a GQ-aware site or genotype filter.

## Next step

- If a sensitivity arm is required, build a per-site low-GQ-carrier-fraction filter (e.g. exclude sites with >25% ALT calls at GQ<90) and rerun chr19 Relate/CLUES2 as an audit-only arm. Do not filter on min ALT GQ alone: it drops 13.65% of sites including 32/35 leads because of carrier-count effects. Genotype-level masking would require re-phasing/imputation and is not supported by the current risk profile.
