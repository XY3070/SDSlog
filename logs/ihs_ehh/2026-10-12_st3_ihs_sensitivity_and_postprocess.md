# SLE ST3 loci: iHS sensitivity experiment and corrected post-processing

Date: 2026-10-12  
Topic: ihs_ehh  
Type: test + post-processing  
Question: marker density、核心位点极化编码、归一化参照分别如何影响 iHS；如何为 184 个 SLE ST3 风险位点提供可信的方向化 iHS？

## Input paths

- `/data/home/grp-wangyf/xuyuan/chenjy/update_merge_samples/pipeline/polarized_ihs/chr{1..22}.ihs.out`
- `/data/home/grp-wangyf/xuyuan/chenjy/update_merge_samples/pipeline/polarized_ihs/phased_beagle_chr{1..22}.polarized.vcf.gz`
- `/data/home/grp-wangyf/xuyuan/chenjy/update_merge_samples/pipeline/polarized_ihs/chr{1..22}.selscan.map`
- `/data/home/grp-wangyf/xuyuan/chenjy/update_merge_samples/pipeline/annotated_ihs_st3/SLE相关位点_ST3位点list.annotated.csv` (existing CSV locus list; source workbook untouched)
- `/data/home/grp-wangyf/xuyuan/chenjy/ihs_sensitivity_experiment.py`
- `/data/home/grp-wangyf/xuyuan/chenjy/annotate_ihs_st3.py`

## Output paths

- `/data/home/grp-wangyf/xuyuan/chenjy/update_merge_samples/ihs_sensitivity/experiment_summary.tsv`
- `/data/home/grp-wangyf/xuyuan/chenjy/update_merge_samples/ihs_sensitivity/core_comparison.tsv`
- `/data/home/grp-wangyf/xuyuan/chenjy/update_merge_samples/ihs_sensitivity/marker_density.tsv`
- `/data/home/grp-wangyf/xuyuan/chenjy/update_merge_samples/pipeline/annotated_ihs_st3/SLE相关位点_ST3位点list.annotated.csv`
- `/data/home/grp-wangyf/xuyuan/chenjy/update_merge_samples/pipeline/annotated_ihs_st3/SLE相关位点_ST3位点list.annotated.summary.tsv`

## Parameters

- 1,378 phased samples; all 22 autosomes; core panel = 130 ST3 loci with `in_EPO=1` and valid iHS.
- selscan iHS run used `MAF=0.01`; marker-density thinning retained each core and hash-sampled flanking markers at 50% or 10%; genetic windows were ±0.1/±0.5 cM.
- Stage C and the final CSV use 100 equal-width DAF bins, pooled autosomal `in_EPO=1` loci only, sample SD (`n-1`), and require at least 20 high-confidence reference loci in a bin.
- Direction convention: selscan raw `log10(iHH_derived/iHH_ancestral)` is positive when derived haplotypes extend further. Multiply by −1 before reporting: **negative means derived-allele extension/selection; positive means ancestral-allele extension/selection**.
- EPO=2 uses the same EPO=1 reference bins but remains low-confidence. EPO=0 has no valid DAF or ancestral/derived direction; raw selscan score is retained as REF/ALT-oriented only.
- Bin definition: `bin = floor(DAF × 100)`, bins 0–99 (equal-width 0.01 DAF intervals; exact boundary values enter the higher bin). Reference = all genome-wide EPO=1 loci present in selscan raw output, pooled across autosomes. Standardize sign-flipped raw score within each bin as `(score − bin mean) / bin sample SD`; require at least 20 reference SNPs per bin. EPO=2 queries the same EPO=1 bins with its polarized derived frequency. EPO=0 has no DAF and is not assigned to a bin.

## Per-status accounting

Counts refer to the 184 ST3 risk-list records. `in_EPO=0` is an output annotation state, not a statement that every one of those records was observed in the phased VCF: 44 records are absent from that VCF, so EPO state cannot be assessed there.

| Category | Listed | In phased VCF | Raw selscan iHS | Polarized raw | DAF | norm iHS | Reason / handling |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | --- |
| `in_EPO=1` | 131 | 131 | 130 | 130 | 130 | 130 | One VCF locus `chr1:161626493` has zero iHH; selscan omitted it. |
| `in_EPO=2` | 7 | 7 | 7 | 7 | 7 | 7 | Low-confidence ancestral annotation; tentative direction, same EPO=1 normalization reference. |
| `in_EPO=0`, present in VCF | 2 | 2 | 2 | 0 | 0 | 0 | `chr1:196856157` and `chr22:18167915`; raw score/frequency retained as ALT-vs-REF; frequency is ALT frequency, not DAF. No polarized raw or norm by design. |
| Absent from phased VCF (stored as `in_EPO=0`) | 44 | 0 | 0 | 0 | 0 | 0 | No VCF alleles/genotypes, no raw iHS, and no EPO annotation can be assigned from this dataset. |
| **Total** | **184** | **140** | **139** | **137** | **137** | **137** | 44 VCF-absent; 1 VCF-present but selscan omitted; 2 EPO=0 raw-only. |

CSV meanings: `iHS raw selscan` preserves the score in `.ihs.out`; `iHS raw polarized` is its negation only for EPO=1/2; `selscan frequency` preserves `.ihs.out` frequency for all raw-output records; `DAF` is populated only for EPO=1/2; `norm iHS` is populated only for EPO=1/2 with a valid score and a reference bin containing ≥20 SNPs. Thus the raw field has 139 values, polarized raw 137, and normalized iHS 137—not 184. The two EPO=0 raw records are rs7519758 (`chr1:196856157`, raw 0.158689, ALT frequency 0.070029) and rs3180408 (`chr22:18167915`, raw −0.201151, ALT frequency 0.150581); these signs are not derived/ancestral directions.

## Scripts / commands / job IDs

- `/data/home/grp-wangyf/xuyuan/chenjy/ihs_sensitivity_experiment.py` — creates controlled A/B marker panels, runs selscan, and summarizes core-level effects.
- `/data/home/grp-wangyf/xuyuan/chenjy/annotate_ihs_st3.py` — streams the 22 raw `.ihs.out` files against polarized VCFs, creates EPO=1 pooled and chromosome-specific references, recomputes Stage C, and writes CSV only.
- Verification: `python3 -m py_compile chenjy/annotate_ihs_st3.py`; full script completed on chr1–22; CSV QC checked row count, allele case, annotation counts, and absence of legacy normalization columns.

## Key result files

- `IHS-001`: `/data/home/grp-wangyf/xuyuan/chenjy/update_merge_samples/pipeline/annotated_ihs_st3/SLE相关位点_ST3位点list.annotated.csv`
- Sensitivity summary: `/data/home/grp-wangyf/xuyuan/chenjy/update_merge_samples/ihs_sensitivity/experiment_summary.tsv`
- Per-core comparisons: `/data/home/grp-wangyf/xuyuan/chenjy/update_merge_samples/ihs_sensitivity/core_comparison.tsv`

## Conclusion

- **A — marker density matters.** Within ±0.5 cM, median marker retention was 49.94% (50% thinning) and 10.08% (10% thinning). Raw iHS sign changed at 15/130 and 24/129 core sites; median absolute change was 0.076 and 0.151. Keep the full QC-passed shared marker set as flanking markers; do not thin it for the primary scan.
- **B — ancestral/derived core coding determines raw sign.** Reversing core allele coding inverted raw iHS at all 130/130 sites exactly (residual 0). Correct EPO coding is therefore a hard prerequisite for directional interpretation.
- **C — normalization reference can alter direction, but less often here.** With the corrected EPO=1-only references, chromosome-wise versus pooled normalization differed in sign at 5/130 sites; median absolute normalized-score difference was 0.0599 (mean 0.0781; Pearson r=0.9938). Use one pooled high-confidence reference to keep chromosomes comparable. The earlier C result was superseded because its reference incorrectly treated unannotated loci as polarized DAF.
- **CSV update:** 184 rows retained: EPO=1: 131 (130 raw/norm; `chr1:161626493` omitted by selscan due to zero iHH), EPO=2: 7 raw/norm, EPO=0: 46 (2 present in VCF with raw-only; 44 absent from VCF and lacking EPO/raw). Totals: 140 in VCF, 139 raw scores, 137 polarized raw, 137 normalized. CSV has separate `REF`, `ALT`, `AA`, `DA`, `in_EPO`, raw selscan, polarized raw, selscan frequency, DAF, and norm columns. The generated XLSX was removed; source workbook was not modified.
- chr22:18167915 is present in the phased VCF and raw output (`REF=C, ALT=T`, `in_EPO=0`, selscan raw `-0.201151`). It is preserved as REF/ALT-oriented raw only; DAF, polarized raw and normalized iHS are blank because ancestral state is unavailable.

## Important interpretation

`in_EPO=1` supports primary signed interpretation. `in_EPO=2` supports sensitivity-only, tentative signed interpretation. `in_EPO=0` must not be called derived- or ancestral-selected from iHS sign; use direction-free magnitude or leave signed normalized iHS blank. Keep all QC-passed markers in the scan regardless of EPO label; EPO status controls annotation/reference use, not whether a variant can serve as an EHH flank marker.

## Next step

- Use the CSV for manual transfer to the risk-locus workbook. For downstream biological claims, prioritize `in_EPO=1`; report EPO=2 separately and do not assign selection direction to EPO=0.
