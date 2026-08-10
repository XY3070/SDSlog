# 2026-08-02 SLE vs non-SLE SDS comparison

## Context

After completing ALL=4383 SDS scan (Gravel CHB gamma), split cohort into SLE (N=1965) and non-SLE (N=2367) to check for differential recent selection signals. Both groups re-derived singleton definitions within their respective sample sets — singletons are group-specific, not inherited from ALL cohort.

**Demographic model:** Gravel CHB (`g_file.Gravel_EAS.txt`)
**Present-day Ne(0):** implicit / unknown
**Input lineage:** `raw/9k/snp_singleton_qc_chr{N}_v*/snp_qc_pass.vcf.gz` → `run_sds_input.sh` with group-specific sample lists
**Sample policy:** SLE=1965, non-SLE=2367, ALL=4383
**Role:** mainline (exploratory comparison)

## Methodology

### Singleton redefinition verification
Verified on chr22: ALL singleton total (584,791) ≠ SLE (336,455) + non-SLE (454,697) sum. Mean singletons/sample: ALL=133.4, SLE=171.2, non-SLE=192.1. Differences consistent with:
- Smaller sample size → more variants qualify as singleton per individual
- Known SLE lower singleton count phenotype (SDSlog 2026-06-26: SLE 626 vs control 678, p<0.001)

### Pipeline
1. Sample lists from `SDSworkspace/manifests/cohort_freeze_sds.tsv` (Phenotype column: SLE/control, SDS_include=True)
2. Input generation: `run_sds_input.sh` per-chromosome per-group (44 tasks)
3. Compute: `submit_sds_genomewide.sh --pops "SLE non-SLE"` on LSF smp queue (44 chr×pop tasks, 0 failures)
4. Postprocess: `postprocess_sds_results.py` with fixed params (bin_mode=maf, maf_threshold=0.05, mask=grch38_combined_blacklist.bed)
5. Three-way merge: SLE ∩ nonSLE ∩ ALL = 4,452,790 variants

## Key Results

### Global statistics

| Metric | Value |
|---|---|
| SLE vs nonSLE Pearson r | **0.4350** (moderate) |
| SLE vs ALL Pearson r | **0.7812** (strong) |
| nonSLE vs ALL Pearson r | **0.7504** (strong) |
| Mean delta (SLE − nonSLE) | −0.0085 |
| SD delta | 1.0616 |
| \|delta\| > 4σ | 726 variants (0.016%) |

**Interpretation:** SLE and non-SLE share moderate correlation in SDS signal — most selection signals overlap. SLE tracks ALL slightly more closely than non-SLE does (r=0.781 vs 0.750). Systematic negative delta suggests non-SLE has marginally higher SDS scores genome-wide, possibly related to higher singleton counts in non-SLE group.

### non-SLE specific signals (nonSLE > SLE)

These regions show elevated SDS in non-SLE but not in SLE:

| Locus | Window | delta (SLE−nonSLE) | SLE | nonSLE | ALL | Notes |
|---|---|---|---|---|---|---|
| **chr19:52.5Mb** | 52,500,437−52,505,738 | **−1.07** | −0.442 | +0.628 | +0.342 | Strongest single-window divergence |
| **chr1:92Mb** | 92,501,859−92,999,212 | **−0.86** | −0.505 | +0.351 | −0.292 | "floating point" — very isolated |
| chr1:92Mb proximal | 92,001,199−92,495,853 | −0.66 | −0.371 | +0.292 | +0.041 | Adjacent window, same direction |
| chr2:108Mb | 108,548,916−108,999,920 | −0.66 | +0.279 | +0.937 | +0.472 | nonSLE much higher |
| chr3:49Mb | 49,045,991−49,497,682 | −0.61 | +0.039 | +0.645 | +0.896 | ALL also elevated |
| chr5:152Mb | 152,500,459−152,997,671 | −0.33 | −0.102 | +0.230 | +0.127 | User-reported chr5 peak |
| chr11:85Mb | 85,000,846−85,499,809 | −0.24 | −0.031 | +0.206 | +0.071 | chr11 peak cluster |
| chr14:42Mb | 41,500,012−41,999,863 | −0.24 | +0.021 | +0.265 | +0.069 | chr14 signal |
| chr22:40Mb | 39,501,787−39,999,526 | −0.26 | −0.057 | +0.201 | +0.138 | chr22 peak higher in nonSLE |

### SLE specific signals (SLE > nonSLE)

| Locus | Window | delta (SLE−nonSLE) | SLE | nonSLE | ALL | Notes |
|---|---|---|---|---|---|---|
| **chr9:92Mb** | 92,001,508−92,499,195 | **+0.25** | +0.201 | −0.047 | +0.134 | SLE unique peak — weak but consistent |
| chr10:87Mb | 87,034,594−87,109,973 | +0.56 | +0.171 | −0.388 | −0.146 | Moderate divergence |
| chr2:196Mb | 196,500,221−196,997,553 | +0.34 | +0.410 | +0.071 | +0.080 | SLE elevated, nonSLE near-zero |
| chr7:72Mb | 72,500,699−72,763,459 | +0.32 | +0.208 | −0.109 | +0.033 | |

### Per-chromosome mean delta

Significant chromosomes (|mean delta| > 0.01):

| Chr | mean delta | mean SLE | mean nonSLE | Direction |
|---|---|---|---|---|
| **chr22** | **−0.0281** | −0.0113 | +0.0168 | nonSLE↑↑ |
| **chr18** | **−0.0188** | −0.0126 | +0.0062 | nonSLE↑ |
| **chr14** | **−0.0166** | −0.0065 | +0.0101 | nonSLE↑ |
| chr3 | −0.0137 | −0.0128 | +0.0009 | nonSLE↑ |
| **chr11** | **−0.0120** | −0.0128 | −0.0008 | nonSLE↑ |
| chr12 | −0.0104 | −0.0008 | +0.0097 | nonSLE↑ |
| chr15 | −0.0111 | −0.0058 | +0.0053 | nonSLE↑ |

**All chromosomes with significant divergence show nonSLE > SLE** — consistent with the systematic bias toward lower SDS in SLE.

## Comparison with ALL=4383

- Both SLE and nonSLE SDS correlate strongly with ALL (r~0.75-0.78), confirming the split preserves the main selection architecture.
- non-SLE specific signals (chr1:92Mb, chr19:52Mb, chr5:152Mb, chr22:40Mb) show ALL intermediate between SLE and nonSLE, with ALL closer to nonSLE — consistent with nonSLE being a larger fraction (2367/4383=54%) of ALL.
- chr9:92Mb SLE-unique peak: SLE=+0.201, nonSLE=−0.047, ALL=+0.134 (ALL intermediate, closer to SLE direction). This is the most notable SLE-specific signal.

## Caveats

1. **Singleton count confound (SDSlog 2026-06-26):** SLE cases have lower singleton counts than controls (626 vs 678). This confound correlates with PC1 (r=−0.50) and may systematically depress SLE SDS scores. The genome-wide negative delta (SLE−nonSLE mean=−0.0085) is consistent with this.
2. **Sample size difference:** SLE (1965) vs non-SLE (2367) — different power to detect selection. Smaller N → fewer singletons overall → potentially weaker SDS signal. But per-individual singleton rate is higher in SLE (171 vs 192 on chr22).
3. **Case/control confounding with region:** SLE prevalence varies from 0% (Guangzhou, Jinan) to 70% (Shenzhen). Geographic structure may contribute to apparent SLE-specific signals.
4. **No ancestry stratification:** SLE and non-SLE groups were not re-balanced for population structure. South-North cline may differ between groups.

## Outputs

| File | Path |
|---|---|
| Three-way merge | `SDSworkspace/results/production/sds_output/sle_vs_nonsle_comparison/threeway_merge.tsv` |
| Mirror Manhattan (CMplot) | `SDSworkspace/results/production/sds_output/sle_vs_nonsle_comparison/mirror_manhattan_SLE_vs_nonSLE_CMplot.png` (SLE top / non-SLE bottom; shared peaks black, SLE-only red, nonSLE-only blue) |
| ~~Mirror Manhattan~~ | ~~`mirror_manhattan_SLE_vs_nonSLE.png`~~ — **superseded, deleted 2026-08-05** (wrong: matplotlib non-CMplot, no peak annotation) |
| ~~Delta Manhattan~~ | ~~`delta_manhattan_SLE_minus_nonSLE.png`~~ — **superseded, deleted 2026-08-05** |
| ~~Three-panel comparison~~ | ~~`three_panel_comparison.png`~~ — **superseded, deleted 2026-08-05** |
| Per-chromosome summary | `SDSworkspace/results/production/sds_output/sle_vs_nonsle_comparison/per_chr_threeway_summary.tsv` |
| Top divergent windows | `SDSworkspace/results/production/sds_output/sle_vs_nonsle_comparison/top_divergent_windows.tsv` |
| SLE Manhattan | `SDSworkspace/results/production/sds_output/SLE/sle.manhattan.png` |
| non-SLE Manhattan | `SDSworkspace/results/production/sds_output/non-SLE/non-sle.manhattan.png` |

## Input provenance

| Layer | Details |
|---|---|
| VCF | `raw/9k/snp_singleton_qc_chr{N}_v*/snp_qc_pass.vcf.gz` (v1/v2/v3) |
| Sample lists | `SDSworkspace/results/production/sds_input/{SLE,non-SLE}/samples.txt` |
| Cohort freeze | `SDSworkspace/manifests/cohort_freeze_sds.tsv` |
| Gamma | `SDSworkspace/results/production/gamma/g_file.Gravel_EAS.txt` |
| Mask | `SDSworkspace/input/reference/mask/grch38_combined_blacklist.bed` |
| Input files | `SDSworkspace/results/production/sds_input/{SLE,non-SLE}/chr{N}_{b,s,t,o}_file.txt` |

## Compute provenance

| Layer | Details |
|---|---|
| Queue | LSF smp |
| Job group | `sle_vs_nonsle` |
| Tasks | 44 (22 chr × 2 pops), 0 failures |
| Chunk rows | 10000 |
| Array parallel | 29 |
| Orchestrator log | `SDSworkspace/results/production/sds_output/sle_vs_nonsle_compute_20260801_182943.log` |

## References

- [[2026-06-26_population_structure_analysis]] — SLE case/control distribution, singleton confound, PC1 structure
- [[2026-08-02_sle_vs_nonsle_confounder_diagnostic]] — **Confounder diagnostic report**: 8-confounder table, 7 diagnostic figures, 20 UMAP plots, decision tree. Key finding: global delta is normalization artifact (delta_rSDS≈0, delta_norm=−0.0085).
- ALL=4383 benchmark: `SDSworkspace/results/production/sds_output/ALL/`
