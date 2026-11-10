# HapMap3 LD scores (L2), GRCh38, SDS ALL=4383 — 2026-10-30

- Role: supporting resource (LD-score panel for external tSDS LD-score regression); not a mainline SDS run.
- Input lineage: SDS ALL=4383 phased VCF, SHAPEIT5 phase_common 1.1, GRCh38, biallelic SNPs, no missing genotypes, chr1-22.
  - `/data/home/grp-wangyf/xuyuan/SDSworkspace/input/raw/vcf/ALL/shapeit5/UKBQC_ALL_chr{N}.phased.vcf.gz`
- Sample policy: ALL=4383 (SLE=1965 + non-SLE=2367), exactly 4,383 samples in the VCF header.
- Site universe: all biallelic SNPs with MAF >= 0.01 (plink2 --snps-only --max-alleles 2 --maf 0.01), 7,983,040 SNPs across 22 chromosomes; 7,980,797 used by ldsc after its internal all-het/missing filter.
- HM3 list: `/data/home/grp-wangyf/leiyao/Database/HapMap3/w_hm3.snplist` (1,217,311 SNPs), not regenerated.
- rsID backfill: GATK dbSNP138 GRCh38 (dbSNP151 unavailable); chr:pos:unordered(REF,ALT) matching with HM3 allele-set check; 1,212,967 HM3 rsIDs found, 1,054,856 assigned in bfiles.
- LD score: ldsc v1.0.1 (commit aa33296), --l2 --ld-wind-kb 1000 (CHB cM map had large centromeric discontinuities), --print-snps HM3; focal SNP included; unbiased L2.
- Output: 1,054,856 HM3 SNPs; L2 min 0.807, mean 115.764, max 3699.039; NaN 0; L2 <= 0: 0.
- Spot check vs independent PLINK2 dosage recalculation: |diff| <= 0.04.
- No GWAS, tSDS, or LDSC regression was run.
- Deliverable: `/data/home/grp-wangyf/xuyuan/SDSworkspace/oneoff/ldscore_hm3_grch38_20261029/final/hm3_ldscore_GRCh38_4383.tar.gz`
  (contains hm3_ldscore_GRCh38.tsv.gz, qc.json, README.md, md5sum.txt).
