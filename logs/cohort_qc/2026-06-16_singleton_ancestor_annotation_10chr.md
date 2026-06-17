# Singleton Ancestral Allele Annotation (10 chromosomes)

Date: 2026-06-16
Topic: cohort_qc
Type: production
Question: chr21 singleton QC 通过后，用 Ensembl ancestor FASTA 对所有已 QC 染色体的 singleton 位点做 AA/DA 注释，有多少位点可以成功注释？

## Input paths

- `SDSworkspace/input/reference/markers/homo_sapiens_ancestor_GRCh38/homo_sapiens_ancestor_{chr}.fa` — Ensembl 10-primate EPO ancestor FASTA (GRCh38), uppercase=high confidence, lowercase=low confidence, `.`=no alignment, `N`=unsupported, `-`=insertion
- `raw/9k/snp_singleton_qc_chr{4,5,7,14,17,18,19,20,21,22}_v{N}/filtered_singletons.vcf.gz` — post-QC singleton VCF per chromosome (10 dirs)

## Output paths

- `raw/9k/snp_singleton_qc_chr{chr}_v{N}/singleton_ancestor_annot.tsv` — per-site annotation (POS/REF/ALT/AA/AA_confidence/DA/annotation)
- `raw/9k/snp_singleton_qc_chr{chr}_v{N}/singleton_ancestor_stats.tsv` — per-chromosome summary stats
- `raw/9k/singleton_ancestor_stats_all_chr.tsv` — aggregated stats across all 10 chromosomes
- `raw/9k/annotate_ancestor_batch.py` — generalized script (accepts chromosome as CLI arg)
- `raw/9k/aggregate_ancestor_stats.py` — aggregation script

## Parameters

- `samtools faidx` batch mode with `-r` region file
- Ancestor FA seq name = bare chromosome number (e.g., `21`), matches VCF `#CHROM`
- AA confidence: uppercase base = high, lowercase = low, `./N/-` = no_coverage
- Classification: REF==AA → DA=ALT (REF=AA); ALT==AA → DA=REF (ALT=AA); neither → no_match

## Scripts / commands / job IDs

- `raw/9k/annotate_ancestor_batch.py` — main annotation script
- `raw/9k/annotate_ancestor.py` — original chr21-only version (superseded)
- `raw/9k/aggregate_ancestor_stats.py` — stats aggregation
- Runtime env: `mamba activate sds` (samtools 1.23.1)

## Key result files

| chr | total | annotated | rate | REF=AA | ALT=AA | no_match | no_coverage |
|---|---|---|---|---|---|---|---|
| 4 | 3,231,086 | 3,129,541 | 96.9% | 3,095,425 | 34,116 | 18,961 | 82,584 |
| 5 | 3,064,730 | 2,960,103 | 96.6% | 2,928,929 | 31,174 | 17,248 | 87,379 |
| 7 | 2,712,121 | 2,555,700 | 94.2% | 2,528,875 | 26,825 | 14,741 | 141,680 |
| 14 | 1,509,905 | 1,444,095 | 95.6% | 1,429,659 | 14,436 | 8,185 | 57,625 |
| 17 | 1,346,938 | 1,265,854 | 94.0% | 1,253,459 | 12,395 | 6,678 | 74,406 |
| 18 | 1,251,743 | 1,210,399 | 96.7% | 1,197,608 | 12,791 | 7,082 | 34,262 |
| 19 | 929,138 | 798,631 | 86.0% | 789,523 | 9,108 | 4,700 | 125,807 |
| 20 | 1,077,341 | 998,194 | 92.7% | 987,372 | 10,822 | 5,675 | 73,472 |
| 21 | 622,335 | 580,173 | 93.2% | 573,756 | 6,417 | 3,548 | 38,614 |
| 22 | 636,644 | 558,823 | 87.8% | 553,017 | 5,806 | 3,318 | 74,503 |
| **ALL** | **16,381,981** | **15,501,513** | **94.6%** | **15,337,623** | **163,890** | **90,136** | **790,332** |

- `raw/9k/singleton_ancestor_stats_all_chr.tsv`

## Conclusion

- 10 条已 QC 染色体共 16.4M singleton 位点，94.6%（15.5M）成功注释 AA/DA。
- 93.6% 位点 REF=AA（singleton 在 ALT = derived），1.0% ALT=AA（singleton 在 REF = derived）。
- chr19/chr22 annotation rate 最低（86-88%），因 ancestor alignment coverage 较差（no_coverage 占比高）。
- no_match（祖先碱基 ≠ REF 且 ≠ ALT）仅 90K，可能是三碱基位点或祖先推断错误。
- 45K-180K 的 low_confidence 注释（lowercase ancestor call）可后续按 confidence 分层使用。

## Next step

- 后续新染色体 QC 完成后，用 `annotate_ancestor_batch.py` 直接跑，再跑 `aggregate_ancestor_stats.py` 更新汇总。
- 考虑把 low_confidence 位点从 downstream analysis 中排除或做 sensitivity analysis。
